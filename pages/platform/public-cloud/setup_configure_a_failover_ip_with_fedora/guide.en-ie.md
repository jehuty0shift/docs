---
title: Configure a failover IP with Fedora
excerpt: Configure a failover IP with Fedora
slug: configure_a_failover_ip_with_fedora
legacy_guide_number: g2045
---


## 
You may need to configure a failover IP address on your instances, for one of the following reasons:

- you have a large number of websites on your instance, 
- you host international projects.

In order to do this you can either buy or import a failover IP address for your Public Cloud instances. 

However failover IPs will not be automatically configured on your instance. 

This guide explains how to configure your instance's network interface so that a failover IP can be allocated to it.


## Requirements

- [Make sure you have launched an instance in your OVH customer account](../public-cloud-first-steps/)
- You must have a failover IP addresse sure you have launched an instance in your OVH customer account OVH[/url]
- You must have a failover IP address




## Configuring the interface

- Edit the configuration file with the following command:

```
vi /etc/sysconfig/network-scripts/ifcfg-ethX:Y
```



|# Variables|#Values|
|X|main interface number (usually eth0 )|
|xxx.xxx.xxx.xxx|failover IP to configure|
|Y|the number of the alias (starting at 0 and then 1 etc. depending on the number of IPs there are to configure)|


Add the following lines to the file:
code]DEVICE="ethX:Y"
BOOTPROTO=static
IPADDR="xxx.xxx.xxx.xxx"
NETMASK="255.255.255.255"
BROADCAST="xxx.xxx.xxx.xxx"
ONBOOT=yes[/code]


## Rebooting the network service

- Restart the network services with the command:

```
ifup ethX:Y
```





## 
[Migrating a failover IP](../migrating_a_failover_ip/)


## 
[Go back to the index of Cloud guides](../)

