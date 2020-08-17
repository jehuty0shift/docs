---
title: Managing nodes and node pools with OVHcloud API
slug: managing-nodes-with-api
excerpt: ''
section: User guides
order: 1
---

**Last updated July 27<sup>th</sup> July, 2020.**

## Objective

OVHcloud Managed Kubernetes service provides you Kubernetes clusters without the hassle of installing or operating them. This guide will cover one of the first steps after ordering a cluster: managing nodes and node pools, using the OVHcloud API.

In this guide, we are assuming you're using the [OVHcloud API](https://api.ovh.com/) to manage your Kubernetes cluster. If you are using a different method, like the [OVHcloud Cloud Manager](https://www.ovh.com/manager/cloud/), please refer to the relevant documentation: [Managing nodes and node pools](../managing-nodes/) guide.

## Requirements

- An OVHcloud Managed Kubernetes cluster

## Nodes and node pools

In your OVHcloud Managed Kubernetes cluster, nodes are grouped in node pools (group of nodes sharing the same configuration).  

When you create your cluster, it's created with a default node pool. Then, you can modify the size of this node pool, or add additional node pools of different sizes and types.

Upon creation, node pool is defined by its name (`name`), the type of instance within our available catalog (`flavor`), the number of identical nodes that you want in that node pool (`desiredsize`), and potentially self-defined boundaries to limit the value of desired nodes (`minnodecount` and `maxnodecount`). 

After creation, the `desiredsize` can be edited, and the node pool will automatically be resized to accommodate this new value. `minnodecount` and `maxnodecount` can also be edited at any time.

We will later propose cluster autoscaling based on node pols. We see some customer developing they own autoscaling scripts. We strongly encourage you to define `minnodecount` and `maxnodecount` in that case.

In this guide we explain how to do some basic operations with nodes and node pools using the [OVHcloud API](https://api.ovh.com/): adding nodes to an existing node pool, creating a new node pool...


## Upsizing and downsizing a Node Pool

By editing the `desiredNodes` value to a value different from `currentNodes` one (for values between the `minnodecount` and `maxnodecount`), i will trigger the node pool upsizing or downsizing.

When downsizing, the last created node will be drained then deleted first, then the before last created is drain and deleted, and this process is reproduced until the "current nodes" is equal to `desirednodes`.

When upsizing, a first node is created and when it is ready, another is created, and this process is reproduced until the `currentnodes` is equal to `desirednodes`.

## The API Explorer

To simplify things, we are using the [API Explorer](https://api.ovh.com/console/), which allows to explore, learn and interact with the API in an interactive way.

Log in to the API Explorer using your OVH NIC.

![Log in to the API Explorer](images/kubernetes-quickstart-api-ovh-com-001.png){.thumbnail}

If you go to the [Cloud section](https://api.ovh.com/console/#/cloud) of the API Explorer, you will see the available `/cloud/project/{serviceName}/kube` endpoint.


## List your OVHcloud Managed Kubernetes clusters

The `GET /cloud/project` API endpoint lists all the available Public Cloud Services associated to your OVHcloud account:

![List your OVHcloud Public Cloud Services](images/kubernetes-quickstart-api-ovh-com-003.png){.thumbnail}

Choose the Public Cloud Service corresponding to your OVHcloud Managed Kubernetes. In this example, we will refer to it as `serviceName`

The `GET /cloud/project/{serviceName}/kube` API endpoint lists all the available clusters in your chosen project:

![List your OVHcloud Managed Kubernetes clusters](images/kubernetes-quickstart-api-ovh-com-004.png){.thumbnail}

By calling it, you can view a list of your Kubernetes clusters ID. Note down the ID of the cluster you want to use. In this example, we will refer to it as `kubeId`


## Getting your cluster information

The `GET  /cloud/project/{serviceName}/kube/{kubeId}` API endpoint provides important information about your OVHcloud Managed Kubernetes cluster, including its status and URL.

![Getting your cluster information in the OVHcloud API](images/kubernetes-quickstart-api-ovh-com-005.png){.thumbnail}


## Listing node pools


The `GET /cloud/project/{serviceName}/kube/{kubeId}/nodepool` API endpoint lists all the available node pools:

![Listing node pools](images/kubernetes-quickstart-api-ovh-com-006.png){.thumbnail}


## Create a node pool

Use the `POST /cloud/project/{serviceName}/kube/{kubeId}/nodepool` API endpoint to create a new node pool:

You will need to give it a `flavorName` parameter, with the flavor of the instance you want to create. For this tutorial choose a general purpose node, like the `b2-7` flavor.

If you want your node pool to have at least one node, set the `desiredSize` to a value above 0.

The API will return you the new node pool information.

![Create a node pool](images/kubernetes-quickstart-api-ovh-com-007.png)


## Get information on a node pool

Use the `GET  /cloud/project/{serviceName}/kube/{kubeId}/nodepool/{nodePoolId}` API endpoint to get information on a specific node pool:

![Get information on a node pool](images/kubernetes-quickstart-api-ovh-com-008.png) 


## Editing the node pool size

To upsize or downsize your node pool, you can use the `POST /cloud/project/{serviceName}/kube/{kubeId}/nodepool/{nodePoolId}` API endpoint, and set the `desiredSize` to the new pool size (between `minnodecount` and `maxnodecount`, that you can modify if needed):

![Editing the node pool size](images/kubernetes-quickstart-api-ovh-com-009.png) 



## Deleting a node pool

To delete a node pool, use the `/DELETE /cloud/project/{serviceName}/kube/{kubeId}/nodepool/{nodePoolId}` API endpoint:

![Deleting a node pool](images/kubernetes-quickstart-api-ovh-com-010.png) 


## Go further

To have an overview of OVHcloud Managed Kubernetes service, you can go to the [OVHcloud Managed Kubernetes site](https://www.ovh.com/public-cloud/kubernetes/).

Otherwise to skip it and push to deploy your first application on your Kubernetes cluster, we invite you to follow our guide to [configuring default settings for `kubectl`](../configuring_default_settings_for_kubectl/) and [deploying an application](../deploying_an_application/deploying_an_application/) .

Join our community of users on [https://community.ovh.com/en/](https://community.ovh.com/en/).
