---
title: "PowerShell Restore Default Gallery Repository"
excerpt: "Restore default PowerShell Gallery repository in PowerShell and PowerShell Core"
categories:

  - PowerShell
  - Tips
tags:
  - PowerShell
  - PowerShell Core
  - Tips
---

When installing PowerShell or PowerShell core on a system it comes configured with a default repository for [PowerShell Gallery](https://www.powershellgallery.com/) which can be used to install modules without any further configuration.

Yesterday I was installing the [AZ Module](https://docs.microsoft.com/en-us/powershell/azure/new-azureps-module-az?view=azps-2.0.0) on my Mac machine but when trying to install the module all I got was the following error

```powershell
Install-Module -Name Az

PackageManagement\Install-Package : No match was found for the specified search criteria and module name 'Az'. Try Get-PSRepository to see all available registered module repositories.
At /usr/local/microsoft/powershell/6/Modules/PowerShellGet/PSModule.psm1:9349 char:21
+ ...          $null = PackageManagement\Install-Package @PSBoundParameters
+                      ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : ObjectNotFound: (Microsoft.PowerShel\u2026lets.InstallPackage:InstallPackage) [Install-Package], Exception
+ FullyQualifiedErrorId : NoMatchFoundForCriteria,Microsoft.PowerShell.PackageManagement.Cmdlets.InstallPackage
```

Issuing a *Get-PSRepository* was yielding the following:

```powershell
Get-PSRepository

WARNING: Unable to find module repositories.
```
Apparently I accidentally removed the repository from my system and trying to reinstate it using the standard way of registering a new repository like this

```powershell
$Repository = @{
    Name = 'PSGallery'
    SourceLocation = 'https://www.powershellgallery.com/api/v2/'
    PublishLocation = 'https://www.powershellgallery.com/api/v2/package/'
    ScriptSourceLocation = 'https://www.powershellgallery.com/api/v2/items/psscript'
    ScriptPublishLocation 'https://www.powershellgallery.com/api/v2/package/'
    InstallationPolicy = 'Untrusted'
}

Register-PSRepository @Repository
```

Was just yielding the following [exception](https://pscustomobject.github.io/powershell/PowerShell-Error-Handling/)

```powershell
Register-PSRepository : Use 'Register-PSRepository -Default' to register the PSGallery repository.
```

It turns out to restore the default PowerShell Gallery repository the only required command is

```powershell
Register-PSRepository -Default
```

This will return no output but you can easily verify it worked like this

```powershell
Get-PSRepository

Name                      InstallationPolicy   SourceLocation
----                      ------------------   --------------
PSGallery                 Untrusted            https://www.powershellgallery.c…
```

Hope this will help in case you accidentally remove the default PSGallery repository.
