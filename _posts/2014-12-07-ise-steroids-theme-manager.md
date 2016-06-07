---
layout: post
title: ISE Steroids Theme Manager
categories: [Powershell, Editors]
author: Matt McNabb
comments: true
---

[DarkModern]: /assets/media/DarkTheme.png "The Dark Modern Theme"
[Solarized]: /assets/media/SolarizedTheme.png "The Solarized Theme"
[Snippets]: /posts/ise-steroids-snippet-manager.html
[Profile]: /posts/portable-profile.html
[WhiteRegion]: /assets/media/WhiteRegion.png "ISE Native Region Highlighting"
[BlueRegion]: /assets/media/BlueRegion.png "ISE Steroids Region Highlighting"

If you haven't heard about it yet, ISE Steroids is a premium add-on for the Powershell ISE that has tons of great new functionality for making your scripting experience more productive. I already covered it's snippets manager [here][Snippets].

Well now we have a release candidate version of ISE Steroids version 2 that has packed even more into this tiny little box (seriously - 39 MB!) So far my favorite is the new UI Theme Manager which allows you to manage the code syntax highlighting colors, fonts, and more. One of the cool features I had heard about was the capability to change token highlighting colors on the fly using the right-click context menu.

### Enable Theme Management
The first time I launched ISE Steroids 2 RC I immediately opened a script and right-clicked on the function token and guess what? Nothing! What's the deal? Ok, so it wasn't really that hard to deal with, you just have to enable the theme management feature before you can use it. Go up to the upper right-hand corner of the toolbar and you'll see a new button for the UI Theme Manager. Click that and select Enable Theme Adjustment and you're all set. You can now right-click any token in the ISE and change the syntax highlighting color for that token type.

### Built-In Themes
The Theme Manager comes with seven built-in themes - five from the ISE's default themes and 2 new ones that demonstrate some of it's new features. These two new themes are 'Dark Modern' and 'Monochrome Green Enhanced.' Let's try one:

![Dark Modern Theme][DarkModern]

Very cool! Right off the bat I notice two features that weren't possible before:

1. Separate fonts for the the script editor and the console
2. Theming for the ISE itself, including the toolbar.

### Editing and Creating Themes
Digging a bit deeper we can get an idea how this is accomplished and how to create our own custom themes. First, click the theme manager icon again and select 'Save Current Theme.' This creates a new theme folder in your local appdata directory. Give the theme a name and click save. Back in the ISE, look at the theme manager again and you will see the 'Open Themes Folder' button is now available. You can use this to add or delete themes manually.

Theme files for ISE Steroids are XML and are similar to native theme files for the ISE, but are much friendlier to read. Tokens are now explicitly named and you can easily see what everything does. Steroids themes are given the file extension 'ISESteroidsThemeXML' whereas traditional ISE themes are 'StorableColorTheme.ps1xml.'

{% highlight XML %}
<?xml version="1.0"?>
<ColorOptions xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <ShareFontForConsole>false</ShareFontForConsole>
  <ScriptPaneFontFamily FontFamilyName="Lucida Console" />
  <ScriptPaneFontSize>9</ScriptPaneFontSize>
  <ConsolePaneFontFamily FontFamilyName="OCR A" />
  <ConsolePaneFontSize>9</ConsolePaneFontSize>
  <ConsolePaneForegroundColor ColorARGB="#8BF5F5F5" />
  <ConsolePaneBackgroundColor ColorARGB="#FF012456" />
  <ConsolePaneErrorForegroundColor ColorARGB="#FFFF0000" />
  <ConsolePaneErrorBackgroundColor ColorARGB="#00FFFFFF" />
  <ConsolePaneWarningForegroundColor ColorARGB="#FFFF8C00" />
  <ConsolePaneWarningBackgroundColor ColorARGB="#00FFFFFF" />
  <ConsolePaneVerboseForegroundColor ColorARGB="#FF00FFFF" />
  <ConsolePaneVerboseBackgroundColor ColorARGB="#00FFFFFF" />
  <ScriptPaneBackgroundColor ColorARGB="#FF012456" />
  <CapitalizeMainMenuHeaders>true</CapitalizeMainMenuHeaders>
{% endhighlight %}

While there is no menu for custom creating a theme the way you would in the ISE natively, there is really no need since you can simply change your colors on the fly using the context menu and then when you are happy with the results you can save the new theme to your appdata folder.

If you want to copy any custom themes you have already created in the ISE to the new Steroids theme format, just apply your theme from the ISE options menu and then save it as a new Steroids theme. Here's my own take on the Solarized theme that I created a while back, now converted to an ISE Steroids theme:

![][Solarized]

### Take Them With You
I take my Powershell profile with me using Onedrive, so I don't really want to create a custom theme on my local computer only. Here's a cool trick that will make your custom themes available anywhere you have access to Onedrive. First, set up a travelling profile following the instructions [here][Profile]. Next drop the ISESteroids module into your modules folder so it's always available. Inside the Steroids module there is a 'Themes' folder that contains the built-in themes. Now you can copy your custom themes from the appdata folder created previously into this folder and they will be available whenever you load the module.

###  Visual Treat
Overall I think the new theme management in ISE Steroids is a great addition that clears up some of the niggles I had with the native themes in the ISE. One really cool thing I noticed is the region highlighting in the script editor now works with dark backgrounds so you don't get those awful white regions in your otherwise slick-looking color scheme:

![Native ISE Region Highlighting][WhiteRegion]

![ISE Steroids Region Highlighting][BlueRegion]

If you aren't familiar with region highlighting, it's what happens when you mouse over a collapsible region on the left of the script editor near the line numbers column. This is just another example of Tobias' attention to detail with his new product.

That's all for today. I can't wait to check out the rest of the new features included in this release of ISE Steroids!