---
layout: post
title: Powershell Studio 2015 Solarized Color Scheme
categories: [Powershell]
author: Matt McNabb
comments: true
---

 [Solarized]: /assets/media/PSStudioSolarized.png

I've been playing around with a trial version of Powershell Studio 2015 an I must say it's pretty nice! I don't typically do GUIs with Powershell, but if I did this would be an amazing tool. Overall there are some great features that really provide some value if you need to move up to a professional editor.

I thought I'd share the color scheme I've been using for the script editor:

![][Solarized]

This is a port of Solarized and it looks pretty good in Powershell Studio! If you'd like to add this as a preset scheme just copy the text below and save it in your presets folder at %appdata%'SAPIEN\PowerShell Studio 2015\Editor Presets.

{% highlight XML %}
<Styles version="1" FontName="Lucida Console" FontSize="10" BackColor="FFFDF6E3" MarginColor="FFF0F0F0">
  <Style name="Text" Bold="False" Italic="False" Underline="False" ForeColor="FF268BD2" BackColor="FFFDF6E3" />
  <Style name="Cmdlet" Bold="False" Italic="False" Underline="False" ForeColor="FF586E75" BackColor="FFFDF6E3" />
  <Style name="Alias" Bold="True" Italic="False" Underline="False" ForeColor="FF0000FF" BackColor="0" />
  <Style name="Reserved Word" Bold="False" Italic="False" Underline="False" ForeColor="FF859900" BackColor="FFFDF6E3" />
  <Style name="Operator" Bold="False" Italic="False" Underline="False" ForeColor="FF93A1A1" BackColor="FFFDF6E3" />
  <Style name="Variable" Bold="False" Italic="False" Underline="False" ForeColor="FFCB4B16" BackColor="FFFDF6E3" />
  <Style name="Number" Bold="False" Italic="False" Underline="False" ForeColor="FF2AA198" BackColor="FFFDF6E3" />
  <Style name="String" Bold="False" Italic="False" Underline="False" ForeColor="FF2AA198" BackColor="FFFDF6E3" />
  <Style name="Comment" Bold="False" Italic="False" Underline="False" ForeColor="FF93A1A1" BackColor="FFFDF6E3" />
  <Style name="Function" Bold="False" Italic="False" Underline="False" ForeColor="FF586E75" BackColor="FFFDF6E3" />
  <Style name="Parameter" Bold="False" Italic="False" Underline="False" ForeColor="FFDC322F" BackColor="FFFDF6E3" />
  <Style name="Command As Parameter" Bold="False" Italic="False" Underline="False" ForeColor="FF00008B" BackColor="0" />
  <Style name="Code Snippet Field" Bold="False" Italic="False" Underline="False" ForeColor="0" BackColor="FFC8FFFF" />
  <Style name="Highlighted Reference" Bold="False" Italic="False" Underline="False" ForeColor="0" BackColor="FFC0C0C0" />
  <Style name="Type" Bold="False" Italic="False" Underline="False" ForeColor="FF6C71C4" BackColor="FFFDF6E3" />
  <Style name="Parameter Attribute" Bold="False" Italic="False" Underline="False" ForeColor="FFB58900" BackColor="FFFDF6E3" />
  <Style name="Unknown Command" Bold="False" Italic="False" Underline="False" ForeColor="FF657B83" BackColor="FFFDF6E3" />
  <Style name="External Tool" Bold="False" Italic="False" Underline="False" ForeColor="FF586E75" BackColor="0" />
</Styles>
{% endhighlight %}

NOTE: I used the font Source Code Pro from Adobe in the screenshot, but since this is not a Windows included font, I used Lucida Console in the preset file. Feel free to change this as you see fit.