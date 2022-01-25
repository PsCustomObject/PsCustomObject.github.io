---
title: "PowerShell convert string to title case"
excerpt: "In this article we will explore how to convert a string in mixed case into a Title Case String."
categories:
  - PowerShell
tags:
  - PowerShell
  - String Conversion
  - PowerShell Title Case

toc: true
---

## String convert case

One of the reasons I love Python is it *comes with batteries included<sup>(TM)</sup>*.

For example to convert a string to title case is as simple as doing this:

```python
myString = "This IS A string WIth MixeD CaSe"

myString.title()

# Output
'This Is A String With Mixed Case'
```

But this is a post about PowerShell so you probably landed here to know how to do this in PowerShell.

## PowerShell string case

Let's start the discussion saying in PowerShell strings are case *insensitive* so the following strings will be identical:

```powershell

$firstString = "A String"
$secondString = "a strinG"

$firstString -eq $SecondString

$true
```

Most of the times this is not an issue but there are cases, for example when generating DisplayNames for users, where you want to *sanitise* the string so that it will use proper **Title Case**.

## PowerShell convert string to Title Case

PowerShell does not natively have a method/cmdlet to convert a string to Title Case but luckily the [TextInfo](https://docs.microsoft.com/en-us/dotnet/api/system.globalization.textinfo.totitlecase?view=netframework-4.8) has a method aptly named **ToTitleCase** that accepts a string as input and produces a *formatted* string that is not necessarily **linguistically correct**.

In PowerShell the *TextInfo* class is accessible via the **Get-Culture** cmdlet

```powershell
(Get-Culture).TextInfo.ToTitleCase
 
OverloadDefinitions
-------------------
string ToTitleCase(string str)
```

Here's an example of how we can use this:

```powershell
$myString = "STRing WIth Mixed CaSe"

# Instantiate required object
$textInfo = (Get-Culture).TextInfo

# Convert String
$textInfo.ToTitleCase($myString)

# Output
String With Mixed Case
```

The above can be expressed like this for brevity:

```powershell
$myString = "STRing WIth Mixed CaSe"

(Get-Culture).TextInfo.ToTitleCase($myString)
```

### Special note on case conversion

As I previously mentioned method not always produces a a *linguistically correct* version of the string, reason for this is pretty simple, algorithm behind it has been created with ease of use and speed in mind.
What does this means exactly?

In a nutshell not always results will be what you are expecting, let's look at a couple of examples:

```powershell
$myString = "STRing WITH Mixed CaSe"

(Get-Culture).TextInfo.ToTitleCase($myString)
```

In the above example expected output would be *String With Mixed Case* but it will really be **String WITH Mixed Case**, why maybe you're wondering.

Answer is method will not convert case for *acronyms* or words all in upper case.

To workaround this *limitation* you can simply use another string method

```powershell

$myString = "STRing WITH Mixed CaSe"

# Convert string to lowercase
$myString = $myString.ToLower()

(Get-Culture).TextInfo.ToTitleCase($myString)
```

The above will produce the expected output as string is being converted to lowercase before using the ToTitleCase method.

Microsoft has some really valid examples in their documentation to remark the fact conversion is not *perfect* and I am pasting here to reiterate what I already wrote above.

| **Input**                      | Language | Expected result                | Actual result                  |
| ------------------------------ | -------- | ------------------------------ | ------------------------------ |
| war and peace                  | English  | War and Peace                  | War And Peace                  |
| Per anhalter durch die Galaxis | German   | Per Anhalter durch die Galaxis | Per Anhalter Durch Die Galaxis |
| les naufragés d'ythaq          | French   | Les Naufragés d'Ythaq          | Les Naufragés D'ythaq          |

I hope you'll find the information in this article useful!
