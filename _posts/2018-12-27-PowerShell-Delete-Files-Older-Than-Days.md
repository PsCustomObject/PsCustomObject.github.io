---
title: "PowerShell delete old files"
excerpt: "Learn how you can use PowerShell to automatically delete files and folders older than a number of days"
categories:
  - PowerShell
  - HowTo
  - Scripts
tags:
  - PowerShell
  - PowerShell Basics
  - HowTo
  - Delete Files
  - PowerShell Core
---

## The requirement

When dealing with automation it is essential to log each operation taken by scripts so that it is possible to easily find what went wrong and when with a specific script. The downside of maintaining log files is that they take up space on the disk and usually you don't want to keep files around forever.

PowerShell allows us to easily delete files older than a specific number of days, in this article I will describe how the process works.

## The goal

Our goal is to find all files under a specific path that have been created more than **90 days** ago. In the article I will use a sample folder with some old files in it to illustrate how the process works.

Before starting to code our solution there are couple of important questions that we need to answer:

- Do we want to recurse in subfolders?
- Which files shall we remove from folder?
- What is the retention period?

Here's a scheme of what I will be using in the article:

| Recurse in subfolders        | File Extension           | Retention in Days  |
| ------------- |:-------------:| -----:|
| Yes      | .log | 90 days |

## Get Relevant files

Once we have defined requirements it's time to start coding our solution, the first step is getting required variables in place:

```powershell
[string]$filePath = 'C:\TestCleanup\'
[string]$fileExtension = '*.log'
[int]$fileAge = 90
[datetime]$ageTimeSpan = (Get-Date).AddDays(-$fileAge)
```

The above will serve as our configuration to instruct the script what to remove. The *$ageTimeSpan* variable is taking our defined file age, 90 days, and subtracting it to the current date so to calculate files' age.

To retrieve a list of relevant files we can use the *Get-ChildItem* cmdlet with desired filters like this:

```powershell
# Get all matching files
[array]$filesToPurge = Get-ChildItem -Path $filePath -Filter $fileExtension -File |
        Where-Object { $_.LastWriteTime -lt $ageTimeSpan }
```

The above will get all files with an extension of *.log* that have been written more than *90 days* ago (less than current date minus 90 days).

## Process Files to remove

Once we have the list of files all we have to do is cycling through them and remove stale ones which is easily accomplished like:

```powershell
foreach ($file in $filesToPurge)
{
    # Get file full path
    [string]$fileName = $file.FullName

    # Remove file without confirmation
    Remove-Item $fileName -Confirm:$false
}
```

The above code has an issue though. Maybe no file in defined folder is older that the defined time span period while this would not generate any error it is a good idea to add a small clause to our code like this:

```powershell
if ($filesToPurge.Count -gt 0)
{
    foreach ($file in $filesToPurge)
    {
        # Get file full path
        [string]$fileName = $file.FullName

        # Remove file without confirmation
        Remove-Item $fileName -Confirm:$false
    }
}
else
{
    # Replace with proper logging
    Write-Host -Object 'No files to purge!'
}
```

You can find the complete code in the following [gist](https://gist.github.com/PsCustomObject/d73c19c85296b6436d9de33ba25197cc)

## Further Script Development

The code in this article is just a minimum working example that can be expanded upon to accommodate needs like:

- Scanning multiple paths
- Ignore or skip specific folders or subfolders
- Support central configuration

There are probably more features that could make the script even more useful and I would love to hear what you think could be useful.

I am developing something to accommodate the above requirements as it is something I do rather frequently, it is not quite ready yet but I will share as soon code has been written and tested.

## Update January 5 2019

I have released a complete solution to perform automatic cleanup of files in folders called **Remove-OldFiles.ps1** it does all that I described in the post in addition to support:

- Central configuration
- Exception notification
- Custom retention policies
- Ignored path(s)

You can find source code either [here](https://github.com/PsCustomObject/Remove-Old-Files) or over [Technet Galleries](https://gallery.technet.microsoft.com/Cleanup-Old-Files-bde3af13)

Hope you can find the script useful and of course if you have any feature you'd like to see implemented don't hesitate to open an issue in Git!
