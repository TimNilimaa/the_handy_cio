---
title: Running microk8s on Intel NUC with Synology NAS
date: 2022-03-02T09:55:56+01:00
draft: true
toc: false
images: null
tags:
  - k8s
  - microk8s
  - synology-csi
  - synology
  - intel-nuc
---
## Executive summary

For small environments or environments used for test and development, a few (relatively) cheap Intel NUCs together with one or two Synology NAS can be used for a High Availability Kubernetes cluster. This environment is not configured for production usage due to a couple of reasons such as no security configurations are done here nor is this tested with dual Synology NAS products - make sure to consult with an expert and/or use a hosted solution such as Azure, Google, AWS, Safespring or ELASTX.

## Instructions

In this guide I used Ubuntu server 20.04 LTS. I had three Intel NUC of varying age that I could use and cramed them up with the most RAM that I could find laying around.

### Environment

During installation of the OS on the NUCs, I could configure them to use VLAN tags in order to provide both an internal network between them, access to the LAN as well as 'direct' access to Internet (just a little tiny firewall inbetween for at lest some security).

### Install microk8s

Ubuntu have a pretty good structure for how to install microk8s, after all they are the ones that built it. Sadly, some steps are getting a bit out of date but also out of sync with each other. As with all documentation regarding technology that is progressing as quickly as Kubernetes, this post too will become stale. I'm hoping that I will be able to give accounts for what things are working when.
One thing that I've done before I started was to set up host records on each node to make sure they could communicate with each other, especially since I didn't have a local DNS system in place. As mentioned, I also wanted the communication between the nodes on a dedicated VLAN, so I created the host names pointing to the IP addresses on that subnet.

#### First node

During my tests, I started out with 1.21/stable of Kubernetes, but then later on upgraded to 1.23/stable and saw that both versions worked. So here is the command to install the first node.
{{< highlight bash >}}
sudo snap install microk8s --classic --channel=1.23/stable
{{< / highlight >}}
Make sure it is up and running by waiting for it to be ready
{{< highlight bash >}}
microk8s status --wait-ready
{{< / highlight >}}
And then, fix so you don't have to sudo for kubectl
{{< highlight bash >}}
sudo usermod -a -G microk8s ubuntu
sudo chown -f -R ubuntu ~/.kube
{{< / highlight >}}
I also made sure that DNS where enabled
{{< highlight bash >}}
microk8s enable dns
{{< / highlight >}}
For more information, read here: <https://ubuntu.com/tutorials/install-a-local-kubernetes-with-microk8s#1-overview>

#### Nodes 2-3

To join additional nodes, run the following command on the primary node. It will tell you what to run on the next nodes.
Taken from the Ubuntu web site, again they do have a decent instruction for this.
{{< highlight bash >}}
Join node with:
microk8s join ip-172-31-20-243:25000/DDOkUupkmaBezNnMheTBqFYHLWINGDbf

If the node you are adding is not reachable through the default interface you can use one of the following:
microk8s join 10.1.84.0:25000/DDOkUupkmaBezNnMheTBqFYHLWINGDbf
microk8s join 10.22.254.77:25000/DDOkUupkmaBezNnMheTBqFYHLWINGDbf
{{< / highlight >}}
So just run one of those commands on the other node, and then do the very same for the additional node(s).

### Storage

The storage unit that I went with is an old but common Synology NAS. Just like the Intel NUC this isn't a very expensive peice of hardware nor is it production ready. From what I've read on other blogs, it might be possible to use dual NICs on the device as well as multiple devices to achieve high availability. Or you know, not host things like this on consumer grade products for production :)

There are also more ways to secure this, but this wasn't so bad that I couldn't have this up for a week while trying if this worked and learned more about k8s. The alternative were to only use one node and local storage, but hey there is no fun in that.

#### Configure NAS

#### Synology CSI

##### Clone repo

##### Adjust for microk8s

### Verify/Test
