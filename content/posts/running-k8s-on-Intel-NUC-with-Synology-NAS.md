---
title: Running microk8s on Intel NUC with Synology NAS
date: 2022-03-02T09:55:56+01:00
draft: false
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

In this guide I used Ubuntu server 20.04 LTS. I had three Intel NUC of varying age that I could use and crammed them up with the most RAM that I could find laying around.

### Environment

During installation of the OS on the NUCs, I could configure them to use VLAN tags in order to provide both an internal network between them, access to the LAN as well as 'direct' access to Internet (just a little tiny firewall in between for at lest some security).

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
sudo microk8s status --wait-ready
{{< / highlight >}}
And then, fix so you don't have to sudo for kubectl
{{< highlight bash >}}
sudo usermod -a -G microk8s <username> 
sudo chown -f -R <username> ~/.kube
newgrp microk8s
{{< / highlight >}}
I also made sure that DNS were enabled
{{< highlight bash >}}
microk8s enable dns
{{< / highlight >}}
For more information, read here: <https://ubuntu.com/tutorials/install-a-local-kubernetes-with-microk8s#1-overview>

#### Nodes 2-3

To join additional nodes, run the following command on the primary node. It will tell you what to run on the next nodes.
Taken from the Ubuntu web site, again they do have a decent instruction for this.
You run the following command on the first node.
{{< highlight bash >}}
microk8s add-node
{{< / highlight >}}
It will tell you exactly what to do, it will look something like this
{{< highlight bash >}}
From the node you wish to join to this cluster, run the following:
microk8s join 192.168.112.172:25000/d0fe312984a43a8fcbeccbbe5b6dd843/8f8557f74cc9

Use the '--worker' flag to join a node as a worker not running the control plane, eg:
microk8s join 192.168.112.172:25000/d0fe312984a43a8fcbeccbbe5b6dd843/8f8557f74cc9 --worker

If the node you are adding is not reachable through the default interface you can use one of the following:
microk8s join 192.168.112.172:25000/d0fe312984a43a8fcbeccbbe5b6dd843/8f8557f74cc9
microk8s join 192.168.112.220:25000/d0fe312984a43a8fcbeccbbe5b6dd843/8f8557f74cc9
{{< / highlight >}}
So just run one of those commands on the other node, and then do the very same for the additional node(s).

### Storage

The storage unit that I went with is an old but common Synology NAS. Just like the Intel NUC this isn't a very expensive piece of hardware nor is it production ready. From what I've read on other blogs, it might be possible to use dual NICs on the device as well as multiple devices to achieve high availability. Or you know, not host things like this on consumer grade products for production :)

There are also more ways to secure this, but this wasn't so bad that I couldn't have this up for a week while trying if this worked and learned more about k8s. The alternative were to only use one node and local storage, but hey there is no fun in that.

#### Configure NAS

Your Synology NAS must be configured with an account that have full permissions to use iSCSI.

#### Synology CSI

The folks over at Synology have made an open source project for their CSI and hosts it a GitHub. Their latest releases have been much better compared to the original version that they released. One of the issues, for us running this with microk8s, is that it doesn't support microk8s. But a friendly pull request should solve that. Any day now.

##### Clone repo

My suggestion is to clone the repository, but you could ofcourse simply download a copy if you prefer that. This benefit of cloning is that with two simple commands you can get the updated fixes ('git fetch' and 'git pull').
To clone the repository, execute the following command.
{{< highlight bash >}}
git clone https://github.com/SynologyOpenSource/synology-csi.git
{{< / highlight >}}

##### Adjust for microk8s

While it probably should be possible to modify each node with a symlink to make the microk8s paths appear at the correct locations for a 'normal' Kubernetes installation, I never got that working in my testings, so I ended up modifying the CSI files.

There are five paths that must be updated in node.yaml 

node.yaml
{{< highlight yaml >}}
apiVersion: v1
kind: ServiceAccount
metadata:
  name: csi-node-sa
  namespace: synology-csi

---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: synology-csi-node-role
rules:
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get", "list"]
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "update"]
  - apiGroups: [""]
    resources: ["namespaces"]
    verbs: ["get", "list"]
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["volumeattachments"]
    verbs: ["get", "list", "watch", "update"]

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: synology-csi-node-role
  namespace: synology-csi
subjects:
  - kind: ServiceAccount
    name: csi-node-sa
    namespace: synology-csi
roleRef:
  kind: ClusterRole
  name: synology-csi-node-role
  apiGroup: rbac.authorization.k8s.io

---
kind: DaemonSet
apiVersion: apps/v1
metadata:
  name: synology-csi-node
  namespace: synology-csi
spec:
  selector:
    matchLabels:
      app: synology-csi-node
  template:
    metadata:
      labels:
        app: synology-csi-node
    spec:
      serviceAccount: csi-node-sa
      hostNetwork: true
      containers:
        - name: csi-driver-registrar
          securityContext:
            privileged: true
          imagePullPolicy: Always
          image: k8s.gcr.io/sig-storage/csi-node-driver-registrar:v2.3.0
          args:
            - --v=5
            - --csi-address=$(ADDRESS)                         # the csi socket path inside the pod
            - --kubelet-registration-path=$(REGISTRATION_PATH) # the csi socket path on the host node
          env:
            - name: ADDRESS
              value: /csi/csi.sock
            - name: REGISTRATION_PATH
              value: /var/snap/microk8s/common/var/lib/kubelet/plugins/csi.san.synology.com/csi.sock
            - name: KUBE_NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
          volumeMounts:
            - name: plugin-dir
              mountPath: /csi
            - name: registration-dir
              mountPath: /registration
        - name: csi-plugin
          securityContext:
            privileged: true
          imagePullPolicy: IfNotPresent
          image: synology/synology-csi:v1.0.1
          args:
            - --nodeid=$(KUBE_NODE_NAME)
            - --endpoint=$(CSI_ENDPOINT)
            - --client-info
            - /etc/synology/client-info.yml
            - --log-level=info
          env:
            - name: CSI_ENDPOINT
              value: unix://csi/csi.sock
            - name: KUBE_NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
          volumeMounts:
            - name: kubelet-dir
              mountPath: /var/snap/microk8s/common/var/lib/kubelet
              mountPropagation: "Bidirectional"
            - name: plugin-dir
              mountPath: /csi
            - name: client-info
              mountPath: /etc/synology
              readOnly: true
            - name: host-root
              mountPath: /host
            - name: device-dir
              mountPath: /dev
      volumes:
        - name: kubelet-dir
          hostPath:
            path: /var/snap/microk8s/common/var/lib/kubelet
            type: Directory
        - name: plugin-dir
          hostPath:
            path: /var/snap/microk8s/common/var/lib/kubelet/plugins/csi.san.synology.com/
            type: DirectoryOrCreate
        - name: registration-dir
          hostPath:
            path: /var/snap/microk8s/common/var/lib/kubelet/plugins_registry
            type: Directory
        - name: client-info
          secret:
            secretName: client-info-secret
        - name: host-root
          hostPath:
            path: /
            type: Directory
        - name: device-dir
          hostPath:
            path: /dev
            type: Directory
{{< / highlight >}}

To get Microsoft SQL Server working, I also had to change storage-class.yaml to use btrfs
{{< highlight yaml >}}
fsType: 'btrfs'
{{< / highlight >}}

You'll find both files in the 'deploy/kubernetes/v1.20/' path.