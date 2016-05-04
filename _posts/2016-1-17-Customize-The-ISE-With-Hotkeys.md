---
layout: post
title: Use Hotkeys to Customize the Powershell ISE
categories: [Powershell]
---

[PowerTip]: http://powershell.com/cs/blogs/tips/archive/2015/12/08/detecting-key-presses-across-applications.aspx

### Detecting Key Presses

I haven't blogged in quite a while, but recently I was inspired by this [Powershell.com PowerTip][PowerTip] that gives us a method to detect key presses using low-level Windows API methods. I took this just a bit further and created a function that allows us to specify exactly which key we'd like to test for and will even check for multiple simultaneous keys (a chord).

{% highlight Powershell %}
#requires -Version 3
function Test-KeyPress
{
    <#
            .SYNOPSIS
            Checks to see if a key or keys are currently pressed.

            .DESCRIPTION
            Checks to see if a key or keys are currently pressed. If all specified keys are pressed then will return true, but if 
            any of the specified keys are not pressed, false will be returned.

            .PARAMETER Keys
            Specifies the key(s) to check for. These must be of type "System.Windows.Forms.Keys"

            .EXAMPLE
            Test-KeyPress -Keys ControlKey

            Check to see if the Ctrl key is pressed

            .EXAMPLE
            Test-KeyPress -Keys ControlKey,Shift

            Test if Ctrl and Shift are pressed simultaneously (a chord)

            .LINK
            Uses the Windows API method GetAsyncKeyState to test for keypresses
            http://www.pinvoke.net/default.aspx/user32.GetAsyncKeyState

            The above method accepts values of type "system.windows.forms.keys"
            https://msdn.microsoft.com/en-us/library/system.windows.forms.keys(v=vs.110).aspx

            .NOTES
            Original idea found here at http://powershell.com/cs/blogs/tips/archive/2015/12/08/detecting-key-presses-across-applications.aspx

            .INPUTS
            None - Test-KeyPress does not accept pipeline input.

            .OUTPUTS
            System.Boolean
    
    #>
    
    [CmdletBinding()]
    param
    (
        [Parameter(Mandatory)]
        [System.Windows.Forms.Keys[]]
        $Keys
    )
    
    # use the User32 API to define a keypress datatype
    $Signature = @'
    [DllImport("user32.dll", CharSet=CharSet.Auto, ExactSpelling=true)] 
    public static extern short GetAsyncKeyState(int virtualKeyCode); 
'@
    $API = Add-Type -MemberDefinition $Signature -Name 'Keypress' -Namespace Keytest -PassThru
    
    # test if each key in the collection is pressed
    $Result = foreach ($Key in $Keys)
    {
        [bool]($API::GetAsyncKeyState($Key) -eq -32767)
    }
    
    # if all are pressed, return true, if any are not pressed, return false
    $Result -notcontains $false
}
{% endhighlight %}

### How to Use Test-KeyPress
Now I can run `Test-KeyPress -Keys Ctrl,Shift` and when the function executes if both CTRL and SHIFT are pressed it will return true. If either or both of those keys are not pressed it will return false. Obivously this is not intended to be used interactively - if you're running commands at the console prompt then you generally won't have any keys pressed other than ENTER at the time of execution.

So how can this be used you say? I use `Test-KeyPress` to make decisions when either the Powershell console or the ISE are launched. For example, sometimes I want to launch the Powershell ISE with the ISE Steroids add-on ready to go, but other times I want to load just the base ISE. To do that, I added this to my profile script:

{% highlight Powershell %}
if (Test-KeyPress -Keys ControlKey) { Start-Steroids }
{% endhighlight %}

Now when I launch the ISE, if I hold down the control key ISE Steroids will be loaded with no interaction from me. Alternatively you could use `Test-KeyPress` to execute just about any code in your profile so if you would like certain modules, variables, etc. to be conditionally available you can make that happen just by holding down a key or combination of keys.

If you have any other ideas for how to use `Test-KeyPress`, I'd love to hear them! Hit me up on [Twitter](http://twitter.com/{{site.author.twitter}}) to discuss. 