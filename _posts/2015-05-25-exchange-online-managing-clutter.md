---
layout: post
title: Managing Clutter in Exchange Online
categories: [Office 365, Powershell]
---

[OfficeBlog]: http://blogs.office.com/2014/11/11/de-clutter-inbox-office-365/
[ByDefault]: http://blogs.office.com/2015/05/18/de-cluttering-everyones-inbox/
[Connect]: https://technet.microsoft.com/en-us/library/jj984289(v=exchg.150).aspx
[Github]: https://github.com/mattmcnabb/O365Admin
[Technet]: https://gallery.technet.microsoft.com/Disable-Clutters-For-All-41834444
[Clutter]: /assets/media/Get-Clutter_Error.png

### Exchange Online Clutter
Clutter is a new productivity aid in Exchange Online that helps save you time by separating your important messages from the rest of the muck that you get on a daily basis. A description of how this feature works can be found [here] [OfficeBlog]. Clutter is optional and can be enabled by users should they choose to use the service.

Microsoft believes in Clutter so much, however, that they have decided to make it [available by default] [ByDefault] on all new Exchange Online mailboxes starting in June. If this timeline doesn't work for your organization, or you would rather give your users the choice to turn Clutter on, there are some new Powershell cmdlets that you can leverage.

First, some prerequisites:

1. You need to have an account that has at least the Exchange Service Administrator role in Office 365 and the ability to connect to Exchange Online with Powershell. Instructions to connect can be found [here] [Connect]. I also have a [module on Github] [Github] that can make working with Exchange Online and other Office 365 services a bit easier.

2. The Clutter cmdlets will only work on server version 15.1 (Build 166.22). To manage Clutter with Powershell, both the server that you connect to via Powershell and the server that the mailboxes you are managing must be updated to this version. Microsoft is working to make sure that all Exchange Online servers are updated as quickly as possible, but you may still have some mailboxes that are on a prior version. If this is the case you will get an error when targeting those mailboxes with the Clutter cmdlets:

![][Clutter]

### Turning Clutter Off
There is currently no way to disable Clutter from being on by default for your entire tenant - you will have to do this for all your mailboxes now and for any new mailboxes that you don't want to have Clutter turned on for.  You can disable Clutter manually for a mailbox by running the `Set-Clutter` cmdlet:

{% highlight Powershell %}
Get-Mailbox 'Abe Lincoln' | Set-Clutter -Enable $false
{% endhighlight %}

Or you can disable it for all mailboxes:

{% highlight Powershell %}
Get-Mailbox -Filter * -ResultSize Unlimited | Set-Clutter -Enable $false
{% endhighlight %}

However, some of your users may have already enabled Clutter and you won't want to disable it for them. In this case you will want to filter your mailbox results to only the mailboxes that don't already have Clutter enabled:

{% highlight Powershell %}
Get-Mailbox -Filter * -ResultSize Unlimited | Foreach {
    $DN = $_.DistinguishedName
    $Status = Get-Clutter -Identity $DN | Select -ExpandProperty isEnabled
    if (!$Status)
    {
        Set-Clutter -Identity $DN -Enable $false
    }
}
{% endhighlight %}

You can fully automate disabling Clutter for your current mailboxes using [this script] [Technet]. Make sure you unblock the script and then run:

{% highlight Powershell %}
Get-Help .\DisableClutterOnByDefault.ps1 -ShowWindow
{% endhighlight %}

for details on how to use the script.

### Moving Forward
After you have run the above commands to disable the Clutter for the desired mailboxes, you'll need a plan to sustain this policy for future mailboxes. If you automate mailbox creations with Powershell, then you'll need to run `Set-Clutter` against each one to disable Clutter by default. At some point you may want to embrace the on by default strategy and make your users aware of the change. At that point you can just allow new mailboxes to have this enabled.

I hope this post has clarified the administrative controls for Clutter and will help your organization make it's decision around this new feature.