---
layout: post
title: Remove Error Entries in Powershell Functions
categories: [Powershell]
author: Matt McNabb
comments: true
---

[NullArray]: /assets/media/NullArrayError.png
[RemoveAt]: /assets/media/RemoveAtError.png

Hi folks! Today's post will be a short one based on something I discovered while testing error handling in a module I'm working on.

One of my module functions enumerates all the variables in the current session and checks the data type of each value. The function will return any variable values that match a particular custom data type.

Inevitably some of the variables in any session are null. Since my code is looking for the TypeNames property of an object I use the 0 index of TypeNames to look at only the first value, an error is generated if there are no TypeName values:

![] [NullArray]

Since this is in our try block I can easily handle this error so that it is not displayed to the user, but this particular function may be run quite often and I don't want to pollute the error stream with tons of negligible errors. My idea was to include a type-specified catch block that will remove this error from the error variable when it occurs. Here is the base code for that:

{% highlight Powershell %}
try
{
    $Variables = Get-Variable -Scope Global -ErrorAction Stop
    foreach ($Variable in $Variables)
        {
            $VarName = $Variable.Name
            $Value = Invoke-Expression "`$$VarName"
            $TypeName = $Value.PSObject.TypeNames[0]
            if ($TypeName -eq 'My.Type') {$Value}
        }
}
catch [System.Management.Automation.RuntimeException]{$Error.RemoveAt(0)}
{% endhighlight %}

So this should work, right? Nope! Whenever I ran the function I would receive the error:

![] [RemoveAt]

However, testing this same method outside of my function worked just fine. So what's the deal? It seems that the error is telling me that the `$Error` variable does not contain any objects. Turns out it's an issue with scope! Changing the catch block of my function to add a variable scope identifier fixed the problem:


{% highlight Powershell %}
catch [System.Management.Automation.RuntimeException]{$Global:Error.RemoveAt(0)}
{% endhighlight %}

So what I learned here was that although the global `$Error` variable is accessible to a function inside a module, the array methods need the variable to be explicitly scoped in order to work. This was a great lesson in error handling and array manipulation.