---
title: "PowerShell - Generate Unique UPN"
excerpt: "In this short I will share two new cmdlets I have developed to generate forest wide unique UPN handling the different edge cases."
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

## Generating Unique User Principal Names (UPNs) in PowerShell

When managing Active Directory (AD) environments, creating unique User Principal Names (UPNs) is a common, and challenging!, task.

This blog post covers two PowerShell functions that help ensure UPN uniqueness: **Test-UPNExist** and **Get-UniqueUPN**. These functions can be particularly useful for automating user creation processes and can be integrated into your automation projects and solutions.

**Note** Both cmdlets are still under active development so do expect some changes, as I progres with development I will update the post. Feel free to report any issue, idea or suggestion so that I can integrate it into the final cmdlet version.

### Function 1: Test-UPNExist

This is a support that is used to check if given UPN already exists in the AD environment.

```powershell
function Test-UPNExist
{
    [CmdletBinding()]
    param (
        [Parameter(Mandatory = $true)]
        [string]$UPN,
        
        [string]$Server
    )

    try
    {
        if ($Server)
        {
            $ldapPath = "LDAP://$Server"
        }
        else
        {
            $forest = [System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest()
            $gc = $forest.FindGlobalCatalog()
            $ldapPath = "GC://$($gc.Name)"
        }
        $domain = New-Object System.DirectoryServices.DirectoryEntry($ldapPath)
        $searcher = New-Object System.DirectoryServices.DirectorySearcher($domain)
        $searcher.SearchScope = "Subtree"
        $searcher.PageSize = 1000
        $searcher.Filter = "(&(objectCategory=person)(userPrincipalName=$UPN))"
        [void]($searcher.PropertiesToLoad.Add("userPrincipalName"))

        $result = $searcher.FindOne()
        return $null -ne $result
    }
    catch
    {
        Write-Error "Error checking UPN existence: $_"
        throw
    }
}

```

Here's a summary explanation of the parameters:

- **Parameters**
  - *$UPN* - A string representing the UPN to check
  - *$Server* -  A string representing the name of the  LDAP server to query. Parameter is optional and if omitted function will automatically select the closest global catalog server
  
Here's an example usage:

```powershell
$upnExists = Test-UPNExist -UPN "john.doe@example.com"

if ($upnExists -eq $true) # Redundant I know but I like to make code redeable :-)
{
    Write-Host "The UPN exists."
} 
else 
{
    Write-Host "The UPN does not exist."
}

```

### Function 1: Test-UPNExist

This function generates a unique UPN based on given name components and ensures it doesn't already exist in the AD forest.

