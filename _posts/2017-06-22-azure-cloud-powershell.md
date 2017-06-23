---
layout: post
title: PowerShell-As-A-Service
tags: [PowerShell, Azure]
author: Matt McNabb
comments: true
---

Today I received an invitation to preview PowerShell in Azure Cloud Shell (PSCloudShell). Azure Cloud Shell is a new feature added to Azure earlier this year which allowed us to run a Bash console inside our web browser to manage our Azure resources. Microsoft has just added PowerShell to this as well and I'm already having a blast poking around to see how this works and imagining all the possibilities.

When I first launched the PSCloudShell interface I noticed it took quite a while to get up and running. You're required to set up a storage account to store all the necessary artifacts you need to run PowerShell such as modules, scripts, etc. Once the shell launches for the first time you'll see a familiar prompt, with the location set to "Azure."

![](/assets/img/CloudPS.png)

<!--more-->

This is interesting, and Get-ChildItem reveals a bit more about how this works:

![](/assets/img/CloudPS2.png)

Notice what is returned? These are my Azure subscription names that appear to be folders in my current directory. Turns out what we're dealing with is an Azure provider as evidenced here:

![](/assets/img/CloudPS3.png)

This is incredible! Using PSCloudShell I can now access my Azure resources directly just by signing in to the Azure Portal, and I can navigate through everything just like it's a file system!

`Get-PSDrive` shows us that there are other drives we'd expect to see as well:

![](/assets/img/CloudPS4.png)

Turns out this is all happening in a Windows VM in Azure, and I can CD right into the C: drive.

I can also manage more than just Azure resources in PSCloudShell. Here you'll see that I can install modules from the PowerShell Gallery and potentially perform any arbitrary actions I can think of.

![](/assets/img/CloudPS5.png)

This is a very early preview, and I'm sure there will be much development poured into this, but this is a great start and I'm excited to see where this leads!