---
title: "PowerShell ISE Clear Variables without restarting"
excerpt: "How to clear variables in PowerShell ISE Editor without restarting the session"
categories:

  - PowerShell
  - HowTo
tags:
  - PowerShell
  - PowerShell ISE
  - PowerShell Editors
---

## PowerShell ISE and Variables values

While it is not my *go-to* editor (more on this in a future post) sometimes I am forced to work and debug from an ISE session which has the bad habit of *maintaining* variables values between script executions.

This can create issues especially while developing/changing a script and executing it multiple times and, trust me, it can be really difficult to troubleshoot. I know I've been there already.

Luckily enough there is an easy solution to this which is running the following command which will clear all variables *stored* in the session

```powershell
Remove-Variable * -ErrorAction SilentlyContinue
```

This will take care of clearing all stored variables so we can be sure we script will not *inherit* any value from a previous run.

## PowerShell ISE Errors and Modules

What I just described also applies to the the *Error* buffer which can similarly be cleared with the following command

```powershell
$error.Clear()
```

Taking this a step further we can even remove any module loaded by our script with the following command

```powershell
Remove-Module *
```

Putting all this together we can use a single command to have a *fresh* ISE Session

```powershell
Remove-Variable * -ErrorAction SilentlyContinue; Remove-Module *; $error.Clear();
```
