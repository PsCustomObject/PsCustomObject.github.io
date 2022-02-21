---
title: "PowerShell - Get day of weekday"
excerpt: "In this short post we will see how we can use PowerShell to get an integer representing the number associated with the day of the week for the current date."
categories:
  - PowerShell
  - Howto

tags:
  - PowerShell
  - Tips
  - HowTo

toc: true
header:
  teaser: "/assets/images/PowerShell_Logo.png"
---

## PowerShell Get current date

PowerShell natively supports getting the current, or specific, date via the following cmdlet:

```powershell
Get-Date
```

Which, by default, will produce the following output:

```powershell
Monday, February 21, 2022 10:11:39 PM
```

Cmdlet supports methods which can be used to display/return the day of the week for the selected date

```powershell
(Get-Date).DayOfWeek
```

There are situations where rather than returning a string with the day name it can be useful to return the **number** (1 to 7) associated with the day's name, this can easily be accomplished with the following command

```powershell
(Get-Date).DayOfWeek.value__

# Output
1
```

This is specially handy when inserting data into SQL datbase supporting dates as _TinyInt_ data types.
