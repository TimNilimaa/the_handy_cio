---
title: "Azure DevOps Cannot Connect to k8s Cluster Version 1.24"
date: 2022-10-02T16:37:05+02:00
draft: true
toc: false
images:
tags:
  - k8s
  - 'azure devops'
---

If you have a k8s cluster running or, if you have recently upgraded to, version 1.24 you'll notice that you cannot use the wizard to connect Azure DevOps to the k8s cluster. One example would be during the creation of an [Environment in Azure DevOps](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/environments?view=azure-devops).
What is actually being created in the background is a [Service Connection](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml#kubernetes-service-connection) to the Kubernetes cluster. Depending on the configuration of your cluster, the permissions being used by the Service Connection differ.
For clusters using [Azure RBAC](https://learn.microsoft.com/en-us/azure/role-based-access-control/), a Service Account is created in the namespace in question and a RoleBinding with permission to just that namespace. Reasonable. However, if you are using a cluster without Azure RBAC, then the very same thing happen however the permissions will be created cluster-wide so that the Service Account have permissions to all namespaces. None of this really matters to the problem, however it is good to know the backstory.

On a cluster pre-1.24, these Service Accounts are created with a Secret per default. This is not the case in 1.24 where the Secret must be created explicitly. Azure DevOps sadly does not handle this.

In order to get Azure DevOps connected to a Kubernetes cluster running 1.24, such as an AKS cluster running 1.24.3 which is the latest supported k8s version in Azure (at the time of writing), one have to create a generic k8s connection in Azure DevOps. All fine and dandy, especially since there is a nice guide to it in the UI - it is just the fact that the guide is only half the story.