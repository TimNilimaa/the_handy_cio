---
title: "Letsencrypt Microk8s"
date: 2022-03-02T11:58:25+01:00
draft: true
---

Install helm if you don't already have it installed on your machine.
{{< highlight bash >}}
sudo snap install helm --classic
{{< / highlight >}}

There is a great tutorial, that I sadly found after writing the below part, located at https://cert-manager.io/docs/installation/helm/ Here I try to keep it all in one place.
Add jetstack, and update the repo cache
{{< highlight bash >}}
helm repo add jetstack https://charts.jetstack.io
helm repo update
{{< / highlight >}}
Install cert-manager with CustomResourceDefinitions. Make sure to get the latest version by checking in on https://artifacthub.io/packages/helm/cert-manager/cert-manager
{{< highlight bash >}}
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.7.1/cert-manager.crds.yaml
{{< / highlight >}}
