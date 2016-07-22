---
layout: post
title: PSCredential Gets a Makeover in PowerShell 5.0
categories: [Powershell]
author: Matt McNabb
comments: true
---

[CredentialAttribute]: https://msdn.microsoft.com/en-us/library/system.management.automation.credentialattribute(v=vs.85).aspx
[CredAttrExplain]: https://msdn.microsoft.com/en-us/library/ee857074(v=vs.85).aspx
[ArgTransform]: https://msdn.microsoft.com/en-us/library/system.management.automation.argumenttransformationattribute(v=vs.85).aspx
[PopUp]: /assets/media/CredAttrPopUp.png
[PSCredential]: https://msdn.microsoft.com/en-us/library/system.management.automation.pscredential%28v=vs.85%29.aspx?f=255&MSPPError=-2147217396

During a recent talk I gave at the Cincinnati PowerShell User Group, I briefly demonstrated the technique I use to create advanced functions that accept credentials. I've been using this approach for a while and thought it would be great to show it off so others can take advantage of it. Of course like most demos it failed miserably. Here's why:

### PowerShell 4.0 - The Way We Were

In PowerShell 2 and above we could specify that a function parameter should accept objects of type `System.Management.Automation.PSCredential`, or use the type adapter `[PSCredential]` starting in v3.0:

{% highlight PowerShell %}
function Test-PSCredential
{
    param
    (
        [PSCredential]
        $Credential
    )

    $Credential
}
{% endhighlight %}

This parameter will accept a pre-created credential object and nothing else:

``` console
$Cred = Get-Credential
Test-PSCredential -Credential $Cred
```

However, if you want to pass in a string username you're out of luck:

``` console
Test-PSCredential -Credential matt
```

``` consoleerror
Test-PSCredential : Cannot process argument transformation on parameter 'Credential'. Cannot convert the "matt"
value of type "System.String" to type "System.Management.Automation.PSCredential".
At line:1 char:31
+ Test-PSCredential -Credential matt
+                               ~~~~~~~~
    + CategoryInfo          : InvalidData: (:) [Test-PSCredential], ParameterBindingArgumentTransformationException
    + FullyQualifiedErrorId : ParameterArgumentTransformationError,Test-PSCredential
```

To do this we need to use the [System.Management.Automation.CredentialAttribute()][CredentialAttribute] parameter attribute:

{% highlight PowerShell %}
function Test-CredentialAttribute
{
    param
    (
        [System.Management.Automation.CredentialAttribute()]
        $Credential
    )

    $Credential
}
{% endhighlight %}

You can read about how this attribute works [here][CredAttrExplain]. When you use this attribute your credential parameter will happily accept either credential object or a string username, in which case you'll be prompted to enter the password:

![][PopUp]

### PowerShell 5.0

So I attempted to demonstrate this approach in my demo and found that when I ran `Test-PSCredential` I didn't receive an error - it behaved exactly the same way as using `CredentialAttribute()`! It seems that the `PSCredential` class has gotten a bit of an upgrade in PowerShell 5.0. Let's check it out:

``` console
Trace-Command -Expression {Test-PSCredential -Credential matt} -Name ParameterBinding -PSHost

DEBUG: ParameterBinding Information: 0 : BIND arg [matt] to parameter [Credential]
DEBUG: ParameterBinding Information: 0 :     Executing DATA GENERATION metadata: [System.Management.Automation.CredentialAttribute]
DEBUG: ParameterBinding Information: 0 :         result returned from DATA GENERATION: System.Management.Automation.PSCredential
DEBUG: ParameterBinding Information: 0 :     Executing DATA GENERATION metadata: [System.Management.Automation.ArgumentTypeConverterAttribute]
DEBUG: ParameterBinding Information: 0 :         result returned from DATA GENERATION: System.Management.Automation.PSCredential
DEBUG: ParameterBinding Information: 0 :     COERCE arg to [System.Management.Automation.PSCredential]
DEBUG: ParameterBinding Information: 0 :         Parameter and arg types the same, no coercion is needed.
DEBUG: ParameterBinding Information: 0 :     BIND arg [System.Management.Automation.PSCredential] to param [Credential] SUCCESSFUL
```

Using `Trace-Command` we can watch the parameter binding process and see that `PSCredential` is now leveraging the `CredentialAttribute` argument transformation in the background! This makes it really easy to create flexible function parameters that accept either a credential object, or a simple string username.

I'm not sure when this feature was added to the `PSCredential` class as the [MSDN documentation][PSCredential] doesn't seem to allude to this at all. Also, keep in mind that since this was only added in PowerShell 5.0, you'll need to continue using the `CredentialAttribute()` decoration to support downlevel versions of PowerShell.

### Extra Credit
The above MSDN article on the `CredentialAttribute` class states that it inherits from the [ArgumentTransformationAttribute class][ArgTransform]. I find this intriguing and would love to learn more about argument transformations and if any other classes inherit from this or leverage it's functionality. If you have any background on this class, please hit me up or blog about it somewhere.