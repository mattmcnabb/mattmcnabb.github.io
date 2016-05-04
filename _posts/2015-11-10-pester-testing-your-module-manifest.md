---
layout: post
title: Testing Your Module Manifest With Pester
categories: [Powershell]
---

[Tests]: https://github.com/pester/Pester/blob/master/Pester.Tests.ps1
[Connect]: https://connect.microsoft.com/PowerShell/feedback/details/1541659/test-modulemanifest-the-psmoduleinfo-is-not-updated

I recently ran into a problem when using Pester to test the validity of my Powershell module manifests. My original idea for testing the manifest files came from [Dave Wyatt's tests][Tests] for the Pester module itself. The first describe block he uses here runs several tests against the module manifest itself such as whether the manifest is versioned, has a valid GUID, and has a name.

One of the problems I sometimes run into when authoring a module is with exporting functions. Sometimes I'll write a new function and think I'm all set to go but when I import the module the function doesn't exist. After a few face-palms I'll realize that I forgot to add the function to the FunctionsToExport array in the module manifest. With this in mind I wrote a test that checks if the functions that are exported from a module match the functions that are enumerated in the module folder itself:

{% highlight Powershell %}
$ModulePath = Split-Path -Parent $MyInvocation.MyCommand.Path
$ModuleName = (Split-Path -Leaf $MyInvocation.MyCommand.Path) -Replace ".Tests.ps1"

$ManifestPath   = "$ModulePath\$ModuleName.psd1"

# test the module manifest - exports the right functions, processes the right formats, and is generally correct
Describe "Manifest" {
    $Manifest = $null
    It "has a valid manifest" {
        {
            $Script:Manifest = Test-ModuleManifest -Path $ManifestPath -ErrorAction Stop -WarningAction SilentlyContinue
        } | Should Not Throw
    }

    It "has a valid name" {
        $Script:Manifest.Name | Should Be $ModuleName
    }

	It "has a valid root module" {
        $Script:Manifest.RootModule | Should Be "$ModuleName.psm1"
    }

	It "has a valid Description" {
        $Script:Manifest.Description | Should Not BeNullOrEmpty
    }

    It "has a valid guid" {
        $Script:Manifest.Guid | Should Be '9dd7e9a4-8525-4fd1-aa13-3a063df4b264'
    }

	It "has a valid prefix" {
		$Script:Manifest.Prefix | Should Not BeNullOrEmpty
	}

	It "has a valid copyright" {
		$Script:Manifest.CopyRight | Should Not BeNullOrEmpty
	}

	It 'exports all public functions' {
		$FunctionFiles = Get-ChildItem "$ModulePath\Scripts\Public" -Filter *.ps1 | Select -ExpandProperty BaseName
		$FunctionNames = $FunctionFiles | foreach {$_ -replace '-', "-$($Script:Manifest.Prefix)"}
		$ExFunctions = $Script:Manifest.ExportedFunctions.Values.Name
		foreach ($FunctionName in $FunctionNames)
		{
			$ExFunctions -contains $FunctionName | Should Be $true
		}
	}
}
{% endhighlight %}

So running these tests against my module returned all green and we were all set, except I noticed a problem - after a successful test run, if I changed the FunctionsToExport value of the manifest to what should be a failing state, such as by commenting out one of the function names, the test still returned a success status. This had me baffled! What I assumed was happening was that Pester tests had some kind of wonky scope rules that I just couldn't reconcile against what I know about scopes in Powershell.

After much playing around with Describe, Context and It blocks in Pester, though, I came to the conclusion that the problem had nothing to do with scope at all - the problem was with the $Manifest variable I had created using Test-ModuleManifest! Testing outside of Pester I found that the Test-ModuleManifest cmdlet only really works once in a Powershell session and afterwards the output will never change, regardless of the state of the manifest file. Run it once, change a property of the manifest and run it again and you'll get the same output. So when I commented out one of the function names in my manifest file the Test-ModuleManifest cmdlet was still including it in the list of exported functions and I would continue to get success test results.

A bug for this behavior has already been filed on [Microsoft Connect][Connect]. So to get around this behavior I decided to test the manifest file simply by importing it as a hash table (since that's what a module manifest really is) and then test the values of each key of that hash table:

{% highlight Powershell %}
$ModulePath = Split-Path -Parent $MyInvocation.MyCommand.Path
$ModuleName = (Split-Path -Leaf $MyInvocation.MyCommand.Path) -Replace ".Tests.ps1"

$ManifestPath = "$ModulePath\$ModuleName.psd1"

# test the module manifest - exports the right functions, processes the right formats, and is generally correct
Describe "Manifest" {
    
    $ManifestHash = Invoke-Expression (Get-Content $ManifestPath -Raw)
    
    It "has a valid manifest" {
        {
            $null = Test-ModuleManifest -Path $ManifestPath -ErrorAction Stop -WarningAction SilentlyContinue
        } | Should Not Throw
    }
    
    It "has a valid root module" {
        $ManifestHash.RootModule | Should Be "$ModuleName.psm1"
    }
    
    It "has a valid Description" {
        $ManifestHash.Description | Should Not BeNullOrEmpty
    }
    
    It "has a valid guid" {
        $ManifestHash.Guid | Should Be '9dd7e9a4-8525-4fd1-aa13-3a063df4b264'
    }
    
    It "has a valid prefix" {
        $ManifestHash.DefaultCommandPrefix | Should Not BeNullOrEmpty
    }
    
    It "has a valid copyright" {
        $ManifestHash.CopyRight | Should Not BeNullOrEmpty
    }

    It "exports all public functions" {
        $ExFunctions = $ManifestHash.FunctionsToExport
        $FunctionFiles = Get-ChildItem "$ModulePath\Scripts\Public" -Filter *.ps1 | Select-Object -ExpandProperty BaseName
        $FunctionNames = $FunctionFiles
        foreach ($FunctionName in $FunctionNames)
        {
            $ExFunctions -contains $FunctionName | Should Be $true
        }
    }
}
{% endhighlight %}

I still run the Test-ModuleManifest cmdlet just to get a check on the overall validity of the manifest, but I don't base my other tests on the result of this cmdlet. Instead I use this line:

{% highlight Powershell %}
$ManifestHash = Invoke-Expression (Get-Content $ManifestPath -Raw)
{% endhighlight %}

To output a hash table for testing. So how are you testing your modules with Pester? And if you aren't using Pester, you'd better get started!

P.S. Thanks to Jakub Jares and Dave Wyatt for helping me discover the issue with Test-ModuleManifest.
