---
layout: post
title: Simple PowerShell Providers with SHiPS
tags: [PowerShell, Azure]
author: Matt McNabb
comments: true
---

At Ignite earlier this week Microsoft announced the PSCloudShell preview would become publicly available to all Azure tenants. If you're not aware of PSCloudShell, it is a fully-functional PowerShell console embedded in your Azure tenant's web management portal. In PSCloudShell, your Azure subscriptions and all of their resources are easily browsable as a PowerShell drive. This Azure drive is made possible by a module called SHiPS.

# What is SHiPS?
SHiPS is a new module that Microsoft has made available in PSCloudShell, and it gives PowerShell developers the ability to create PowerShell providers written in PowerShell. It exposes a set of classes that you can inherit to create functionality similar to providers written in C# or another .NET Framework language, but without all the mucky-muck. To achieve this it leverages an existing open source project called [Simplex](https://github.com/beefarino/simplex) from Jim Christopher (AKA Beefarino) of CodeOwls.

<!--more-->

# But What is a PowerShell Provider?
A PowerShell provider is an abstract interface that allows you to navigate data hierarchies in a familiar manner, similar to navigating a file system tree. In fact, the Windows file system is made available to PowerShell by using a provider. Providers give you the ability to create drives, which are conceptually equivalent to a file system drive in Windows. Run [Get-Help about_providers](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_providers?view=powershell-5.1) to get a better idea of providers and what they are capable of.

# SHiPS vs. Simplex
SHiPS appears to be a bit of a wrapper around the base functionality of Simplex, and to all appearances just provides a different user experience for achieving the same goal of creating providers. While the original Simplex took the approach of a domain-specific language for describing a provider, SHiPS uses a library of classes that can be inherited to implement their functionality.

# Using SHiPS
In order to demonstrate how to create a PowerShell provider with SHiPS, I first had to have a hierarchical data structure that would serve as a basis for the directory tree in the drive I'd create. This could be any arbitrary data that I've made up, or something that I can already get to with existing PowerShell commands. I've chose to use our solar system as the basis for my demonstration, where the planets will be the "containers" and the moons will be the "leaves." For brevity's sake I'm only including up to the five largest moons of each of the eight planets (yes I said "eight" - let's not get started on the Pluto discussion!).

## The data
Here's what the tree will look like:

<pre>
Solar System
├───Mercury
├───Venus
├───Earth
│   └───Luna
├───Mars
│   └───Deimos
|   └───Phobos
├───Jupiter
│   └───Io
│   └───Europa
│   └───Ganymede
│   └───Calisto
│   └───Thebe
├───Saturn
│   └───Titan
│   └───Rhea
│   └───Dione
│   └───Tethys
│   └───Iapetus
├───Uranus
│   └───Oberon
│   └───Titania
│   └───Umbriel
│   └───Ariel
│   └───Miranda
└───Neptune
    └───Proteus
    └───Triton
</pre>

> #### DISCLAIMER
> Although SHiPS is publicly available in PSCloudShell, there is no license included, so I don't believe it would be wise to distribute it in any way. That said I don't believe it would violate any licensing restrictions to simply inherit from the base classes in your own project. Also please remember that PSCloudShell is still in public preview and SHiPS is still probably quite incomplete, so I wouldn't be surprised to see some major changes to it in the near future.

## The Root

To begin with, you'll need to be able to inherit classes from the SHiPS namespace, so begin your script with the `using` statement as seen below. Of course, if you're editing this script on your local computer then the SHiPS module probably won't be available so you may see red squiggles depending on your editor and configuration.

Next you'll create a new class, in my case I'm calling it SolarSystem, that will be the "root" of your drive. This class inherits from the class `SHiPSDirectory`. Inside that class you'll create a constructor that inherits from the base class constructor.

{% highlight PowerShell %}
using namespace Microsoft.PowerShell.SHiPS

class SolarSystem : SHiPSDirectory
{
    SolarSystem([string]$Name): base($Name)
    {

    }
}
{% endhighlight %}

