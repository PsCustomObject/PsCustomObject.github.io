---
title: "PowerShell Create Folder if it does not exist"
excerpt: "Learn how PowerShell can check if a folder exists and automatically create it if not."
categories:
  - PowerShell
  - HowTo
tags:
  - PowerShell
  - PowerShell Basics
  - HowTo

toc: true
comments: true
---

This will be a short post regarding a question I get or see asked rather frequently and that is worth writing about.

## PowerShell Check if folder exists

Sometimes when interacting with filesystem, for example when writing a log file to disk, it is important making sure destination folder we are going to write files to really exists to avoid PowerShell throwing an exception or, worse, losing data.

PowerShell and PowerShell Core have a native cmdlet to check existence of a given path

```powershell
Test-Path -Path 'C:\Temp\'
```

The *Test-Path* cmdlet will take a path as argument and return a *True* (boolean value) if the given path exists. This can easily be used in scripts to check if a given folder exists and create if not like in the following example

```powershell
# Define where to store logs
[string]$logPath = 'C:\MyLogPath\'

# Create folder if does not exist
if (!(Test-Path -Path $logPath))
{
    $paramNewItem = @{
        Path      = $logPAth
        ItemType  = 'Directory'
        Force     = $true
    }

    New-Item @paramNewItem
}
```

This is something I do so often that I have put it in one of my script templates so not to worry about it every time I write code for a new script.

## Check if folder exists the .Net Way

The *Test-Path* cmdlet is calling a built-in .Net function under the hood, in case you want to impress your friends the above can be rewritten like this

```powershell
# Define where to store logs
[string]$logPath = 'C:\MyLogPath\'

# Create folder if does not exist
if (!([System.IO.Directory]::Exists($logPath))
{
    [system.io.directory]::CreateDirectory($logPath)
}
```

## Closing Notes

The two methods to check if a folder exists I have described in the article are equivalent. Second approach is slightly more performant but risk to make your code less readable.

Given the negligible performance gain I tend to prefer the first method as makes code much clear for somebody who's taking over or modifying my work.
