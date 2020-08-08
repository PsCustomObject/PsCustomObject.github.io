---
title: "PowerShell run script as administrator"
excerpt: "How to write a PowerShell script that will self elevate the session if it is not being run
as administrator"
categories:

  - PowerShell
  - HowTo
tags:
  - PowerShell
  - Tips
  - Tutorials
  - PowerShell elevate script
  - PowerShell elevation
---

## Right-Click to elevate

There are situation where it is desirable or even required to run a script in an elevated PowerShell session. While it is rather easy to right-click the PowerShell launching link and select *Run as Administrator* there is a better way to handle this.

## Am I elevated?

First of all we can easily find out if current PowerShell session is elevated with the following command:

```powershell
# Check if session is elevated
(New-Object Security.Principal.WindowsPrincipal([Security.Principal.WindowsIdentity]::GetCurrent())).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)

# Session is not elevated
False
```

Arguably the above command will true when run within an elevated session or, like this case, false for a non elevated one. We can easily assign the return value to a *boolean* variable for use like this:

```powershell
[bool]$isElevated = (New-Object Security.Principal.WindowsPrincipal([Security.Principal.WindowsIdentity]::GetCurrent())).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)

if ($isElevated)
{
  Write-Host 'Session is elevated!'
}
else
{
  Write-Warning 'Session is not elevated!'
}
```

## Self Elevating PowerShell Session

Armed with our newly acquired knowledge we can now create a self elevating PowerShell script which will elevate the session automatically if it now

```powershell
# Check if session is elevated
[bool]$isElevated = (New-Object Security.Principal.WindowsPrincipal([Security.Principal.WindowsIdentity]::GetCurrent())).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)

if ($isElevated)
{
	Write-Host 'Session is elevated - We can Continue'
}
else
{
	# Start a new elevated PowerShell session
	Start-Process -FilePath PowerShell.exe -Verb Runas
}
```

Of course you can even specify a variable holding your commands and use it in conjunction with the *-ArgumentList* parameter to specify which command should be run.

## Elevate PowerShell session as another user

While not as common as the previous scenario it happened to me requiring to launch an elevated session as *another user* this can be done via the following command

```powershell
Start-Process powershell.exe -Credential "Domain\User1" -ArgumentList "Start-Process powershell_ise.exe -Verb RunAs"
```

The above will launch an elevated ISE session under the authentication context of *User1* which can be handy for example to run scripts under the context of that user without having to log off and again with the correct credentials.
