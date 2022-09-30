---
title: "Ingress Traffic Not Working After AKS Upgrade 1.22 to 1.24"
date: 2022-09-30T05:59:25+02:00
draft: false
toc: false
images:
tags:
  - nginx
  - aks
---

We recently upgraded one of our AKS (k8s in Azure) clusters from 1.22 through 1.23 to 1.24. This is easily done from the web GUI in Azure Portal. One thing that ended up us, our normal go-to people as well as a support ticket to Microsoft before we found the issue (ourselves...), was that for an yet unknown reason ingress traffic stopped working.
This was most noticeable for, well for users trying to access services hosted by that cluster, but for us in the management plane when looking at the Load Balancer in Azure Portal as we could see that traffic wasn't coming through to the backend servers. There is a great UI there developed by  Microsoft if you hit the Insights tab while looking at the Load Balancer.

We ended up changing the following line
{{< highlight yaml >}}
  externalTrafficPolicy: Cluster
{{< /highlight >}}
to
{{< highlight yaml >}}
  externalTrafficPolicy: Local
{{< /highlight >}}
by executing the following command
{{< highlight bash >}}
kubectl edit services nginx-ingress-ingress-nginx-controller -n ingress-basic
{{< /highlight >}}

The following articles helped us figure out the issue [1](https://github.com/Azure/AKS/issues/2908) and [2](https://github.com/kubernetes/ingress-nginx/issues/8501#issuecomment-1108428615)