Creating the root of your drive won't be very useful unless you can actually retrieve its child objects, so for that we'll implement a method called `GetChildItem()`. Your implementation of this method will vary depending on the data store your working with. Here we're just calling a command called `Get-Planet` which will return all of the planet objects in the SolarSystem module.

{% highlight PowerShell %}
[object[]] GetChildItem()
{
    return @(Get-Planet)
}
{% endhighlight %}

In the AzurePSDrive module, this method returns Azure artifacts such as subscriptions, resource groups, and VMs by calling the Azure PowerShell commands.

## Containers

Our second level below the root of the drive will of course be planets, and these objects will have two properties - a name, and a collection of moons. We're still inheriting from the `SHiPSDirectory` class, but this time the `GetChildItem()` method will return the moons of the current planet directory rather than the planets themselves.

{% highlight PowerShell %}
class Planet : SHiPSDirectory
{
    [string]$PlanetName
    [Moon[]]$Moons

    Planet([string]$PlanetName, [Moon[]]$Moons): base($PlanetName)
    {
        $this.PlanetName = $PlanetName
        $this.Moons = $Moons
    }

    [object[]] GetChildItem()
    {
        return @(Get-Moon -PlanetName $this.PlanetName)
    }
}
{% endhighlight %}

## Leaves

And finally we'll create our leaf object class called moons. Moons only have a single name property and of course won't include any child objects, so we'll inherit from the `SHiPSLeaf` class to create these.

{% highlight PowerShell %}
class Moon : SHiPSLeaf
{
    [string]$MoonName

    Moon($MoonName): base($MoonName)
    {
        $this.MoonName = $MoonName
    }
}
{% endhighlight %}

## Creating the drive

You can find the entire example of the solar system browser on [Github](https://github.com/mattmcnabb/SolarSystem). To actually use it, you can run these commands:

{% highlight console %}
PS> Import-Module SolarSystem
PS> New-PSDrive -Name Sol -Provider SHiPS -Root SolarSystem#SolarSystem
{% endhighlight %}

Now you can navigate into the Sol Drive and look around:

{% highlight console %}
PS> Set-Location Sol:\
PS> Get-ChildItem

    Container: Microsoft.PowerShell.SHiPS\SHiPS::SolarSystem#SolarSystem

Mode  Name
----  ----
+     Neptune
+     Jupiter
+     Uranus
+     Mercury
+     Mars
+     Earth
+     Saturn
+     Venus
{% endhighlight %}

Now you can navigate into each planet to browse its moons:

{% highlight console %}
PS> Set-Location .\Saturn
PS> Get-ChildItem

    Container: Microsoft.PowerShell.SHiPS\SHiPS::SolarSystem#SolarSystem\Saturn

Mode  Name
----  ----
+     Titan
+     Rhea
+     Dione
+     Tethys
+     Iapetus
{% endhighlight %}

# Further capabilities
So far I've only demonstrated how to create a navigable tree using Get-ChildItem, but providers offer many more features such as creating, renaming and removing items, and some lesser-known features such as transactions. The SHiPS module doesn't appear to implement these features yet, but it may in the future. Right now I'll stick to what I can see demonstrated by the authors of the AzurePSDrive module in PSCloudShell.

# Conclusion
Pretty cool, right? Now imagine that you can leverage providers with any of your own modules using SHiPS. I think it would be fascinating to try this with one of my own projects - for instance, one that works against a REST API. I hope you've found this article useful and I can't wait to see how others use the SHiPS module in their projects.

> ## Update 10/26/2017
> Microsoft released the SHiPS module on the PowerShell Gallery on 10/17/2017, so you can now install it locally using the command `Install-Module SHiPS`. They've also included [documentation and examples]>(https://github.com/PowerShell/SHiPS/blob/development/docs/README.md) to help you get started.
> 
> Also, it should be noted that the statement above that SHiPS is a wrapper for Simplex is incorrect. Simplex and SHiPS both leverage Jim Christopher's [P2F](https://github.com/beefarino/p2f) which is a framework for creating PowerShell providers. However, the comparison between Simplex's DSL implementation and SHiPS' class inheritance approach is still accurate.
