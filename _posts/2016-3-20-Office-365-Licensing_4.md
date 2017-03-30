---
layout: post
title: Managing Office 365 User Licenses with PowerShell - Part 4
tags: [PowerShell, Office 365]
author: Matt McNabb
comments: true
---

[Part1]: /Office-365-Licensing_1
[Part2]: /Office-365-Licensing_2
[Part3]: /Office-365-Licensing_3
[Part4]: /Office-365-Licensing_4
[Gist]:  https://gist.github.com/mattmcnabb/13c0f230bde0359e4aeb5dad0de84712

Series: [Part 1][Part1] [Part 2][Part2] [Part 3][Part3] **Part 4**

## Full Automation

### Overview
So far in this blog series I've covered the basics of user licensing in Office 365 with PowerShell by demonstrating how to add and modify license skus. This is very useful in scripting the service entitlements for new users, but not all users are static and in many cases you'll need to manage licenses as users move between positions and licensing needs change. This isn't as easy as it sounds (or should be), and has a couple of obstacles.

### Problem 1 - DisabledPlans
The biggest drawback to configuring user licenses via PowerShell lies in the design of the New-MsolLicenseOptions cmdlet. The problem is that the -DisabledPlans parameter is inherently the wrong approach to license automation. For example, let's say we've set up a script that licenses users for the EnterprisePack and you've added Sharepoint to the disabled plans. In it's original state, this would have enabled Exchange, Skype for Business, and Yammer. However, last year Microsoft added a new service plan to the license - Sway. This means that as soon as Sway became available as an assignable license in your tenant, Sway would have been assigned to your users because it hasn't been explicitly added to the list of disabled plans.

<!--more-->

So what we need to solve this problem is a method of setting enabled plans rather than disabled ones. That way no services will be provisioned for users unless you have explicitly added it in your script.

### Problem 2 - AddLicenses
When you license a user who is currently unlicensed you use the -AddLicenses parameter of Set-MsolUserLicense to provision the license for the first time. However, if a user is already licensed and you need to modify the provisioned service plans, you need to omit the -AddLicenses parameter. If you don't you'll receive an error - "The license is invalid." This isn't a very descriptive error, and you'll get the same error if you assign an appropriate license but with invalid options. This can be a big roadblock to an automated solution since we won't know how to react to the error appropriately.

### Solutions
Let's start with problem 1. To approach this problem I started by imagining a data structure that would contain license configurations in a way that is native to PowerShell. I use hash tables for nearly everything, so it's no surprise that what I came up with utilizes them. Check out this example license template:

{% gist 046774cab99eecb091a458ea407f752b 1.ps1 %}

This script snippet will create a new array of hashtables and save it in a variable called `LicenseTemplate`. Notice that our hash tables have a key called `EnabledPlans` rather than `DisabledPlans`.

Next, we need a method to consume this license template. We can do that by looping through each hashtable, inverting the enabled plans to disabled plans, and then outputting an appropriate license options object:

{% gist 046774cab99eecb091a458ea407f752b 2.ps1 %}

That ought to do it! We can now prevent unexpected service plans from being added to our users.

To solve problem 2, we need to employ a bit of error handling. Remember when I said that attempting to add a license to a user who already owns said license will result in an error? We're going to use that knowledge to create method for detecting whether the license is already added and if so, we'll just pass in options for the license. Check it out:

{% gist 046774cab99eecb091a458ea407f752b 3.ps1 %}

I'm using a try/catch construct here to capture these errors. This approach will attempt to assign the desired license with the preconfigured options to the user. If the license is already assigned we'll get an error of type `[Microsoft.Online.Administration.Automation.MicrosoftOnlineException]` and we'll retry by removing the `-AddLicenses` parameter.

Building on these approaches I've created the `Set-O365UserLicense` function that can be found [here][Gist]. This function accepts license templates in the form of one or more hash tables and can set this license template for multiple users. And to fully round out it's functionality, it will also remove any licenses for a user that are not contained in the license template. What we now have is a declarative style of licensing for Office 365 users that is much more usable than the functionality native to the MSOnline module.

### Using Set-O365UserLicense
Here's an example of how you might use the function to automate licensing for the ENTERPRISEPACK sku:

{% gist 046774cab99eecb091a458ea407f752b 4.ps1 %}

That's all for this blog series on Office 365 user licensing. This series was a challenge to write as I had been kicking these ideas around in my head for quite some time, but fully realizing them and committing them to paper took some effort! But I had fun and have already seen the benefit of this new approach in my own Office 365 licensing scripts. I hope you find this as useful as I have! Thanks for reading!