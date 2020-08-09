---
title: "PowerShell check if string is a valid Guid"
excerpt: "Releasing a PowerShell function that will check if an input string is a valid Guid or not"
categories:
  - PowerShell
  - Functions
tags:
  - PowerShell
  - PowerShell Functions
---

Sometimes when dealing with external data it is required to validate input we are receiving is in the correct format.

Today I needed to validate user input was in the correct format, specifically a GUID, while PowerShell supports generation of new GUIDs easily with the *New-Guid* cmdlet there is no built-in method to validate input or a string is a valid GUID.

Before moving on describing the solution here's you can easily generate GUIDs via PowerShell

```powershell
# Built in PowerShell cmdlet
New-Guid

Guid
----
a3fc3a10-705b-4177-88e1-d9e2f55ec98f

# Alternate with .Net accellerator
[guid]::NewGuid()
```

The above will both generate a valid GUID but when we need to validate input is in the correct format there is no equivalent reason for which I wrote a small function named **Test-IsGuid** which will return *True* is input string is a valid GUID or *False* in case it is not.

You can find the function code in my [function repository](https://github.com/PsCustomObject/PowerShell-Functions) and here for reference:

```powershell
function Test-IsGuid
{
    [OutputType([bool])]
    param
    (
        [Parameter(Mandatory = $true)]
        [string]$ObjectGuid
    )

    # Define verification regex
    [regex]$guidRegex = '(?im)^[{(]?[0-9A-F]{8}[-]?(?:[0-9A-F]{4}[-]?){3}[0-9A-F]{12}[)}]?$'

    # Check guid against regex
    return $ObjectGuid -match $guidRegex
}
```

Here are couple of examples to show how the function works:

```powershell
# Test valid guid
Test-IsGuid -ObjectGuid '4a6b7fcb-f1e4-42c8-ae6f-d3862dbbcbca'

$True

# Test arbitrary string
Test-IsGuid -ObjectGuid '4444-23-ec-789'

$False

Test-IsGuid -ObjectGuid '(CA761232-ED42-11CE-BACD-00AA0057B223)'

$True
```

Hope you will find this useful!
