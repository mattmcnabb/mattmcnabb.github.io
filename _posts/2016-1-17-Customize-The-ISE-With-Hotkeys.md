---
layout: post
title: Use Hotkeys to Customize the Powershell ISE
categories: [Powershell, Editors]
author: Matt McNabb
comments: true
---

[PowerTip]: http://powershell.com/cs/blogs/tips/archive/2015/12/08/detecting-key-presses-across-applications.aspx

### Detecting Key Presses

I haven't blogged in quite a while, but recently I was inspired by this [Powershell.com PowerTip][PowerTip] that gives us a method to detect key presses using low-level Windows API methods. I took this just a bit further and created a function that allows us to specify exactly which key we'd like to test for and will even check for multiple simultaneous keys (a chord).

{% gist 93033227b4629cf696ea53af4b1ac3ed 1.ps1 %}

### How to Use Test-KeyPress
Now I can run `Test-KeyPress -Keys Ctrl,Shift` and when the function executes if both CTRL and SHIFT are pressed it will return true. If either or both of those keys are not pressed it will return false. Obivously this is not intended to be used interactively - if you're running commands at the console prompt then you generally won't have any keys pressed other than ENTER at the time of execution.

So how can this be used you say? I use `Test-KeyPress` to make decisions when either the Powershell console or the ISE are launched. For example, sometimes I want to launch the Powershell ISE with the ISE Steroids add-on ready to go, but other times I want to load just the base ISE. To do that, I added this to my profile script:

{% gist 93033227b4629cf696ea53af4b1ac3ed 2.ps1 %}

Now when I launch the ISE, if I hold down the control key ISE Steroids will be loaded with no interaction from me. Alternatively you could use `Test-KeyPress` to execute just about any code in your profile so if you would like certain modules, variables, etc. to be conditionally available you can make that happen just by holding down a key or combination of keys.

If you have any other ideas for how to use `Test-KeyPress`, I'd love to hear them! Hit me up on [Twitter](http://twitter.com/{{site.author.twitter}}) to discuss.