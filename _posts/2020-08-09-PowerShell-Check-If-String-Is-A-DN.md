---
title: "PowerShell Validate Distinguished Name"
excerpt: "In this post we will explore how to use PowerShell and some regex magic to check if a string is a valid distinguishedName (DN)"
categories:
  - PowerShell
  - HowTo
  - Identity Management
tags:
  - PowerShell
  - Active Directory
  - Identity Management

header:
    teaser: "/assets/images/AD_Logo.png"

toc: false
---

Any administrator or engineer who ever had to deal with an LDAP database will know this by heart but anyhow a **DistinguishedName(DN)** is a string that uniquely identifies an entry in the Directory Database, in the Microsoft implementation *Active Directory*.

**Note:** I love **[LDAP Wiki explanation on the subject](https://ldapwiki.com/wiki/Distinguished%20Names)**.
{: .notice--primary}

In my [Get users reporting to manager](https://pscustomobject.github.io/powershell/howto/identity%20management/Active-Directory-Get-Report-Chain/) I explored how we can leverage AD OID to get a full report of use users, direct and indirect, reporting to a specific manager and the main argument to our LDAP query was the user's *DN* which raised the question

> How can I validate an input string is a valid object DN?

Finding an answer was not super easy as there are lot of variables to take into account but finally I've been able to come up with the **Test-IsValidDN** function which you can find below

```powershell
function Test-IsValidDN
{
    <#
        .SYNOPSIS
            Cmdlet will check if the input string is a valid distinguishedname.

        .DESCRIPTION
            Cmdlet will check if the input string is a valid distinguishedname.

            Cmdlet is intended as a dignostic tool for input validation

        .PARAMETER ObjectDN
            A string representing the object distinguishedname.

        .EXAMPLE
            PS C:\> Test-IsValidDN -ObjectDN 'Value1'

        .NOTES
            Additional information about the function.
    #>

    [OutputType([bool])]
    param
    (
        [Parameter(Mandatory = $true)]
        [ValidateNotNullOrEmpty()]
        [Alias('DN', 'DistinguishedName')]
        [string]
        $ObjectDN
    )

    # Define DN Regex
    [regex]$distinguishedNameRegex = '^(?:(?<cn>CN=(?<name>(?:[^,]|\,)*)),)?(?:(?<path>(?:(?:CN|OU)=(?:[^,]|\,)+,?)+),)?(?<domain>(?:DC=(?:[^,]|\,)+,?)+)$'

    return $ObjectDN -match $distinguishedNameRegex
}

# Test if string is a valid DN
Test-IsValidDN -ObjectDn 'OU=Sales+CN=J. Smith,DC=example,DC=net'

# Output
$True
```

**Important:** Function has been written with Active Directory DN format in mind, which does not strictly adhere to LDAP standards, so function will not match DNs like *UID=jsmith,DC=example,DC=net* or *1.3.6.1.4.1.1466.0=#04024869,DC=example,DC=com* which are perfectly valid DNs.
{: .notice--danger}

As usual you can find code function in my **[PowerShell Functions Repository](https://github.com/PsCustomObject/PowerShell-Functions)** or in my **[IT-Toolbox](https://github.com/PsCustomObject/IT-ToolBox)** module.
