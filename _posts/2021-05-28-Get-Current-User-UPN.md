---
title: "Get current user UPN"
excerpt: "In this post we will explore how to easily retrieve currently logged in user UPN via PowerShell"
categories:
  - PowerShell
  - Tips
  - HowTo

tags:
  - PowerShell
  - Tips

toc: false
header:
    teaser: "/assets/images/PowerShell_Logo.png"
---

A few days ago while developing a cmdlet for an internal module in support to a larger automation workflow I found myself in need to easily derive *UserPrincipalName* of the currently logged on user running the command.

If you look  this up in your favorite search engine chances are you came across results like the following (there are more but these are the more common *suggestions* from users):

```powershell
# 1
$env:UserName

# 2
[System.Security.Principal.WindowsIdentity]::GetCurrent().Name

# 3
[Environment]::UserName

# 4
$(Get-WMIObject -class Win32_ComputerSystem | select username).username
```

All the above methods are more or less equivalents and will yield the following results:

1. PsCustomObject  - Which is my samAccountName
2. domain\PsCustomObject - Same as above just with the domain prefix
3. PsCustomObject - Identical to number 1
4. domain\PsCustomObject - Identical to number 2

Issue is none of them will easily return the **UserPrincipalName** of the currently logged in user.

This can easily be achieved with the following command:

```powershell
([ADSI]"LDAP://<SID=$([System.Security.Principal.WindowsIdentity]::GetCurrent().User.Value)>").UserPrincipalName
```

This command, despite not as user friendly as the other commands, will return the full UPN of the user for example **PsCustomObject@domain.com**.

I encourage you to explore the other available methods and properties of the *[System.Security.Principal.WindowsIdentity]* [class](https://docs.microsoft.com/en-us/dotnet/api/system.security.principal.windowsidentity?view=net-5.0) as it can be really handy when trying to get details about the current user.