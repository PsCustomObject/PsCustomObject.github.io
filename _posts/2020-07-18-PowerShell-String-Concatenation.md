---
title: "PowerShell string concatenation"
excerpt: "Concatenating strings in PowerShell is a common task in this short post I will give some tips to avoid the performance penalty associated with the task"
categories:
  - PowerShell
  - HowTo
tags:
  - PowerShell
  - Performance Tips
---

Many times when writing a PowerShell script I find myself manipulating lot of strings, for example appending *html* code to a notification mail body, to build what the final value I want to be.

The most common way of doing this is using *concatenation* with the most common form being something similar to this:

```powershell
[string]$myString = $null

# Logic here

$myString += $someValue

# Additional logic here

$myString += $someOtherValue
```

This is perfectly valid PowerShell code and will execute without any issue but there is a catch. **Strings** are immutable objects, this is the same in many languages like Python for example, and as such *read only*.

This is taken from [MSDN](https://docs.microsoft.com/en-us/dotnet/api/system.string?view=netcore-3.1#Immutability) which makes a great job, far better than I can, to explain what is really happening behind the scenes when using the above approach:

> A String object is called immutable (read-only), because its value cannot be modified after it has been created. Methods that appear to modify a String object actually return a new String object that contains the modification.
>
> Because strings are immutable, string manipulation routines that perform repeated additions or deletions to what appears to be a single string can exact a significant performance penalty. Although the code appears to use string concatenation to append a new character to the existing string named str, it actually creates a new String object for each concatenation operation.

*TLTR* **It's magic.**

I am just joking of course but we can easily benchmark the above statement with a code similar the following:

```powershell
# Iterate 100000 and add text to myString
Measure-Command {$myString=$null; for($i=0; $i -lt 100000; $i++){$myString+='Some text'}}

Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 15
Milliseconds      : 224
Ticks             : 152242041
TotalDays         : 0.000176206065972222
TotalHours        : 0.00422894558333333
TotalMinutes      : 0.253736735
TotalSeconds      : 15.2242041
TotalMilliseconds : 15224.2041
```

So is there a better way to implement the above?

## Enter StringBuilder

*StringBuilder* is a class that was designed to solve the issue we just described and documentation can be found [here](https://docs.microsoft.com/en-us/dotnet/api/system.text.stringbuilder?view=netcore-3.1).

To use *StringBuilder* you first need to instantiate an object like this:

```powershell
# Crete new string builder
[System.Text.StringBuilder]$myStringBuilder = [System.Text.StringBuilder]::new()

# Alternative/legacy way
$myStringBuilder = New-Object -TypeName 'System.Text.StringBuilder'
```

Once the object has been created you can use the *Append* or *AppendLine* methods to concatenate strings, here's an example:

```powershell
[void]($myStringBuilder.Append('Text1'))
[void]($myStringBuilder.Append('Text2'))
```

This will append the desired text to the *StringBuilder* object returning it, so the last step to use our string would be using the *.ToString()* method to retrieve content:

```powershell
# Get content from string builder and assign to a string
[string]$myString = $myStringBuilder.ToString()
```

The extra code and conversion step is well worth the effort let's see this in action repeating the same example we ran earlier

```powershell
Measure-Command {$myStringBuilder=New-Object -TypeName System.Text.StringBuilder;for($i=0; $i -lt 100000; $i++){$null=$myStringBuilder.Append('Some text')} $myString = $myStringBuilder.ToString()}

Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 0
Milliseconds      : 239
Ticks             : 2394890
TotalDays         : 2.77186342592593E-06
TotalHours        : 6.65247222222222E-05
TotalMinutes      : 0.00399148333333333
TotalSeconds      : 0.239489
TotalMilliseconds : 239.489
```

Can you see the difference? This time command only took **239 milliseconds** compared to *15* seconds it took running this with the *+=* operator.

This could not seem much in a script running unattended but when dealing with larger scripts performing a lot of string concatenation or management this can make a huge difference so keep it in mind when writing your code.

I hope you will find this tip useful.
