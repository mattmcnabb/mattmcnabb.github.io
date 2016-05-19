---
layout: post
title: Testing Your Module Manifest With Pester - Revisited
categories: [Powershell, Pester]
comments: true
permalink: 
---

[PreviousPost]: /pester-testing-your-module-manifest

I ran into a problem recently when running a TeamCity build process for a PowerShell module that I have published to the PowerShell Gallery. The build task continually returned errors and I couldn't quite figure out what was going on. I decided to run Publish-Module locally to troubleshoot the issue and was surprised to see this error returned:

{% highlight consoleerror %}
C:\Publish.ps1 : Failed to publish module 'O365ServiceCommunications': 'Failed to process request. 'A nuget package's Tags 
property extracted from the PowerShell manifest may not contain spaces: 'Office 365'.'. 
The remote server returned an error: (500) Internal Server Error..
'.
    + CategoryInfo          : InvalidOperation: (:) [Write-Error], WriteErrorException
    + FullyQualifiedErrorId : FailedToPublishTheModule,Publish.ps1
{% endhighlight %}

As you can see, the problem was nothing more than a single space in one of the module tags. These tags are contained in the module manifest, in the "PrivateData" key and are used to categorize your module in the gallery.

To prevent this from happening in the future I wrote a simple test to make sure that my tags don't contain spaces. I previously wrote about how you can [test your module manifest values using Pester][PreviousPost], and I just plugged these tests into my existing test file for the module.

{% highlight PowerShell %}
It "gallery tags don't contain spaces" {
    foreach ($Tag in $ManifestHash.PrivateData.Values.tags)
    {
        $Tag -notmatch '\s' | Should Be $true
    }
}

{% endhighlight %}

For good measure I threw in a couple more tests to validate that the manifest has a license URI and a project URI.

{% highlight PowerShell %}
It 'has a valid project Uri' {
    $ManifestHash.PrivateData.Values.ProjectUri | Should Be 'https://github.com/mattmcnabb/O365ServiceCommunications'
}

It 'has a valid license Uri' {
    $ManifestHash.PrivateData.Values.LicenseUri | Should Be 'http://opensource.org/licenses/MIT'
}
{% endhighlight %}

 Now when my build tasks run, if the module manifest doesn't have the correct values then the Pester tests will catch this and the publish task won't even run. Pretty cool!