---
layout: post
title: Office 365 Health Monitoring with PowerShell
date: 2016-2-2
author: Matt McNabb
comments: true
---

[Github]: http://github.com/mattmcnabb/O365ServiceCommunications
[Screen]: /assets/media/SampleMessage.png
[ServiceInfo]: /assets/media/SCServiceInfo.png
[Event]: /assets/media/SCEvent.png

If you manage an Office 365 tenant then you may be interested in a new module I published to the PowerShell Gallery. The O365ServiceCommunications module can be used to retrieve messages regarding your tenant health status, incident closure, and general information about planned downtime or new features. The Office 365 Message Center lacks email alerting on incidents and this is definitely a gap that needs to be filled. You can use this module to script this automatic alerting and make your boss happy! You could even drop the event data returned into a SQL database and generate reports and track the health of your Office 365 tenant.

This module uses the Office 365 Service Communications REST API and you'll need to be a global administrator for your tenant, or a delegated partner administrator in order to authenticate to the service.

### Installing the O365ServiceCommunications module

To get started with the O365ServiceCommunications module, you'll first need to install the module. If you have Powershell version 5.0 or have updated version 4.0 with the latest bits you can install the module using PowerShellGet by running this command:

{% highlight console %}
Find-Module O365ServiceCommunications | Install-Module
{% endhighlight %}

If you don't have PowerShellGet you can install the module by downloading directly from the [Github project site][Github]. Click on "Download ZIP" and then once the download has completed, right click the ZIP file and select "Unblock." Now extract the contents of the ZIP archive to a directory in your PowerShell module path (e.g. %Program Files%\WindowsPowerShell\Modules).

Once the module is installed, you can make it's commands available like this:

{% highlight console %}
Import-Module O365ServiceCommunications
{% endhighlight %}

### Authenticating

Before you can retrieve events from your tenant, you'll need to sign in to the REST API. Create a credential object first:

{% highlight console %}
$MyCred = Get-Credential
{% endhighlight %}

And then pass that credential to the New-SCSession command:

{% highlight console %}
$MySession = New-SCSession -Credential $MyCred
{% endhighlight %}

Notice that I assigned the output of the New-SCSession command to the $MySession variable - you'll use this later when you make subsequent calls to the API to retrieve service events.

### Getting service info

The next command you'll want to take a look at is Get-SCServiceInfo. This command will return information about the services that are available in your tenant - in other words, things that may have service incidents or maintenance messages.

{% highlight console %}
Get-SCServiceInfo -SCSession $MySession
{% endhighlight %}

Here you can see that the $MySession variable is passed in to the command. This sends along your authentication token to grant you access to the retrieve information from the REST API. Depending on the subscriptions you've purchased you'll see something like this:

{% highlight console %}
PS C:\> Get-SCServiceInfo -SCSession $MySession

ServiceName              FeatureNames
-----------              ------------
Exchange Online          {Sign-in, E-Mail and calendar access, E-Mail timely delivery, Management and Provisioning...}
Office Subscription      {Licensing and Renewal, Network Availability, Office Professional Plus Download}
Identity Service         {Sign-In, Administration}
Office 365 Portal        {Portal, Administration, Purchase and Billing}
Skype for Business       {Audio and Video, Federation, Management and Provisioning, Sign-In...}
SharePoint Online        {Provisioning, SharePoint Features, Tenant Admin, Search and Delve...}
Yammer Enterprise        {Yammer Components}
OneDrive for Business    {Service Health}
Mobile Device Management {Mobile Device Management}
Planner                  {Planner}
Sway                     {Sway}
Power BI                 {PowerBI.com}
{% endhighlight %}

### Getting events

Get-SCEvent is the real meat and potatoes of the O365ServiceCommunications module. It retrieves information for service incidents, maintenance announcements, and plain old messages about things like new features or items of concern for supporting your tenant. These are the same messages that you can see by going to the Message Center in your Office 365 administrative portal. You can run this just like the previous command - just pass in the session object:

{% highlight console %}
Get-SCEvent -SCSession $MySession
{% endhighlight %}

By default this will return only information about incidents:

{% highlight console %}
PS C:\> Get-SCEvent -SCSession $SCSession

StartTime             EndTime               ID      EventType Service           Status
---------             -------               --      --------- -------           ------
2/24/2016 12:00:00 PM 3/7/2016 6:32:00 PM   EX43875 Incident  Exchange Online   Post-incident report published
3/1/2016 12:00:00 AM                        EX43992 Incident  Exchange Online   Extended recovery
3/11/2016 7:00:00 AM  3/11/2016 8:10:00 AM  EX44123 Incident  Exchange Online   Service restored
3/10/2016 9:59:00 PM  3/10/2016 10:19:00 PM YA44110 Incident  Yammer Enterprise Service restored
3/10/2016 11:18:00 PM 3/10/2016 11:30:00 PM YA44111 Incident  Yammer Enterprise Service restored
{% endhighlight %}

To get other event types you can specify the -EventTypes parameter. This takes one or more of three arguments: Incident, Maintenance, and Message.

### How to use it

I have included a sample script in version 1.1 of the O365ServiceCommunications module that demonstrates how to generate an email alert when there are current service incidents that affect your tenant. It gathers data about the events and builds HTML tables that are then added to the body of an email message to be sent out to recipients of your choice:

![] [Screen]

This sample script should serve as a good jumping off point for learning how to consume the data from this module.

### For partners

If you're a delegated partner administrator of an Office 365 tenant, you can use the partner equivalent commands: Get-SCTenantServiceInfo and Get-SCTenantEvent. In this case you'll also need to specify the -Domains parameter with the domain name of the partner tenant you'd like to gather information for. You can still use the New-SCSession command to authenticate to the service first.

That covers getting started with the O365ServiceCommunications PowerShell module. I think this will be a great addition to your Office 365 management toolbelt and I hope you find it useful!
