---
layout: post
title: Test Business Rules with Pester
categories: [PowerShell]
author: Matt McNabb
comments: true
---

[tests]: /assets/media/Test-ADStaffAttributes.png
[quiet]: /assets/media/Test-ADStaffAttributesQuiet.png

Have you heard of this new thing called Pester? Seriously though, Pester is all over the place in the PowerShell world right now, and is now included in Windows 10 out of the box. Pester was created in 2011 by Scott Muc to satisfy his need for a unit testing framework in PowerShell, and since 2012 it has been lovingly developed by Dave Wyatt, Jakub Jares, and others. Currently in version 4.0.2, Pester is responsible for teaching developer practices to us lowly PowerShell scripters. One of the noticeable trends I've seen lately is using Pester for testing things that it was not designed for; infrastructure testing is the new buzz-word, and toward that end some Microsoft folks have offered up the Operation Validation Framework, a thin wrapper around Pester that allows for organizing infrastructure test code.

I've been abusing Pester as well as of late, but towards slightly different objectives. I don't really know what to call this, so for now I'll just call it business rule testing. This testing strategy is checking to make sure that things in your environment are configured a certain way to match up to your business cases and the particular needs of your organization. How is this different than infrastructure testing? It's not really, but is just a slightly different way of looking at things and validating according to your particular needs. As an example, if I were performing infrastructure testing of my Active Directory environment, I might check on the domain and forest configurations, tests sites and links, DNS set-up, etc. [Irwin Strachan][https://pshirwin.wordpress.com/author/istrachan/] wrote some great articles detailing how he does this type of testing to validate AD infrastructure when going on site at a client. I'm not a field engineer - I'm chained to a desk - and for me there are unique business rules around user accounts that I need to validate on a regular basis to make sure our end users are up and running correctly.

When HR contacts us to let us know that we have new staff coming aboard, there are several steps involved in getting their accounts up and running. First we create an Active Directory account, and this account is responsible for most aspects of the user's network identity. Next we configure a mailbox in Office 365, and there are several other information systems and LOB applications that they may be granted access to. Some of the account provisioning in applications is automated, some of it is manual, and if the source AD account has typoes or other mis-configurations, then cleaning the accounts up after they have been created can be quite a pain. Also, the various account creation processes belong to several members of my department which can make consistency difficult. To prevent the headaches caused by simple typoes I like to validate that a newly created user has been configured with all the right attributes, that these attribute values are configured properly, and that there are no mis-spellings in names or other inconsistencies. I've always done this by sight, simply querying users in PowerShell and making sure everything looks ok, but this is tedious and error-prone, and I often get it wrong.

I needed a way to validate our staff user accounts easily and accurately at account creation time before moving on to provisioning other applications, and to that end I wrote the function `Test-ADStaffAttributes` to do just that. This function wraps a set of Pester tests that validate of our unique business needs for each user and is easily modified or extended simply by adding or changing the tests that it runs. This is convenience automation at its' best!

Test-ADStaffAttributes consists of two components, which I've separated into two separate files intended to be stored in the same directory. The first part is a relatively simple function that wraps a call to Invoke-Pester:

{% gist ece527628bfe407a407765c6ded8ae4b Test-ADStaffAttributes.ps1 %}

The second component is the test file itself, Test-ADStaffAttributes.tests.ps1, saved in the same directory as the function definition above:

{% gist ece527628bfe407a407765c6ded8ae4b Test-ADStaffAttributes.tests.ps1 %}

Now I can run this command against a user and see if their attributes meet our business rules:

![][tests]

Whoops! Looks like i have some attributes that aren't quite up to snuff! Oh well, there are always special cases, right?

Sometimes I may not want the full Pester test output, but would just like to see if the user's attributes are in full compliance. I can use the -Quiet switch for this:

![][quiet]

If you were so inclined you could even run these tests against multiple users and report out whether anyone was out of spec:

{% gist ece527628bfe407a407765c6ded8ae4b TestAllUsers.ps1 %}

Schedule this script to run and report out on a regular basis and you've got a nice solution for preventing AD user account drift. Thanks for reading and let me know what you think in the comments below.