```powershell
function Get-UniqueUPN
{
    [CmdletBinding()]
    param (
        [Parameter(Mandatory = $true, ParameterSetName = "ADObject")]
        [object]$ADObject,
        
        [Parameter(Mandatory = $true, ParameterSetName = "Strings")]
        [string]$FirstName,
        
        [Parameter(Mandatory = $true, ParameterSetName = "Strings")]
        [string]$LastName,
        
        [Parameter(ParameterSetName = "Strings")]
        [string]$MiddleName,
        
        [Parameter(Mandatory = $true)]
        [string]$UPNSuffix,
        
        [string]$FirstNameFormat = "Full",
        [switch]$IncludeMiddleName,
        [string]$DuplicateSuffix = "Numeric",
        [string]$CustomDuplicateSuffix,
        [string]$Server,
        [string]$Separator = "."
    )

    try
    {
        if ($PSCmdlet.ParameterSetName -eq "ADObject")
        {
            Write-Verbose "Processing AD Object"
            switch ($ADObject.GetType().FullName)
            {
                "Microsoft.ActiveDirectory.Management.ADUser"
                {
                    $firstName = $ADObject.GivenName
                    $lastName = $ADObject.Surname
                    $middleName = $ADObject.MiddleName
                    break
                }
                "System.DirectoryServices.DirectoryEntry"
                {
                    $firstName = $ADObject.Properties["givenName"][0]
                    $lastName = $ADObject.Properties["sn"][0]
                    $middleName = $ADObject.Properties["middleName"][0]
                    break
                }
                "System.DirectoryServices.SearchResult"
                {
                    $firstName = $ADObject.Properties["givenName"][0]
                    $lastName = $ADObject.Properties["sn"][0]
                    $middleName = $ADObject.Properties["middleName"][0]
                    break
                }
                default
                {
                    throw "Unsupported AD object type: $($ADObject.GetType().FullName)"
                }
            }
        }
        else
        {
            Write-Verbose "Processing string inputs"
            $firstName = $FirstName
            $lastName = $LastName
            $middleName = $MiddleName
        }

        Write-Verbose "First Name: $firstName, Last Name: $lastName, Middle Name: $middleName"

        $firstName = switch ($FirstNameFormat)
        {
            "Full" { $firstName }
            "FirstLetter" { $firstName.Substring(0, 1) }
            default { $firstName }
        }

        $middleNamePart = if ($IncludeMiddleName -and $middleName)
        {
            "$Separator$middleName"
        }
        else { "" }

        $baseUPN = "$firstName$middleNamePart$Separator$lastName@$UPNSuffix".ToLower()
        Write-Verbose "Base UPN: $baseUPN"

        $uniqueUPN = $baseUPN
        $counter = 1

        while (Test-UPNExist -UPN $uniqueUPN -Server $Server)
        {
            Write-Verbose "UPN $uniqueUPN already exists, generating alternative"
            if ($DuplicateSuffix -eq "Numeric")
            {
                $uniqueUPN = "{0}{1}@{2}" -f ($baseUPN.Split('@')[0]), $counter, $UPNSuffix
            }
            else
            {
                $uniqueUPN = "{0}{1}@{2}" -f ($baseUPN.Split('@')[0]), $CustomDuplicateSuffix, $UPNSuffix
            }
            $counter++
        }

        Write-Verbose "Final Unique UPN: $uniqueUPN"
        return $uniqueUPN
    }
    catch
    {
        Write-Error "Error generating UPN: $_"
        throw
    }
}
```

Here's a summary explanation of the parameters:

- **Parameters**
  - *$ADObject* - We can pass an existing AD object to the function, for example output from the *Get-AdUser* cmdlet
  - *$FirstName* - A string representing the *First Name* of a given user
  - *$LastName* - A string representing the *Last Name* of a given user
  - *MiddleName* - A string representing the *Middle Name* of a given user (of course optional)
  - *$UPNSuffix* - A string representing the domain suffix for the UPN
  - *FirstNameFormat* - The format of the first name in the UPN (e.g. Full or First letter of name)
  - *IncludeMiddleName* - Switch parameter indicating to cmdlet if Middle Name should be used or not in the UPN generation
  - *DuplicateSuffix* - By default if a duplicate is found a progressive number is added to the UPN, using this paramter you can specify a *custom* character to use
  - *CustomDuplicateSuffix* - Allows us to specify the custom suffix for duplicates
  - *$Server* - A string representing the name of the  LDAP server to query. Parameter is optional and if omitted function will automatically select the closest global catalog server
  - *$Separator* - The separator to use between the name parts. If not specified a '.' (dot) is used

Below you can find couple examples of functions at work

```powershell
# Using string parameters
$uniqueUPN = Get-UniqueUPN -FirstName "John" -LastName "Doe" -UPNSuffix "example.com" -FirstNameFormat "FirstLetter" -IncludeMiddleName -MiddleName "A"

Write-Host "Generated UPN: $uniqueUPN"

# Using an AD object
$adUser = Get-ADUser -Identity "johndoe"
$uniqueUPN = Get-UniqueUPN -ADObject $adUser -UPNSuffix "example.com"

Write-Host "Generated UPN: $uniqueUPN"
```

## Conclusion

By using **Test-UPNExist** and **Get-UniqueUPN**, you can automate the process of generating unique UPNs for users in your Active Directory environment.

These functions ensure that each UPN is unique and adhere to the naming conventions you specify.

This approach minimizes the risk of conflicts and simplifies user management.

As mentioned both functions are still under development and I need to cleanup code and optimize some parts of it but the core functionality is there.

As usual all the code is available on my [GitHub repository](https://github.com/PsCustomObject/PowerShell-Functions).

I would love to receive feedback, ideas or implemntation ideas for them!
