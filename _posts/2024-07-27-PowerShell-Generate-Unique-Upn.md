---
title: "PowerShell - Generate Unique UPN"
excerpt: "In this short post I will share two new cmdlets I have developed to generate forest wide unique UPN handling the different edge cases."
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
<#
    .SYNOPSIS
        Cmdlet will check if a given UPN exists in the forest.
    
    .DESCRIPTION
        Cmdlet is a diagnostic tool to check if a given UPN is already assigned to a user in the forest.
    
    .PARAMETER UPN
        A string representing the UPN to check for uniqueness.
    
    .PARAMETER AdServer
        A string representing the name of the domain controller to be used for the check, if parameter
        is not specified the closest Global Catalog is used.
    
    .EXAMPLE
        PS C:\> Test-UPNExist -UPN 'John.Doe@example.com'
#>
    
    [CmdletBinding()]
    param
    (
        [Parameter(Mandatory = $true)]
        [ValidateNotNullOrEmpty()]
        [string]$UPN,
        [ValidateNotNullOrEmpty()]
        [string]$AdServer
    )
    
    if ([string]::IsNullOrEmpty($AdServer) -eq $true)
    {
        $adForest = [System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest()
        [string]$ldapPath = '{0}{1}' -f 'GC://', $($adForest.FindGlobalCatalog().Name)
    }
    else
    {
        [string]$ldapPath = '{0}{1}' -f 'LDAP://', $AdServer
    }
    
    # Instantiate required objects and run query
    $adDomain = New-Object System.DirectoryServices.DirectoryEntry($ldapPath)
    $adSearcher = New-Object System.DirectoryServices.DirectorySearcher($adDomain)
    $adSearcher.SearchScope = 'Subtree'
    $adSearcher.PageSize = 1000
    $adSearcher.Filter = "(&(objectCategory=person)(userPrincipalName=$UPN))"
    [void]($adSearcher.PropertiesToLoad.Add("userPrincipalName"))
    
    [array]$searchResult = $adSearcher.FindOne()
    
    return $null -ne $searchResult
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
<#
    .SYNOPSIS
        Cmdlet will generate a forest wide unique UPN.
    
    .DESCRIPTION
        Cmdlet will generate a forest wide unique UPN according to generation rules
        defined by the user.
        
        Cmdlet accept different types of objects to generate the UPN to allow greater flexibility
        
        ADObject - For example and object from Get-AdUser cmdlet
        Strings - Representing First Name, Last Name etc.
        DirectoryService Objects - For example when using native .Net methods to retrieve the identity
    
    .PARAMETER ADObject
        An ADObject for example output of the Get-ADUser cmdlet
    
    .PARAMETER FirstName
        A string representing the First Name of the user
    
    .PARAMETER LastName
        A string representing the Last Name of the user
    
    .PARAMETER MiddleName
        A string representing the Middle Name of the user, parameter is optional.
    
    .PARAMETER UPNSuffix
        A string representing the UPN suffix to be used.
    
    .PARAMETER FirstNameFormat
        A string representing the format to be for the First Name part of the UPN.
    
    .PARAMETER LastNameFormat
        A string representing the format to be for the Last Name part of the UPN.
    
    .PARAMETER IncludeMiddleName
        When paramenter is specified user Middle Name, if present, will be included in the UPN generation process.
    
    .PARAMETER ADServer
        A string representing the name of the AD Domain Controller that will be used to query Active Directory.
    
        If no server is specified the closest Global Catalog will be automatically selected.
    
    .PARAMETER Separator
        A string representing the separator to be used between UPN parts, defaults to a '.'.
#>
    
    [CmdletBinding(DefaultParameterSetName = 'Strings')]
    param
    (
        [Parameter(ParameterSetName = 'ADObject',
                   Mandatory = $true)]
        [object]$ADObject,
        [Parameter(ParameterSetName = 'Strings',
                   Mandatory = $true)]
        [ValidateNotNullOrEmpty()]
        [string]$FirstName,
        [Parameter(ParameterSetName = 'Strings',
                   Mandatory = $true)]
        [ValidateNotNullOrEmpty()]
        [string]$LastName,
        [Parameter(ParameterSetName = 'Strings')]
        [ValidateNotNullOrEmpty()]
        [string]$MiddleName,
        [Parameter(Mandatory = $true)]
        [ValidateNotNullOrEmpty()]
        [string]$UPNSuffix,
        [ValidateSet('FullName', 'FirstLetter', IgnoreCase = $true)]
        [ValidateNotNullOrEmpty()]
        [string]$FirstNameFormat = 'Full',
        [ValidateSet('FullName', 'FirstLetter', IgnoreCase = $true)]
        [ValidateNotNullOrEmpty()]
        [string]$LastNameFormat = 'FullName',
        [switch]$IncludeMiddleName,
        [ValidateNotNullOrEmpty()]
        [string]$ADServer,
        [ValidateNotNullOrEmpty()]
        [string]$Separator = '.'
    )
    
    if ($PSCmdlet.ParameterSetName -eq 'ADObject')
    {
        switch ($ADObject.GetType().FullName)
        {
            'Microsoft.ActiveDirectory.Management.ADUser'
            {
                [string]$firstName = $ADObject.GivenName
                [string]$lastName = $ADObject.Surname
                [string]$middleName = $ADObject.MiddleName
                
                break
            }
            'System.DirectoryServices.DirectoryEntry'
            {
                [string]$firstName = $ADObject.Properties['givenName'][0]
                [string]$lastName = $ADObject.Properties['sn'][0]
                [string]$middleName = $ADObject.Properties['middleName'][0]
                
                break
            }
            'System.DirectoryServices.SearchResult'
            {
                [string]$firstName = $ADObject.Properties['givenName'][0]
                [string]$lastName = $ADObject.Properties['sn'][0]
                [string]$middleName = $ADObject.Properties['middleName'][0]
                
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
        [string]$firstName = $FirstName
        [string]$lastName = $LastName
        [string]$middleName = $MiddleName
    }
    
    # Format first name
    $firstName = switch ($FirstNameFormat)
    {
        'FullName'
        {
            $firstName
        }
        'FirstLetter'
        {
            $firstName.Substring(0, 1)
        }
    }
    
    # Format last name
    $LastName = switch ($FirstNameFormat)
    {
        'FullName'
        {
            $LastName
        }
        'FirstLetter'
        {
            $LastName.Substring(0, 1)
        }
    }
    
    # Use middle name
    [string]$middleNamePart = if ($IncludeMiddleName -and $MiddleName)
    {
        '{0}{1}' -f $Separator, $MiddleName
    }
    
    # Setup required attributes
    [string]$baseUPN = ('{0}{1}{2}{3}@{4}' -f $FirstName, $middleNamePart, $Separator, $LastName, $UPNSuffix).ToLower()
    [string]$uniqueUPN = $baseUPN
    [int]$counter = 1
    
    while (Test-UPNExist -UPN $uniqueUPN -Server $ADServer)
    {
        $uniqueUPN = '{0}{1}@{2}' -f ($baseUPN.Split('@')[0]), $counter, $UPNSuffix
        
        $counter++
    }
    
    return $uniqueUPN
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
