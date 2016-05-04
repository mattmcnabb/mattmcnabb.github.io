---
layout: post
title: Thoughts on Powershell Parameter Naming Conventions
categories: [Active Directory, Powershell]
---

[MVA]: http://www.microsoftvirtualacademy.com/training-courses/using-powershell-for-active-directory
[MSDN]: http://msdn.microsoft.com/en-us/library/dd878270(v=vs.85).aspx

If you haven't seen the Microsoft Virtual Academy series on Active Directory and Powershell yet, you should check it out [here][MVA]. Hosted by Ashley McGlone and Jason Helmick, it has tons of information on how to get started using Powershell to manage Active Directory and there is some great supplemental material that you can download and work through on your own.

Anyway, I was thinking about one of Jason's comments in the series regarding Powershell parameter naming conventions. Specifically he says that the `-Properties` parameter of `Get-Aduser` should be `-Property` instead. According to [MSDN][MSDN], Powershell cmdlet parameter names should always be singular unless they only take multiple values as an argument. Since `-Properties` accepts either a single value or an array of values, it should instead be -Property.

First off, `-Property` is a valid alias for this parameter so if you type it this way instead it will still work. However tab completion will always fill out the plural version of the parameter. I think that this particular practice in general is flawed and I'll tell you why - because it doesn't help anyone.

The practice I prefer is to always use singular names for parameters that can only accept single values, and always use plural names for any parameter that can accept multiple values. The reasoning is simple - you can tell just by looking at the parameter name whether it accepts multiple values or not. You don't need to run get-help at all in order to see that `Get-ADUser -Properties Title,Department` will work. Also, I don't know of many cmdlet parameters that only accept multiple values. I am sure some exist, but it's far more common to have parameters that will accept an array with only one element.

As always, comments and criticism are welcome.