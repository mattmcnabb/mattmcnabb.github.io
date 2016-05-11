---
layout: post
title: Managing Office 365 User Licenses with PowerShell - Part 2
categories: [Powershell, Office 365]
comments: true
---

[Part1]: /Office-365-Licensing_1
[Part2]: /Office-365-Licensing_2
[Part3]: /Office-365-Licensing_3
[Part4]: /Office-365-Licensing_4
[ServicePlans]: /assets/media/ServicePlans.png

Series: [Part 1][Part1] **Part 2** [Part 3][Part3] [Part 4][Part4]

## License Options

### Overview
In the [previous blog post][Part1] I showed you how to connect to Azure Active Directory using PowerShell and assign a license sku to an Office 365 user which entitles a user for all the services contained therein. In many cases this is sufficient, but some organizations may have more specific needs regarding what services are available to its' users. For instance, you may want to provision Exchange to your users after you've migrated from another mail service, but you want to wait a bit to deploy Sharepoint, Skype for Business, and Onedrive for Business until your users have gotten used to Office 365. To achieve this you'll need to use selective license entitlement. This can be done in the Admin Center by deselecting the services that are included with a license sku, 

![][ServicePlans]

but to do this in bulk we'll need to use PowerShell.

### Finding available service plans
Each license sku in Office 365 contains one or more service plans that can be enabled to provision a service for a user. These service plans include things like Exchange, Sharepoint, Skype for Business, and even external services like Sway or Intune. In part 1 of this series we looked at the Office 365 for Education license - let's take a look at how to find the services that are available in that sku: 

``` console
Get-MsolAccountSku | Where-Object AccountSkuId -like '*Woffpack_fac*' | Select-Object -ExpandProperty ServiceStatus

ServicePlan            ProvisioningStatus
-----------            ------------------
SWAY                   Success
INTUNE_O365            PendingActivation
YAMMER_EDU             Success
SHAREPOINTWAC_EDU      Success
MCOSTANDARD            Success
SHAREPOINTSTANDARD_EDU Success
EXCHANGE_S_STANDARD    Success
```

So we can see that the service plans available to assign from the Office 365 for Education license are:

* Sway (*SWAY*)
* Yammer (*YAMMER_EDU*)
* Office Web Apps (*SHAREPOINTWAC-EDU*)
* Skype or Business (*MCOSTANDARD*)
* Sharepoint (*SHAREPOINTSTANDARD_EDU*; includes Onedrive for Business)
* Exchange (*EXCHANGE_S_STANDARD*)

> NOTE: Intune's status is marked as `PendingActivation` because my organization has not purchased that service.

### Assign selected service plans
Last time we licensed Honest Abe with all included service plans in the Office 365 for Education license, so this time we'll find someone else and assign only select service plans from the license.

``` console
PS C:\> Get-MsolUser -UnlicensedUsersOnly

UserPrincipalName                DisplayName       isLicensed
-----------------                -----------       ----------
Abe.Lincoln@whitehouse.gov       Abe Lincoln       False
Grover.Cleveland@whitehouse.gov  Grover Cleveland  False
Ronald.Reagan@whitehouse.gov     Ronald Reagan     False
```

This time we'll pick Ronald Reagan, and we only want to give him Exchange and Sharepoint. To do this, we need to learn a new cmdlet - `New-MsolLicenseOptions`. We'll run this before running the Set-MsolUserLicense cmdlet in order to configure the settings we want to assign along with the license. Pay special attention to the `-DisablePlans` parameter of `New-MsolLicenseOptions`, as this is how we configure selected service plans. Repeat - **Enabled service plans are configured by setting those that we want to be disabled**. If this sounds a little backward to you, then you'd be right and it can be problematic, and we'll talk more about that in a later post.

``` console
PS C:\> New-MsolLicenseOptions -AccountSkuId whitehouse:STANDARDWOFFPACK_FACULTY -DisabledPlans SWAY, YAMMER_EDU, SHAREPOINTWAC_EDU, MCOSTANDARD

ExtensionData AccountSkuId                                         DisabledServicePlans
------------- ------------                                         --------------------
              Microsoft.Online.Administration.AccountSkuIdentifier {Sharepointstandard_edu}
```

Here we can see that running this command outputs an object of type `Microsoft.Online.Administration.LicenseOption` which can passed to the `Set-MsolUserLicense` cmdlet. In order to do that, we'll need to save this object in a variable:

{% highlight Powershell %}
$Options = New-MsolLicenseOptions -AccountSkuId whitehouse:STANDARDWOFFPACK_FACULTY -DisabledPlans SWAY, YAMMER_EDU, SHAREPOINTWAC_EDU, MCOSTANDARD
{% endhighlight %}

Now we can set the license for Ronnie:

{% highlight Powershell %}
Set-MsolUserLicense -UserPrincipalName Ronald.Reagan@whitehouse.gov -AddLicenses whitehouse:STANDARDWOFFPACK_FACULTY -LicenseOptions $Options
{% endhighlight %}

### Assign selected service plans from multiple licenses
We can assign multiple licenses at the same time and still enabled selected plans by doing the following:

{% highlight Powershell %}
$Options = New-MsolLicenseOptions -AccountSkuId whitehouse:STANDARDWOFFPACK_FACULTY -DisabledPlans SWAY, YAMMER_EDU, SHAREPOINTWAC_EDU, MCOSTANDARD
$Options += New-MsolLicenseOptions -AccountSkuId whitehouse:OFFICESUBSCRIPTION_FACULTY -DisabledPlans ONEDRIVESTANDARD
{% endhighlight %}

Now we can set the licenses for Ronnie:

{% highlight Powershell %}
Set-MsolUserLicense -UserPrincipalName Ronald.Reagan@whitehouse.gov -AddLicenses whitehouse:STANDARDWOFFPACK_FACULTY, whitehouse:OFFICESUBSCRIPTION_FACULTY -LicenseOptions $Options
{% endhighlight %}

### Conclusion
In this article I've shown you how to assign a license sku to an unlicensed user while enabling selected service plans. In part three of this four part series I'll show you how to modify the enabled service plans for a user who has already been licensed.

Thanks for reading and I hope you found this article useful!