---
layout: post
title: Managing Office 365 User Licenses with PowerShell - Part 3
categories: [Powershell, Office 365]
author: Matt McNabb
comments: true
---

[Part1]: /Office-365-Licensing_1
[Part2]: /Office-365-Licensing_2
[Part3]: /Office-365-Licensing_3
[Part4]: /Office-365-Licensing_4
[Select]: https://technet.microsoft.com/en-us/library/hh849895.aspx

Series: [Part 1][Part1] [Part 2][Part2] **Part 3** [Part 4][Part4]

## Modifying License Assignments

### Overview

In the first two parts of this series I showed you how to find available licenses to assign to an Office 365 user, how to assign those licenses, and how to enable only selected service plans from those licenses. All of the guidance I have seen to date around this topic stops at this point, but this is not where the work actually ends. For instance, how do we manage licensing for a user whose role changes within the organization? For many companies using Office 365 this means a change in entitlements as well. They may get different security and distribution group membership, but what if your company's needs also include a different set of services in Office 365? How can we achieve that?

### Finding currently assigned service plans
The first step in modifying a user's license assignments is finding out what services they currently has been assigned. To do this, we use the `Get-MsolUser` cmdlet:

```powershell
PS> Get-MsolUser -UserPrincipalName ronald.reagan@whitehouse.gov

UserPrincipalName            DisplayName    isLicensed
-----------------            -----------    ----------
ronald.reagan@whitehouse.gov Ronald Reagan  True
```

Ok, that doesn't give us much other than telling us that the user is licensed - let's try another approach:

```powershell
PS> $User = Get-MsolUser -UserPrincipalName ronald.reagan@whitehouse.gov
PS> $User.Licenses

ExtensionData          : System.Runtime.Serialization.ExtensionDataObject
AccountSku             : Microsoft.Online.Administration.AccountSkuIdentifier
AccountSkuId           : whitehouse:OFFICESUBSCRIPTION_FACULTY
GroupsAssigningLicense : {}
ServiceStatus          : {Microsoft.Online.Administration.ServiceStatus,
                         Microsoft.Online.Administration.ServiceStatus, Microsoft.Online.Administration.ServiceStatus,
                         Microsoft.Online.Administration.ServiceStatus}

ExtensionData          : System.Runtime.Serialization.ExtensionDataObject
AccountSku             : Microsoft.Online.Administration.AccountSkuIdentifier
AccountSkuId           : whitehouse:STANDARDWOFFPACK_FACULTY
GroupsAssigningLicense : {}
ServiceStatus          : {Microsoft.Online.Administration.ServiceStatus,
                         Microsoft.Online.Administration.ServiceStatus, Microsoft.Online.Administration.ServiceStatus,
                         Microsoft.Online.Administration.ServiceStatus...}
```

That's progress, now we can see that the user has two licenses assigned, but we can't really see which service plans are enabled in each of those. The license objects returned are a bit complex and the properties are nested quite a bit, so we'll need to get a little script-y to get a view of this that makes sense:

{% gist 78775a6f3bd7ec1e9fb3d294c5b9974f 1.ps1 %}

> NOTE: If the syntax above is unfamiliar to you, check out the help for [Select-Object][Select], particularly the fourth example that details how to create a calculated property.

If we run this in the console we should get something like this:

```powershell
AccountSkuId                               ServicePlans
------------                               ------------
whitehouse:OFFICESUBSCRIPTION_FACULTY   OFFICESUBSCRIPTION
whitehouse:STANDARDWOFFPACK_FACULTY     {YAMMER_EDU, EXCHANGE_S_STANDARD, SHAREPOINTWAC_EDU, SHAREPOINTENTERPRISE_EDU}
```

Now we have it! We can now see that Ronnie has two licenses assigned and we can see which plans are enabled in each one.

### Modify the currently assigned service plans

We can see from the above example that Ronald has access to Office Pro Plus, Yammer, Exchange Online, Sharepoint and Onedrive, and Office Web Apps. Now let's say that he needs to have access to Skype for Business as well. From the previous post in this series we know that this service plan is called `MCOSTANDARD`. We'll use the `New-MsolLicenseOptions` cmdlet to set up the correct disabled plans:

```powershell
PS> $Options = New-MsolLicenseOptions -AccountSkuId 'whitehouse:STANDARDWOFFPACK_FACULTY' -DisabledPlans 'Sway'
```

Now to actually modify the plan assignment we'll use the `Set-MsolUserLicense` cmdlet again, but with a key difference - since the user already has the license that contains plan assigned this time we don't use the `-AddLicenses` parameter:

```powershell
PS> Set-MsolUserLicense -UserPrincipalName ronald.reagan@whitehouse.gov -LicenseOptions $Options
```

Now if we re-run the script above to return the user's assigned licenses and plans, we should see that Skype for Business is enabled:

```powershell
AccountSkuId                               ServicePlans
------------                               ------------
whitehouse:OFFICESUBSCRIPTION_FACULTY   OFFICESUBSCRIPTION
whitehouse:STANDARDWOFFPACK_FACULTY     {YAMMER_EDU, MCOSTANDARD, EXCHANGE_S_STANDARD, SHAREPOINTWAC_EDU, SHAREPOINTENTERPRISE_EDU}
```

> NOTE: If we had used the `-AddLicenses` parameter we'd get an error that tells us the license we're assigning is invalid. This isn't important right now, but we'll use that knowledge later when we expand on these processes to create a licensing solution.

### Extra credit

To take this a step further, let's try an advanced example: say for instance that the user needs to have one currently assigned license modified while simultaneously adding a new license. This is possible because the license options objects contain the Sku ID that tells the `Set-MsolUserLicense` cmdlet which license to apply the options to. So if we want to modify the `STANDARDWOFFPACK_FACULTY` license just like the previous example, but we also want to add a Power BI license for the user, we'd do it like this:

```powershell
PS> $Options = New-MsolLicenseOptions -AccountSkuId 'whitehouse:STANDARDWOFFPACK_FACULTY' -DisabledPlans 'Sway'
PS> Set-MsolUserLicense -UserPrincipalName ronald.reagan@whitehouse.gov -LicenseOptions $Options -AddLicenses 'whitehouse:POWER_BI_STANDARD_FACULTY'
```

### Conclusion

In this article I've demonstrated how to modify the existing license assignments for an Office 365 user. Stay tuned for the last part in this series where I show you how to **manage the license assignments for users through their entire lifecycle**, in bulk or one at a time, and also prevent inadvertent service plan assignments in the event that Microsoft adds services to an existing license subscription.

Thanks for reading and I hope this has been helpful!