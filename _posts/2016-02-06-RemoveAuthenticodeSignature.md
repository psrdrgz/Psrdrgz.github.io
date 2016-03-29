---
layout: post
title: Removing Authenticode Signatures
---
What you would do if you had to remove authenticode signatures from multiple PowerShell scripts?

It's trivial to remove an authenticode signature manually. 

1. Open the file.
2. Cut off the signature portion at the bottom.
3. Save the file.

Boom! Script sans signature. Easy peasy. 

But what if you have to remove signatures from hundreds of PowerShell scripts? I'm sure given your PowerShell prowess you're probably thinking, "I use Set-AuthenticodeSignature for signing. Maybe there's a Remove-AuthenticodeSignature cmdlet?" Although I like the way you're thinking, I'm sorry to say there is no such cmdlet out of the box.

But fret not! You can always use the Remove-Signature function shown below.

```powershell
function Remove-Signature
{
    [cmdletbinding()]

    Param(
        [Parameter(Mandatory = $False,Position = 0,ValueFromPipeline = $True,ValueFromPipelineByPropertyName = $True)]
        [Alias('Path')]
        [system.io.fileinfo[]]$FilePath
    )

    Begin{
        Push-Location -Path $env:USERPROFILE
    }

    Process{
        $FilePath |
        ForEach-Object -Process {
            $Item = $_
			
            If($Item.Extension -match '\.ps1|\.psm1|\.psd1|\.ps1xml')
            {
                Try
                {
                    $Content = Get-Content -Path $Item.FullName -ErrorAction Stop
    
                    $StringBuilder = New-Object -TypeName System.Text.StringBuilder -ErrorAction Stop
    
                    Foreach($Line in $Content)
                    {
                        If($Line -match '^# SIG # Begin signature block|^<!-- SIG # Begin signature block -->')
                        {
                            Break
                        }
                        Else
                        {
                            $null = $StringBuilder.AppendLine($Line)
                        }
                    }
    
                    Set-Content -Path $Item.FullName -Value $StringBuilder.ToString()
                }
                Catch
                {
                    Write-Error -Message $_.Exception.Message
                }
            }
        }
    }

    End{
        Pop-Location
    }
}
```