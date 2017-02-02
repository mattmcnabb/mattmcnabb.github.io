---
layout: post
title: Why I Don't Like PowerShell DSLs (and Neither Should You)
categories: [PowerShell]
author: Matt McNabb
comments: true
---

[DonJones]: https://powershell.org/2016/12/15/the-key-to-understanding-powershell-on-windows-or-linux/
[Pester]: https://github.com/pester/Pester
[Youtube]: https://www.youtube.com/watch?v=sefYvIFbG5M
[Shovel]: /assets/media/Shovel.gif?raw=true

![][Shovel]

DSLs seem to be all the rage right now in the PowerShell world. Pester has quickly become a staple in many PowerShell developer's toolbelts and there seem to be new DSLs cropping up on a regular basis. Simplex, Pscribo, and even DSC are all examples of DSLs written in or for use in PowerShell. I'm not a big fan of DSLs and I'll explain why.

DSL stands for Domain Specific Language, and it means a programming language designed to tackle a very "specific" problem domain. Most of the common programming languages like C#, Java, and even PowerShell are general purpose and can be used to approach a very wide variety of uses and applications. In contrast, a DSL has a narrow scope of focus. A common reason for using DSLs is that they can use more natural language that is closer to the idiom of the problem domain itself. Pester, for instance, expresses test conditions using natural language elements like "It" and "Should" to imitate the way that we think about units of code.

PowerShell DSLs come in two flavors these days - function-based and keyword-based. Function based DSLs have been around for quite some time in PowerShell, and this is how Pester is implemented. This approach typically involves using (and abusing!) standard advanced functions and positional parameters with natural language names. A basic Pester test really boils down to a function called `It` with a string parameter `-name` and a `-scriptblock` parameter that performs the test. Inside that scriptblock, the output of any command can be piped to a function `Should` which has some very special magic that performs tests on your code. Keyword-based DSLs are created using the DynamicKeyword .NET class and are relatively new, having been introduced in PowerShell v4.0. The declarative language powering Desired State Configuration was created using keywords.

PowerShell, however, revels in being general purpose. Its' primary strenth is consistency of behavior across different managed technologies. I'm not the only one who thinks this way, evidenced by [Don Jones' article ][DonJones] explaining the benefits of using PowerShell for systems administration. In the old days of Windows we had many different automation tools at our disposal. We had command line applications like net, netsh, xcopy, robocopy, nltest. We had VBS for coverage with WMI and COM, and many GUI-based management applications offered some sort of automation capabilities in there own arena. See where I'm going with this? Just in the world of CMD we had numerous applications each with its' own idea of how to do things. Some commands used "-" in front of parameter names, some used "/". Some used both! Other commands had mind-boggling numbers of parameters - just try running `Robocopy /?` and let me know what you think. And then commands like netsh can be run interactively and implement a new language parser outside of the CMD session. All these elements have their own learning curve which makes large-scale automation difficult and unattainable for some organizations.

PowerShell was the great equalizer in the Windows administration world and enables all manner of complex automation, but with consistent, repeatable behavior. At its' best it abstracts the details of varied technologies away so that you can focus on the higher level needs of the task at hand. That's why I have such a problem with DSLs. To me they feel like a return to the old way of doing things, where automation in every technology had its' own learning curve. Even the normally sane and rational Kirk Munro has jumped on the DSL bandwagon and is now an open advocate for DSLs in PowerShell, going so far as to give a [talk][Youtube] about them at last year's PowerShell Summit. This is alarming and gives me nightmares of wading through context-based help menus:

```console
netsh /?
```

Ok.

```console
netsh firewall /?
```

Crap!

```console
netsh advfirewall /?
```

Anyway, stop it with the DSLs and just scope your problems to functions and modules like a real PowerShell purist! Add a comment below if you'd like to let me know why you agree with my point of view. Otherwise, get off my lawn!

Thanks for reading!
