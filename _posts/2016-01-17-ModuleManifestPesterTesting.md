---
layout: post
title: Pester Testing Your Module Manifest
---
**Update:** Looks like there is a new Import-PowerShellDataFile function available in PSv5 RTM. The function uses the AST similarly to what's shown below. If you want to take a look, feel free to check it out as follows:
+ 
+ ```powershell
+ (get-command Import-PowerShellDataFile).ScriptBlock
```

****
When you create a new public function for your PowerShell module do you sometimes forget to add it to the module manifest like I do? If so, read on!

I can't tell you how many times I've created a fancy new function to add to my custom PowerShell module only to find out later that I can't use it because I didn't update the FunctionsToExport key of my module's manifest. Now I'm sure some of you are thinking, "Why not just use a wildcard and export all functions in the module?" Well in my case I'd prefer not to for two reasons:

+ I have a few helper functions that I'd prefer to be private because they shouldn't necessarily be used on their own.
+ By populating the FunctionsToExport key of the module manifest, someone searching for my module in a Nuget repository can easily see the functions contained within.

So rather than continue to suffer, I decided to set up a Pester test to check whether or not all my public functions are included in the module manifest. Now at first I assumed I could easily set up a Pester test by leverging the ``Test-ModuleManifest`` cmdlet, but unfortunately this was not the case. You see the ``Test-ModuleManifest`` cmdlet has a quirky little bug that doesn't always accurately return the correct information contained within a module manifest. I'm far from the first person to have noticed this issue. In fact, a fellow PowerShell enthusiast by the name of Matt McNabb wrote about this issue on his [blog](http://psescape.azurewebsites.net/pester-testing-your-module-manifest/) when he was attempting to Pester test his module manifest.  

So how did Matt overcome this issue? Well using a combination of ``Get-Content`` and ``Invoke-Expression`` he was able to evaluate the module manifest as a hashtable (because that's all it really is) and validate each key and its value. It's really a very elegant solution, but ...

Using ``Invoke-Expression`` leaves a bad taste in my mouth. Now don't get me wrong, there are certain cases where using it may be unavoidable. However, this is not one of those cases. 

So how then might we tackle this problem? By using the PowerShell Abstract Syntax Tree (AST)! Let's take a look.

```powershell
Describe "ARTools Module Manifest" {
    It "FunctionsToExport key contains all public functions." {
        $ManifestAST = [System.Management.Automation.Language.Parser]::ParseFile("$PSScriptRoot\ARTools.psd1",[ref]$null,[ref]$null)

        $ManifestHashTable = $ManifestAST.FindAll({$args[0] -is [System.Management.Automation.Language.HashtableAst]}, $true).SafeGetValue()

        $FunctionsToExport = $ManifestHashTable.FunctionsToExport

        Get-ChildItem -Path $PSScriptRoot\Public | Select-Object -ExpandProperty BaseName | Where-Object -FilterScript {$_ -notin $FunctionsToExport} | Should Be $Null
    }
}
```
    
As you can see above, it's starts by calling the static ParseFile method of the SMA Language Parser class to parse the PSD1 module manifest file. Using the FindAll method of the AST we can then find all ASTs which represent the hashtables contained within the file. Finally, thanks to the SafeGetValue method we are then able to extract the hashtable. From there we can easily evaluate any Key/Value pair of the manifest as we normally would any hashtable.

If you'd like to know more about parsing PowerShell scripts with the AST, I'd highly recommend you check out the below video presented by Doug Finke.

<iframe width="853" height="480" src="https://www.youtube.com/embed/R8To09xrBMo" frameborder="0" allowfullscreen></iframe><br/>
