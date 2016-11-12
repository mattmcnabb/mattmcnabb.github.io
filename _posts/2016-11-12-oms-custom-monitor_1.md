---
layout: post
title: Creating a Custom Monitoring Solution with Microsoft Operations Management Suite - Part 1
date: 2016-10-30
categories: [Powershell, OneLogin, OMS]
author: Matt McNabb
comments: true
---

[OneLoginPSGallery]: https://www.powershellgallery.com/packages/OneLogin/
[OMSDataCollector]: https://azure.microsoft.com/en-us/documentation/articles/log-analytics-data-collector-api/
[OMSDataInjection]: https://www.powershellgallery.com/packages/OMSDataInjection/

## Part 1

## Homebrew logging

My organization has been using OneLogin for quite some time now and it's a great web identity management platform. Recently I've been tasked with figuring out how to gain better visibility into our OneLogin account for security and troubleshooting. To this end I created the [OneLogin PowerShell module][OneLoginPSGallery] that works against OneLogin's REST API. Using this module I can get retrieve event logs from the OneLogin service which is a great start, but to really take advantage of this power I needed a way to track and analyze the events. A SIEM product is a natural fit for this, and we just happen to be evaluating Microsoft's Operations Management Suite (OMS), particularly the Log Analytics feature. In this series I'll show you how I achieved our goal using PowerShell and OMS.

### The OneLogin PowerShell module

To install the OneLogin PowerShell module from the PowerShell Gallery, run:

```powershellgallery
Install-Module OneLogin
```

This module can be used to automate actions in your OneLogin account, such as creating users, adding roles, attributes, and most importantly for this article, retrieving events. OneLogin records an event for just about every action that takes place - logins, administrative actions, directory connector actions, etc. You can use these events to gain insight into your account and see how your users are taking advantage of applications, when something is going wrong, and potentially even detect risky activity. Here is some example code that will gather OneLogin events from the past hour:

{% gist 1e8585b12c7b0ef8b27fc6949ceb6bbd 1.ps1 %}

### The OMSDataInjection module

I knew that I wanted to record and analyze OneLogin events in OMS, but was unsure how to get the events into the service. OMS has an http listener for posting data, and OneLogin has a broadcast feature that can feed events to any web listener, so it seemed that this would be a no-brainer. However, [OMS's Data Collector API][OMSDataCollector] authorization mechanism is relatively complex, requiring current timestamp information and a dynamically generated authorization signature. OneLogin's broadcaster feature did not allow this sort of manipulation of request headers, so I decided on using a custom feed script to post data to the API.

So I set out creating a module for the OMS Data Collector API so that once I had figured out how to post this data, I could potentially reuse this approach with other types of data. No sooner had I begun to work on this module than I found out that Tao Yang had already done my work for me and published the excellent [OMSDataInjection module][OMSDataInjection]! 

```powershell
Install-Module OMSDataInjection
```

And here's a sample script that will post all the OneLogin events gathered for the past hour and post them to OMS:

{% gist 1e8585b12c7b0ef8b27fc6949ceb6bbd 2.ps1 %}

Note that you'll need to provide your OMS workspace ID, and the primary and secondary keys in order to authenticate to the Data Collector API.

### Up next...
This is the first in a series on using PowerShell to send data to your OMS workspace. In upcoming articles I'll show you how to create custom views based on that data and how to alert on log entries.
