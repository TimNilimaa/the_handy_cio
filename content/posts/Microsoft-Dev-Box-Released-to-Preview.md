---
title: "Microsoft Dev Box Released to Preview"
date: 2022-09-22T06:45:27+02:00
draft: false
toc: false
images:
tags:
  - tooling
  - 'developer enablement'
  - 'workplace'
---

Microsoft recently announced the preview of their development-machine-as-a-service, called [Dev Box](https://learn.microsoft.com/en-us/azure/dev-box/overview-what-is-microsoft-dev-box). While we are not there yet, my crew and I have been looking at tooling such as [gitpod](https://www.gitpod.io) and vscode server [1](https://code.visualstudio.com/blogs/2022/07/07/vscode-server), [2](https://github.com/gitpod-io/openvscode-server) to provide easy access to a coding environment. We've successfully been hitting one or another reason why those solutions wasn't feasable for us. Being that we used full (read old) .NET Framework, having dependency that we had a thight coupling to or one of Jupiters moon hadn't risen. Regardless of the reason, this Dev Box could be a stepping stone getting there - or a complete alternative.

Looking at how to create a Dev Box of your own, it is quite easy (but not always that straight forward) following [their guide](https://learn.microsoft.com/en-us/azure/dev-box/quickstart-configure-dev-box-service?tabs=AzureADJoin).
Just as with many other products coming from the big software giant from the north west of US of A, Dev Box integrates neatly into other services such as Azure Active Directory, Intune and more. You'll probably be up and running your machine in a matter of minutes (plus the initial boot time for a Dev Box of 30-90 minutes that is).

Once you are connected to your first machine you'll notice that it isn't that much more than a branded set of Windows 365. Don't get me wrong, it sure is a really neat thing, but in both good and bad it is way much more than gitpod or open vscode server. What you and your project needs will probably be the answer to what service you'll be using.