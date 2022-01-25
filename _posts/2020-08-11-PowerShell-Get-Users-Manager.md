---
title: "PowerShell - Get Active Directory reports"
excerpt: "In this article I'm sharing a function that will gets you all users directly or indirectly reporting to a specific manager"
categories:
  - PowerShell
  - HowTo
  - Identity Management
tags:
  - PowerShell
  - Active Directory
  - Identity Management

toc: false
header:
    teaser: "/assets/images/AD_Logo.png"
---

In my post [PowerShell - Get users reporting to manager](https://pscustomobject.github.io/powershell/howto/identity%20management/Active-Directory-Get-Report-Chain/) I have described the different approaches we can use to get a list of users reporting, directly or indirectly, to a specific manager.

In the post I have explained how we can use *LDAP Filter* and *OID* **1.2.840.113556.1.4.1941** to get results in a lightning fast manner.

As I find myself doing this rather frequently I have written a small function named **Get-ReportChain** which abstract from the various filters and provides a simple interface to get full direct/indirect reports for a manager.

```powershell
function Get-ReportChain
{
    <#
        .SYNOPSIS
            Cmdlet will get a complete report of all users reporting to a specific manager.

        .DESCRIPTION
            Cmdlet will get a complete report of all users reporting to a specific manager.

            By default the following properties are returned:

            - SamAccountName
            - UserPrincipalName
            - Mail
            - Manager
            - DirectReports

            Custom properties can be returned via the -Properties parameter

        .PARAMETER SamAccountName
            A string representing the SamAccountName of the mager for which reports should be enumerated.

        .PARAMETER UserPrincipalName
            A string representing the UserPrincipalName of the mager for which reports should be enumerated.

        .PARAMETER UserDN
            A string representing the UserPrincipalName of the mager for which reports should be enumerated.

        .PARAMETER DomainController
            A string representing the name of the domain controller that should be used to query Active Directory.

            If parameter is not specified a random domain controller will be automatically used.

        .PARAMETER Properties
            An array object representing user properties that should be returned as part of the results.

            Result array will be ordered by the Properties parameter.

            If all objects should be returned the * character can be used with the parameter.

        .EXAMPLE
            PS C:\> Get-ReportChain -UserDN 'value1'

        .OUTPUTS
            System.Array
    #>

    [CmdletBinding(DefaultParameterSetName = 'DistinguishedName',
                   SupportsPaging = $false,
                   SupportsShouldProcess = $false)]
    [OutputType([array])]
    param
    (
        [Parameter(ParameterSetName = 'SamAccountNAme',
                   Mandatory = $true)]
        [ValidateNotNullOrEmpty()]
        [Alias('UserSam', 'SAM')]
        [string]
        $SamAccountName,
        [Parameter(ParameterSetName = 'UserPrincipalName',
                   Mandatory = $true)]
        [ValidateNotNullOrEmpty()]
        [Alias('UPN', 'UserUPN')]
        [string]
        $UserPrincipalName,
        [Parameter(ParameterSetName = 'DistinguishedName',
                   Mandatory = $true)]
        [Alias('DN', 'DistinguishedName')]
        [string]
        $UserDN,
        [Parameter(ParameterSetName = 'DistinguishedName')]
        [Parameter(ParameterSetName = 'SamAccountNAme')]
        [Parameter(ParameterSetName = 'UserPrincipalName')]
        [ValidateNotNullOrEmpty()]
        [string]
        $DomainController,
        [Parameter(ParameterSetName = 'DistinguishedName')]
        [Parameter(ParameterSetName = 'SamAccountNAme')]
        [Parameter(ParameterSetName = 'UserPrincipalName')]
        [ValidateNotNullOrEmpty()]
        [string[]]
        $Properties = @(
            'SamAccountName',
            'UserPrincipalName',
            'Mail',
            'Manager',
            'DirectReports'
        )
    )

    begin
    {
        # Prepare command hash
        [hashtable]$paramGetUserDn = @{
            Properties = $Properties
        }

        # Prepare command hash
        [hashtable]$paramGetADReportChain = @{
            Properties = $Properties
        }

        switch ($PsCmdlet.ParameterSetName)
        {
            'UserPrincipalName'
            {
                # Check if UPN is valid
                if (!(Test-IsEmail -EmailAddress $UserPrincipalName))
                {
                    Write-Warning -Message "$UserPrincipalName is not a valid UPN"

                    continue
                }
                else
                {
                    # Append to command hash
                    $paramGetUserDn.Add('Filter', "UserPrincipalName -eq '$UserPrincipalName'")
                }
            }
            'DistinguishedName'
            {
                # Check if DN is in the correct format
                if (!(Test-IsValidDN -ObjectDN $UserDN))
                {
                    Write-Warning -Message "$UserDN is not a valid object DN"

                    continue
                }
                else
                {
                    # Append to command hash
                    $paramGetUserDn.Add('Identity', $UserDN)
                }
            }
            'SamAccountName'
            {
                # Append to command hash
                $paramGetUserDn.Add('Identity', $SamAccountName)
            }
        }
    }

    process
    {
        try
        {
            # Check if we should use specific DC
            if ($PSBoundParameters.ContainsKey('DomainController'))
            {
                # Append parameter
                $paramGetUserDn.Add('Server', $DomainController)
                $paramGetADReportChain.Add('Server', $DomainController)
            }

            # Get object DN
            [string]$objectDn = (Get-ADUser @paramGetUserDn).'DistinguishedName'

            # Define LDAP filter
            [string]$ldapFilter = "(manager:1.2.840.113556.1.4.1941:=$objectDn)"

            # Append paramter
            $paramGetADReportChain.Add('LDAPFilter', $ldapFilter)

            [array]$reportChain = Get-ADUser @paramGetADReportChain | Select-Object -Property $Properties
        }
        catch
        {
            Write-Warning -Message "Could not find identity $object in AD"
        }
    }

    end
    {
        return $reportChain
    }
}
```

Function accepts **SamAccountName**, **UserPrincipalName** or **DistinguishedName** of a user account and will return an array containing all direct and indirect reports of the user.

```powershell
# Get reports for user
Get-ReportChain -UserPrincipalName 'Test@automation.lab'

# Output
SamAccountName    : report1
UserPrincipalName : report1@automation.lab
Mail              : report1@automation.lab
Manager           : CN=report1,OU=Test,OU=Users,DC=automation,DC=lab
DirectReports     : {}

SamAccountName    : report2
UserPrincipalName : report2@automation.lab
Mail              : report2@automation.lab
Manager           : CN=report2,OU=Test,OU=Users,DC=automation,DC=lab
DirectReports     : {}
```

As you can see the cmdlet will return information about the user reports together with a set of *default* properties. If you need to have more properties in the output the **-Properties** switch can be used which accepts an array containing properties to be returned exactly as the standard AD cmdlet specifying the *'\*'* string all available properties will be returned.

**Note:** Cmdlet has a dependency on *two* external functions [Test-IsValidDn](https://pscustomobject.github.io/powershell/howto/identity%20management/PowerShell-Check-If-String-Is-A-DN/) and **Test-IsEmailAddress** both of which are available through my **[IT-ToolBox module](https://github.com/PsCustomObject/IT-ToolBox)**
