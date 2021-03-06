---
layout: post
title: ISE Steroids - Make It Yours
tags: [PowerShell, Editors]
author: Matt McNabb
comments: true
---

[ISE]: https://technet.microsoft.com/en-us/library/dd819494.aspx
[MyCommands]: /assets/img/MyCommands.png
[WMITest]: /assets/img/WBEMTest.png
[Tools]: /assets/img/ToolsMenu.png
[RightClick]: /assets/img/RightClick1.png
[Output]: /assets/img/SteroidsVersion.png

In case you can't tell, I get excited about the tools I use. ISE Steroids has become a go-to tool for my PowerShell scripting needs and has helped me get even better at doing what I do. ISE Steroids v2.0 RC2 was released a couple days ago and there are some great new capabilities that can help you customize the development experience even further. One very powerful feature is what Tobias calls '<a href="http://www.powertheshell.com/isesteroids-rc2-highlights/">Make it Yours</a>.'

Make it Yours is a set of features that allow you to add custom commands to context menus in PowerShell ISE. We have always had this in some respect using the ISE's native object model. You can create custom add-ons which execute any blocks of PowerShell code that you like. Make it Yours takes this one step further and allows you to create commands that run from the Menu Bar or the right-click context menu, and you can now add custom tools into the Menu Bar's Tools menu.

<!--more-->

###  Add a Right-Click Menu Entry
To create a context menu entry, use Add-SteroidsContextMenuCommand. These context menu commands are dynamic, so you can specify which token types they are available for. To demonstrate, let's create a menu entry that will find details about the module that a given command belongs to:

{% gist fa25dedb8ed23a3b8b79c0c23c480fde 1.ps1 %}

Notice that in the script block that we define we use `$args[1]` to refer to the command name. Add-SteroidsContextMenuCommand's -ScriptBlock parameter receives two arguments in the `$args` array - `$args[0]` is the editor that was clicked and `$args[1]` is the token.

After executing the above code you can see that we now have an entry in the right-click context menu:

![] [RightClick]

Selecting Get Parent Module gives us this output:

![] [Output]

### Adding a Tools Menu Item
Ok, this one is pretty cool! We can now add application shortcuts directly into the ISE Tools menu for quick access to tools we use frequently. I use WMI quite often, and the WMI Test utility is useful for composing WMI queries and discovering classes in a graphical interface. Let's add it to our Tools menu so we can quickly pull it up without leaving the ISE:

{% gist fa25dedb8ed23a3b8b79c0c23c480fde 2.ps1 %}

Running the above code will add the WMI Test tool to the Tools menu in the menu bar at the top of the ISE GUI:

![] [Tools]

Here is the WMI Tester in action

![] [WMITest]

### Adding a New Add-On Menu Item
The PowerShell ISE allows you to programmatically create add-ons that run any arbitrary script code that you would like. Although ISE Steroids automates nearly everything you can imagine for your script development needs, you may want to add your own automations to the ISE. In the top menu bar, click on MyCommands and then Add New Command. A new template script will be launched in the editor pane with example code to create a new command:

{% gist fa25dedb8ed23a3b8b79c0c23c480fde 3.ps1 %}

Here I will create an add-on item that removes trailing blank spaces from the end of all lines in the currently selected script:

{% gist fa25dedb8ed23a3b8b79c0c23c480fde 4.ps1 %}

Which gives us a new command item in the MyCommands menu:

![] [MyCommands]

Here is a reference on the [ISE object model] [ISE] that explains in detail how to create add-on items.

So what can you add to the PowerShell ISE to aid in developing your scripts?