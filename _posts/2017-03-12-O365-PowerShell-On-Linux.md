---
layout: post
title: Connect to Office 365 with PowerShell on Linux
categories: [PowerShell]
author: Matt McNabb
comments: true
---

[ConnectExchange]: https://technet.microsoft.com/en-us/library/jj984289(v=exchg.160).aspx
[ConnectCompliance]: https://technet.microsoft.com/en-us/library/mt587092(v=exchg.160).aspx
[ProgBar]: /assets/media/O365linuxProgBar.png

This Wednesday Microsoft announced the release of PowerShell 6.0 Alpha 17. One new feature in particular intrigued, and that's the capability to [connect to custom remoting configurations](https://github.com/PowerShell/psl-omi-provider/releases/tag/v1.0.0.18). This opens up the possibility of connecting to an Exchange endpoint, including Office 365. I manage Office 365 on a daily basis so I decided to give this a try from Linux just to see how and if it works. My set up for testing is a Hyper-V VM with Ubuntu 16.04 installed, and I installed PowerShell directly from the [Github releases page](https://github.com/PowerShell/PowerShell/releases/tag/v6.0.0-alpha.17).

Here's how you can create an implicit remoting connection to Exchange Online:

```powershell
PS /home/matt> $O365Credential = Get-Credential
PS /home/matt> $Session = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri https://outlook.office365.com/powershell-liveid/ -Credential $UserCredential -Authentication Basic -AllowRedirection
PS /home/matt> Import-PSSession -Session $Session
```

You'll see a funky progress bar detailing the commands being imported from the remoting session:

![][ProgBar]

Now you can run any commands from the Exchage Online module:

```powershell
PS /home/matt> get-mailbox 'matt'                                                  

Name                      Alias                ServerName       ProhibitSendQuota                  
----                      -----                ----------       -----------------                  
Matthew McNabb            matt                 bn3pr07mb2755    49.5 GB (53,150,220,288 bytes)
```

And the process for connecting to the Compliance Center is very similar:

```powershell
PS /home/matt> $Session = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri https://ps.compliance.protection.outlook.com/powershell-liveid/ -Credential $UserCredential -Authentication Basic -AllowRedirection
PS /home/matt> Import-PSSession -Session $Session
```

> NOTE: Note that I've added the prefix "CC" to the commands imported from the Compliance Center - that's because some of the included command names overlap with existing commands in the Exchange Online module.

Now in the release notes linked above the author states that this will allow connections to Office 365 from Linux, but that's not strictly true. It will allow connections to Exchange Online and the Compliance Center since both of these service rely on implicit remoting to an Exchange endpoint. But Office 365 is more than just Exchange - on Windows we can also use PowerShell to manage Azure Active Directory, Sharepoint, and Skype For Business. These other services rely on installers or assemblies that only work in Windows, so for now I think we'll only be able to use PowerShell to connect to the administrative services that are supported strictly with implicit remoting. Hopefully this will improve when PowerShell 6.0 is released and we see more folks using PowerShell for administration on Linux or OSX.

Time for a shameless plug: You can check out my module for making it easier to connect to all Office 365 services from PowerShell by running this command:

```powershell
PS C:> Install-Module O365Extensions
```

This module currently supports connections to Exchange,Azure AD, Sharepoint, Compliance Center, and Skype using a simple connection command. Support for Linux and OSX is coming soon, along with some other features that may make your day-to-day Office 365 management much easier.
