---
title:  Managing nodes with the NodePools CRD
slug: node-pools-crd
excerpt: ''
section: User guides
order: 2
---

<style>
 pre {
     font-size: 14px;
 }
 pre.console {
   background-color: #300A24; 
   color: #ccc;
   font-family: monospace;
   padding: 5px;
   margin-bottom: 5px;
 }
 pre.console code {
   border: solid 0px transparent;
   font-family: monospace !important;
   font-size: 0.75em;
   color: #ccc;
 }
 .small {
     font-size: 0.75em;
 }
</style>

**Last updated July 29<sup>th</sup> July, 2020.**


## Objective

OVHcloud Managed Kubernetes service provides you Kubernetes clusters without the hassle of installing or operating them. This guide will cover one of the first steps after ordering a cluster: managing nodes and node pools, using the NodePools CRD.

In this guide, we are assuming you're using the `NodePools` CRD via `kubectl` to manage your Kubernetes cluster. If you are using a different method, like the [OVHcloud Cloud Manager](https://ca.ovh.com/auth/?action=gotomanager), please refer to the relevant documentation: [Managing nodes and node pools](../managing-nodes/) guide.


## Requirements

- An OVHcloud Managed Kubernetes cluster
- The [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/){.external} command-line tool. You can find the [detailed installation instructions](https://kubernetes.io/docs/tasks/tools/install-kubectl/){.external} for this tool on Kubernetes' official site.


## Nodes and node pools

In your OVHcloud Managed Kubernetes cluster, nodes are grouped in node pools (group of nodes sharing the same configuration).  

When you create your cluster, it's created with a default node pool. Then, you can modify the size of this node pool, or add additional node pools of different sizes and types.

In this guide we explain how to do some basic operations with nodes and node pools using the `NodePools` CRD: adding nodes to an existing node pool, creating a new node pool...

## NodePools CRD

Kubernetes [Custom Resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) are extensions of the Kubernetes API. Like the default Kubernetes resources, the Custom Resources are endpoints in the Kubernetes API that store  collections of API objects of a certain kind. Custom Resources allows to easily extend Kubernetes by adding new features and behaviors.

The simplest way to add a Custom Resource to Kubernetes is to define a [`CustomResourceDefinition` (CRD)](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions) with the resource schema. 

One of our targets in developing the node pools for OVHcloud Managed Kubernetes was to give our users the capability to fully manage node pools (and by extension nodes themselves) from within Kubernetes, so the logical way to do it was to propose them as Custom Resources in your Kubernetes cluster, by developing the `NodePools` CRD.


To verify that the `NodePools` CRD in available in your cluster, do:

```bash
kubectl get crd
```

You get the list of installed CRDs and inside it the `nodepools.kube.cloud.ovh.com`

<pre class="console"><code>$ kubectl get crd
NAME                                             CREATED AT
bgpconfigurations.crd.projectcalico.org          2020-07-26T20:00:00Z
bgppeers.crd.projectcalico.org                   2020-07-26T20:00:00Z
blockaffinities.crd.projectcalico.org            2020-07-26T20:00:00Z
clusterinformations.crd.projectcalico.org        2020-07-26T20:00:00Z
felixconfigurations.crd.projectcalico.org        2020-07-26T20:00:00Z
globalnetworkpolicies.crd.projectcalico.org      2020-07-26T20:00:00Z
globalnetworksets.crd.projectcalico.org          2020-07-26T20:00:00Z
hostendpoints.crd.projectcalico.org              2020-07-26T20:00:00Z
ipamblocks.crd.projectcalico.org                 2020-07-26T20:00:00Z
ipamconfigs.crd.projectcalico.org                2020-07-26T20:00:00Z
ipamhandles.crd.projectcalico.org                2020-07-26T20:00:00Z
ippools.crd.projectcalico.org                    2020-07-26T20:00:00Z
networkpolicies.crd.projectcalico.org            2020-07-26T20:00:00Z
networksets.crd.projectcalico.org                2020-07-26T20:00:00Z
<b style="color: yellow">nodepools.kube.cloud.ovh.com                     2020-07-26T20:00:22Z</b>
volumesnapshotclasses.snapshot.storage.k8s.io    2020-07-26T20:00:20Z
volumesnapshotcontents.snapshot.storage.k8s.io   2020-07-26T20:00:20Z
volumesnapshots.snapshot.storage.k8s.io          2020-07-26T20:00:20Z
</code></pre>


You can get the details of the `NodePools` CRD by doing:

```bash
kubectl describe crd nodepools.kube.cloud.ovh.com
```

The most interesting part is the spec of the CRD, describing the `NodePool` object and its properties:

```yaml
spec:
  description: NodePoolSpec defines the desired state of NodePool
  properties:
    desiredNodes:
      description: Represents number of nodes wanted in the pool.
      format: int32
      maximum: 100
      minimum: 0
      type: integer
    flavor:
      description: Represents the flavor nodes wanted in the pool.
      type: string
    maxNodes:
      description: Represents the maximum number of nodes which should be
        present in the pool.
      format: int32
      maximum: 100
      minimum: 0
      type: integer
    minNodes:
      description: Represents the minimum number of nodes which should be
        present in the pool.
      format: int32
      maximum: 100
      minimum: 0
      type: integer
  required:
  - desiredNodes
  - flavor
  - maxNodes
  - minNodes
  type: object
```

After creation, the `desiredNodes` can be edited, and the node pool will automatically be resized to accommodate this new value. `minNodes` and `maxNodes` can also be edited at any time.

We will later propose cluster autoscaling based on node pols. We see some customer developing they own autoscaling scripts. We strongly encourage you to define `minNodes` and `maxNodes` in that case.


## Listing node pools

To list node pools, you can use:

```bash
kubectl get nodepools
```

In my case I have one node pool in my cluster, called `my-node-pool`, with 2 B2-7 nodes:

<pre class="console"><code>$ kubectl get nodepools
NAME            FLAVOR   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   MIN   MAX   AGE
nodepool-b7-2   b2-7     2         2         2            2           0     100   22d
</code></pre>

You can see the state of the node pool, how many nodes you want in the pool (`DESIRED`), how many actually are (`CURRENT`), how many of them are up-to-date (`UP-TO-DATE`) and how many are available to be used (`AVAILABLE`).

## Create a node pool

To create a new node pool, you simply need to create a new node pool manifest.

Copy the next YAML manifest in a `new-nodepool.yaml` file:

```yaml
apiVersion: kube.cloud.ovh.com/v1alpha1
kind: NodePool
metadata:
  name: my-new-node-pool
spec:
  desiredNodes: 3
  flavor: b2-7
  maxNodes: 100
  minNodes: 0
```

Then apply it to your cluster:

```bash
kubectl apply -f new-nodepool.yaml
```

Your new node pool will be created:


<pre class="console"><code>$ kubectl apply -f new-nodepool.yaml
nodepool.kube.cloud.ovh.com/my-new-node-pool created

$ kubectl get nodepools
NAME               FLAVOR   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   MIN   MAX   AGE
my-new-node-pool   b2-7     3                                            0     100   3s
nodepool-b7-2      b2-7     2         2         2            2           0     100   22d                                          0     100   24h
</code></pre>

At the beginning the new node pool is empty, but if you wait a few seconds, you will see how the nodes are progressively created and made available (one after another)...

<pre class="console"><code>$ kubectl get nodepools
NAME               FLAVOR   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   MIN   MAX   AGE
my-new-node-pool   b2-7     3         3         3            0           0     100   65s
nodepool-b7-2      b2-7     2         2         2            2           0     100   22d
</code></pre>


## Editing the node pool size

To upsize or downsize your node pool, you can use simply edit the YAML file and re-apply  it.
For example, raise the `desiredNodes` to 5 in `new-nodepool.yaml` and apply the file:

<pre class="console"><code>$ kubectl apply -f examples/new-nodepool.yaml
nodepool.kube.cloud.ovh.com/my-new-node-pool configured

$ kubectl get nodepools
NAME               FLAVOR   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   MIN   MAX   AGE
my-new-node-pool   b2-7     5         3         3            3           0     100   115m
nodepool-b7-2      b2-7     2         2         2            2           0     100   22d
</code></pre>

The `DESIRED` number of nodes has changed, and the two additional nodes will be created.

Then, after some minutes:

<pre class="console"><code>$ kubectl get nodepools
NAME               FLAVOR   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   MIN   MAX   AGE
my-new-node-pool   b2-7     5         5         5            3           0     100   117m
nodepool-b7-2      b2-7     2         2         2            2           0     100   22d

$ kubectl get nodepools
NAME               FLAVOR   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   MIN   MAX   AGE
my-new-node-pool   b2-7     5         5         5            5           0     100   120m
nodepool-b7-2      b2-7     2         2         2            2           0     100   22d
</code></pre>

You can also use `kubectl scale —replicas=X` to change the number of desired nodes. For example, let's resize it back to 2 nodes:

```bash
kubectl scale --replicas=2 nodepool my-new-node-pool
```


<pre class="console"><code>$ kubectl scale --replicas=2 nodepool my-new-node-pool
nodepool.kube.cloud.ovh.com/my-new-node-pool scaled

$ kubectl get nodepools
NAME               FLAVOR   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   MIN   MAX   AGE
my-new-node-pool   b2-7     2         5         5            5           0     100   129m
nodepool-b7-2      b2-7     2         2         2            2           0     100   22d
</code></pre>

Then, after some minutes:

<pre class="console"><code>$ kubectl get nodepools
NAME               FLAVOR   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   MIN   MAX   AGE
my-new-node-pool   b2-7     2         2         2            2           0     100   133m
nodepool-b7-2      b2-7     2         2         2            2           0     100   22d
</code></pre>


## Deleting a node pool

You can simply use `kubectl` to delete a node pool, as any other Kubernetes resource:

```bash
kubectl delete nodepool my-new-node-pool
```

## Go further

To have an overview of OVHcloud Managed Kubernetes service, you can go to the [OVHcloud Managed Kubernetes site](https://www.ovh.com/public-cloud/kubernetes/).

Otherwise to skip it and push to deploy your first application on your Kubernetes cluster, we invite you to follow our guide to [deploying an application](../deploying_an_application/deploying_an_application/).

Join our community of users on [https://community.ovh.com/en/](https://community.ovh.com/en/).
