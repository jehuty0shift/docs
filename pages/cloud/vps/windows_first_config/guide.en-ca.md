---
title: 'Configuring your Windows server for the first time'
slug: windows-first-config
excerpt: 'This guide will show you which settings need to be changed to re-enable remote desktop connection and ICMP'
section: 'Getting started'
---

**Last updated 2019/06/15**

## Objective

When you install Windows Server on a [VPS](https://www.ovhcloud.com/en-ca/vps/){.external}, the connection to your remote desktop can sometimes be disabled, as can the ICMP protocol response.

**This guide will show you which settings need to be changed to re-enable remote desktop connection and ICMP.**

## Requirements

* a [VPS](https://www.ovhcloud.com/en-ca/vps/){.external} with Windows Server installed
* access to the OVH [Control Panel](https://ca.ovh.com/auth/?action=gotomanager){.external}

## Instructions

### Step 1: Log in to the KVM

To access the KVM of your VPS, please follow the [VPS KVM guide](https://docs.ovh.com/ca/en/vps/use-kvm-for-vps/){.external}

### Step 2: Configure Windows settings

In the first setup screen, you will need to configure the settings for your **Country/region**, **Language**, and **Keyboard layout**. When you have done this, click `Next`{.action}.

![KVM](images/setup-03.png){.thumbnail}

Next, choose a password for your administrator account. Enter it twice, then click `Finish`{.action}.

![KVM](images/setup-04.png){.thumbnail}

Windows will now apply your settings. When this is done, you will see the following screen. At this point you will need to click the `Send CtrlAltDel`{.action} button to sign in.

![KVM](images/setup-05.png){.thumbnail}

On the sign-in screen, enter the password you created for your Administrator account and hit the `Enter`{.action} key on your keyboard.

![KVM](images/setup-06.png){.thumbnail}

### Step 3: Modify Windows Firewall

Once the setup is complete and you have signed in, go to `Administrative Tools`{.action}, then `Windows Firewall with advanced security`{.action}.

![Admin](images/windows4.png){.thumbnail}

You will then need to enable the ICMP and the remote desktop connection rules *(right-click -> Enable rule)*.

![Enabled](images/windows5.png){.thumbnail}

Your server will now be set up for remote desktop connections.

## Go further

Join our community of users on <https://community.ovh.com/en/>.
