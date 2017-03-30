---
layout: post
title: What's in Your PowerShell Profile?
tags: [PowerShell, Profile]
author: Matt McNabb
comments: true
---

[PortableProfile]: /posts/portable-profile

"What's in Your Profile?"

This is a question you see often in PowerShell blogs and forums. PowerShell profiles can be a powerful way to automate your command line environment and most importantly save typing and time.

In my [previous post][PortableProfile] I went over how to make a portable PowerShell profile. In this post I'll go over what I have in my profile scripts and why.

### The Main Profile

In this profile script I configure most of the settings that will apply to both the PowerShell console and the ISE:

<!--more-->

{% gist 57aff66df11c7e97c3fb10db552d3730 1.ps1 %}

Let's walk through this bit-by-bit. The first two lines import credential objects that I have previously saved using the New-SavedCredential function and saves them in variables. This saves the username and password securely in an XML file that represents a PowerShell credential object. I typically run powershell with a standard user account and use these credentials in commands that require more privilege. I save the credentials in SAM and UPN formats - SAM format is for things like domain servers and Active Directory, and the UPN credentials can be used against web services like Office 365.

{% gist 57aff66df11c7e97c3fb10db552d3730 2.ps1 %}

Next I extend the PowerShell module path to include my script library; this adds quick access to modules stored in OneDrive. I also remove my local documents modules folder from the modulepath variable. I do this because I often work over a VPN connection and my documents are stored on a file server in our datacenter. This can mean that automatic module enumeration is very slow over the WAN connection. This cripples tab-completion and Intellisense because it constantly checks your module path to discover commands and parameters.

{% gist 57aff66df11c7e97c3fb10db552d3730 3.ps1 %}

Next in line is a prompt function. When you create a function named prompt, the text output of that function will be used as the command-line prompt. I like mine with some color and it also displays the execution time of the last command run.

{% gist 57aff66df11c7e97c3fb10db552d3730 4.ps1 %}

{% gist 57aff66df11c7e97c3fb10db552d3730 5.ps1 %}

Next I load several modules that I work with on a regular basis. Since PowerShell version 3.0 this is not really necessary since modules when auto-load when they are called upon, but I prefer that they load at startup to save time later. Besides that I still sometimes run PowerShell version 2.0 which does not auto-load modules, so this ensures the right modules are loaded when I run PowerShell. I have recently begun using the Windows Azure module but I don't really want to load it when I launch the shell, so I save the path to the module in a variable and call that later when I need to access the Azure cmdlets.

{% gist 57aff66df11c7e97c3fb10db552d3730 6.ps1 %}

I manage a print server on a daily basis and use cmdlets in the PrintManagement module to do so. These modules leverage WMI/CIM to retrieve and configure printer settings. For quick access I like to have a CIM session to my print server already open when I load PowerShell. I use one of my imported admin credentials to establish this connection.

{% gist 57aff66df11c7e97c3fb10db552d3730 7.ps1 %}

I use the $PSDefaultParameterValues automatic variable to configure default values when I run certain commands. This is a huge time-saver if you find yourself typing the same parameter values in on a regular basis. An example of this is setting the -CimSession parameter of all of the PrintManagement cmdlets with a value of $CimPs01 from the previous line in my profile script. This makes sure that I am always running printer cmdlets against the right computer without having to enter the parameter at the command line.

{% gist 57aff66df11c7e97c3fb10db552d3730 8.ps1 %}

Lastly, if the current PowerShell host is the ISE editor, I import a module called ISELibrary with a collection of functions for interacting with the ISE. I then run another profile script that makes some changes like the default pane view and loading custom add-ins.

{% gist 57aff66df11c7e97c3fb10db552d3730 9.ps1 %}

{% gist 57aff66df11c7e97c3fb10db552d3730 10.ps1 %}

### Other scripts

The ISEConfig script above also calls another profile script - load-addons.ps1. This script uses some of the functions in my ISELibrary module to create ISE add-ons. These add-ons take advantage of the extensibility of the ISE object model and allow you to create ISE menu items that run PowerShell code and can be called with a keyboard shortcut.

{% gist 57aff66df11c7e97c3fb10db552d3730 11.ps1 %}

Well, that's my PowerShell profile! I hope this wasn't too long-winded but instead illustrates how flexible and dynamic PowerShell profiles can be.

So, what's in your profile?