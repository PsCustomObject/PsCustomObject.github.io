---
title: "PowerShell Measure Script Execution Time"
excerpt: "Learn how you can measure the time required by a script to complete its
execution."
categories:
  - PowerShell
  - HowTo
tags:
  - PowerShell
  - PowerShell Basics
  - HowTo
  - Measure Script Time
  - PowerShell Core
---

Sometimes when executing a lengthy script, for example one that has external dependencies,it can be useful to measure the time it takes for the script to complete its execution, let's see how this can be achieved.

# The Measure-Command method

The first, and most commonly described, approach is the built-in *Measure-Command* cmdlet that will output a [TimeSpan](https://docs.microsoft.com/en-us/dotnet/api/system.timespan?view=netframework-4.7.2) object value that you can use to measure how much time it required your script took to run.

Here's an example:

```powershell
 # Measure execution time 
 Measure-Command {Get-ChildItem -Path 'Users/PsCustomObject/' -Recurse}
 
 # Output
Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 26
Milliseconds      : 379
Ticks             : 263794614
TotalDays         : 0.000305317840277778
TotalHours        : 0.00732762816666667
TotalMinutes      : 0.43965769
TotalSeconds      : 26.3794614
TotalMilliseconds : 26379.4614
```

The above will get all files under my Desktop folder and print out time required to carry on operation, we can of course wrap everything up to a variable for later user 

```powershell
# Define Search Path
[string]$userFolder = 'Users/PsCustomObject/Desktop/'

# Measure execution time
[timspam]$executionTime =  Measure-Command {Get-ChildItem -Path  $userFolder -Recurse}

Write-Host -Object $executionTime.'Seconds'
```

This approach has the minor downside of *hiding* stdout output so if anything is printed on screen it will be *lost*.

# The Stopwatch method

Luckily enough PowerShell is based on native .Net functions which can be leveraged via **Type Accelerators** (I will write a future article about this) and chances are you already did via something like this

```powershell
[datetime]::now

Wednesday, December 26, 2018 07:27:43 PM
```

Which leads us to the [Stopwatch Class](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.stopwatch?view=netframework-4.7.2) which can be used to measure script's execution time. 

First of all instantiate a new Stopwach object and use the *Start()* method to start the counter

```powershell
# Full Type Name would be System.Diagnostics.Stopwatch

# Start the stopwatch
$stopWatch = [Diagnostics.Stopwatch]::StartNew()
```

Now carry on any action required by the script

```powershell
# Define Search Path
[string]$userFolder = 'Users/PsCustomObject/Desktop/'

# Get all files and recurse in subfolders
Get-ChildItem -Path $userFolder -Recurse
```

Finally alt the stopwatch via the *Stop()* method and print elapsed time

```powershell
# Alt the stopwatch
$stopWatch.Stop()

# Print elapsed time
$stopWatch.'Elapsed'

 # Output
Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 26
Milliseconds      : 379
Ticks             : 263794614
TotalDays         : 0.000305317840277778
TotalHours        : 0.00732762816666667
TotalMinutes      : 0.43965769
TotalSeconds      : 26.3794614
TotalMilliseconds : 26379.4614
```

Object returned by *$stopWatch* is again a **timespan** with the main difference being any output printed to stout.

Full code for the above example can be found in the following *[Gist](https://gist.github.com/PsCustomObject/6c22b6cb040d195f05ebb9df1a772bff)*

# Closing Notes

As we have seen there both methods are equivalent when measuring script's execution time with *Measure-Command* hiding any output that would be otherwise be printed on screen.

As a best practice no script that is running unattended, for example via scheduled task, should ever rely on on-screen output but always use a proper [log](https://github.com/PsCustomObject/New-LogEntry) function so that no output is ever lost. 

If printing information on screen, for example to let the user know which actions are being taken, you can have the log function both write on screen and to a log file so to be able to know exactly which actions have been taken even after script execution.

**Note:** Methods illustrated in this article will work both with PowerShell and PowerShell Core