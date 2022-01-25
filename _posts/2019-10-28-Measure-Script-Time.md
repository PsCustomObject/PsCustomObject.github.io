---
title: "PowerShell Measure Script execution time"
excerpt: "Learn how to measure execution time of PowerShell scripts and commands using different techniques and approaches"
last_modified_at: 2019-11-01T17:27:48-05:00
categories:
  - PowerShell
  - HowTo
tags:
  - PowerShell
  - Measure-Command
  - PowerShell Timer
  - PowerShell StopWatch
  - StopWatch how-to
  - PowerShell Execution Time
---

Sometime it is useful or necessary to measure time it takes PowerShell to execute a script or block of code. Maybe you need to calculate how often you can schedule a script, maybe curiosity on how long it will take to run your code or some other reason.

PowerShell offers different approaches to this which I am going to explore in this post.

## Method 1 - Measure-Command cmdlet

I will start with the built-in method the **[Measure-Command](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/measure-command?view=powershell-6)** cmdlet which can be invoked like in the following example

```powershell
# Count from 0 to 10000 and measure
Measure-Command {0..10000 | ForEach-Object {$i++} }

Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 0
Milliseconds      : 148
Ticks             : 1482658
TotalDays         : 1.71603935185185E-06
TotalHours        : 4.11849444444444E-05
TotalMinutes      : 0.00247109666666667
TotalSeconds      : 0.1482658
TotalMilliseconds : 148.2658
```

The cmdlet will return a *timespan* object which can easily be assigned to a variable like this: 

```powershell
[timespam]$executionTime = Measure-Command {0..10000 | ForEach-Object {$i++} }
```

This way it is easy to retrieve desired value for elapsed time

```powershell
$executionTime.Milliseconds

148
```

Keep in mind, depending on how busy your CPU is, launching the same command over and over could yield slightly different results for each run even with a simple code as the one above.

## Method 2 - Calculate difference in a [datetime] object

I use, *abuse* would be a better fit, the **Get-Date** cmdlet and measuring script execution time is no exception to this rule, using again the same code as in the previous example it could be written like this

```powershell
[int]$startMs = (Get-Date).Millisecond

0..10000 | ForEach-Object { $i++ }

[int]$endMs = (Get-Date).Millisecond

# Calculate elapsed time
Write-Host $($startMs - $endMs)
```

In the above example we initialise a variable containing begin of operation milliseconds, again have PowerShell count from 1 to 10000, set another variable at the end of the cycle and finally print out difference between the start and end time giving us elapsed milliseconds.

**Warning Notice:** I personally discourage, and never use myself, this approach for different reasons. *First* it makes code more difficult to read and generally **slower** due use of multiple variables and extra calculation. *Second* it poses problems with the elapsed time itself, what if the example above  would require 5 seconds to execute? Would it still work according to you?
{: .notice--warning}

## Method 3 - The The .NET **Stopwatch** class

The last method is even my preferred one, the **[stopwatch class](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.stopwatch?view=netframework-4.8)** which works exactly like a real life stopwatch, you start it, it will keep marking execution time and finally you stop it getting back execution time.

PowerShell does not have a native cmdlet for this but you can use an *accelerator* to instantiate a new *stopwatch* instance with the following syntax

```powershell
# Instantiate and start a new stopwatch
$stopwatch = [System.Diagnostics.Stopwatch]::StartNew()

# Get relevant properties and methods
$stopwatch | Get-Member

TypeName: System.Diagnostics.Stopwatch

Name                MemberType Definition
----                ---------- ----------
Equals              Method     bool Equals(System.Object obj)
GetHashCode         Method     int GetHashCode()
GetType             Method     type GetType()
Reset               Method     void Reset()
Restart             Method     void Restart()
Start               Method     void Start()
Stop                Method     void Stop()
ToString            Method     string ToString()
Elapsed             Property   timespan Elapsed {get;}
ElapsedMilliseconds Property   long ElapsedMilliseconds {get;}
ElapsedTicks        Property   long ElapsedTicks {get;}
IsRunning           Property   bool IsRunning {get;}

```

The above example shows the *stopwatch* class has an *Elapsed* properties which will return a timespan object containing the same information we observed in the first example making it easy to gather information about performances of our script, let's see this in action (keep in mind I've left it running for a bit so this is not representative of actual performances)

```powershell
PS C:\Users\lethe> $stopwatch.Elapsed

Days              : 0
Hours             : 0
Minutes           : 4
Seconds           : 14
Milliseconds      : 498
Ticks             : 2544984068
TotalDays         : 0.00294558341203704
TotalHours        : 0.0706940018888889
TotalMinutes      : 4.24164011333333
TotalSeconds      : 254.4984068
TotalMilliseconds : 254498.4068
```

As with the previous examples we can easily assign the whole object or some of its properties to a variable

```powershell
[int]$elapsedSecods = $stopwatch.Elapsed.Seconds

$elapsedSecods

14
```

Once script execution is complete we can simply stop the stopwatch, pun not intended, using the **stop()** method

```powershell
$stopwatch.Stop()
```

This will not produce any output so it is up to us saving details of the stopwatch and eventually communicating this back to the user.

Among the various methods described in this post this is my preferred as it is the less performant intensive and giving precise results.

As PowerShell does not include a built-in cmdlet for this I ended up writing my own function, part of my [IT-ToolBox PowerShell Module](https://github.com/PsCustomObject/IT-ToolBox) that I use daily in my scripts.

If you prefer to download the single functions/cmdlets I have created a dedicated repository where I am publishing functions making up my module or as standalone bits of code that can be found here [PowerShell Functions Repository](https://github.com/PsCustomObject/PowerShell-Functions) keep an eye on it as I will be adding new code and functions regularly.
