---
layout: post
title: Testing Your Module Manifest With Pester - Revisited
tags: [PowerShell, Pester]
comments: true
---

[PreviousPost]: /pester-testing-your-module-manifest

I ran into a problem recently when running a TeamCity build process for a PowerShell module that I have published to the PowerShell Gallery. The build task continually returned errors and I couldn't quite figure out what was going on. I decided to run Publish-Module locally to troubleshoot the issue and was surprised to see this error returned:

{% highlight console %}
C:\Publish.ps1 : Failed to publish module 'O365ServiceCommunications': 'Failed to process request. 'A nuget package's Tags
property extracted from the PowerShell manifest may not contain spaces: 'Office 365'.'.
The remote server returned an error: (500) Internal Server Error..
'.
    + CategoryInfo          : InvalidOperation: (:) [Write-Error], WriteErrorException
    + FullyQualifiedErrorId : FailedToPublishTheModule,Publish.ps1
{% endhighlight %}

<!--more-->

As you can see, the problem was nothing more than a single space in one of the module tags. These tags are contained in the module manifest, in the "PrivateData" key and are used to categorize your module in the gallery.

To prevent this from happening in the future I wrote a simple test to make sure that my tags don't contain spaces. I previously wrote about how you can [test your module manifest values using Pester][PreviousPost], and I just plugged these tests into my existing test file for the module.

{% gist 02f6fcf93d66d00c7147fad48ec7dbd8 1.ps1 %}

For good measure I threw in a couple more tests to validate that the manifest has a license URI and a project URI.

{% gist 02f6fcf93d66d00c7147fad48ec7dbd8 2.ps1 %}

 Now when my build tasks run, if the module manifest doesn't have the correct values then the Pester tests will catch this and the publish task won't even run. Pretty cool!

### Update 5/19/2016

I have corrected the assertion in the `It` block named "gallery tags don't contain spaces"  to use the `Not` keyword, instead of using a boolean to test the regex match like this

{% gist 02f6fcf93d66d00c7147fad48ec7dbd8 3.ps1 %}

I wasn't aware of how to properly negate an assertion - thanks to Dave Wyatt for the suggestion!