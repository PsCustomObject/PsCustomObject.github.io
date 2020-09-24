---
title: "Check if computer is connected to Domain Network"
excerpt: "How to check if a computer is connected to the domain network"
categories:
  - PowerShell
  - HowTo
tags:
  - PowerShell
  - Domain Network
  - Tips
  - How to check if computer is connected to domain
---

## Remote or Domain Network?

In our connected world sometimes we need to find out if a computer is connected to the domain network or a remote one for example if we need to run a script **only** when user is in the corporate network.

I had to solve this challenge to accommodate a configuration deployment via SCCM only for users that were physically connected to the domain network via VPN or cable.

Luckily enough the [System.DirectoryServices.ActiveDirectory.Domain](https://docs.microsoft.com/en-us/dotnet/api/system.directoryservices.activedirectory.domain?view=netframework-4.7.2) class allows us to get this information pretty easily with code similar the following:

```powershell
[System.DirectoryServices.ActiveDirectory.Domain]::GetComputerDomain()
```

The above will return information about the forest and domain similar the following if computer can reach a domain controller

```powershell
# Output
Forest                  : test.lab
DomainControllers       : {DC01.test.lab}
Children                : {}
DomainMode              : Windows8Domain
DomainModeLevel         : 5
Parent                  :
PdcRoleOwner            : DC01.test.lab
RidRoleOwner            : DC01.test.lab
InfrastructureRoleOwner : DC01.test.lab
Name                    : test.lab
```

On the other hand if computer is connected to a remote network an exception will be returned.

Putting all together this can easily be turned into a script similar the following:

```powershell
try
{
  # Check if computer is connected to domain network
  [void]::([System.DirectoryServices.ActiveDirectory.Domain]::GetComputerDomain())

  Write-Host 'Domain Network'
}
catch
{
    Write-Host 'Remote Network'
}
```

Using a simple [Try/Catch](https://pscustomobject.github.io/powershell/PowerShell-Error-Handling/) block we have our script verify if computer should execute some code or not.
