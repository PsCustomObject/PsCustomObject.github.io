---
title: "PowerShell - Get users reporting to manager"
excerpt: "Subtitle - How do I get the full report chain for a specific manager from Active Directory"
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

    last_modified_at: 2020-08-09T18:54:02-01:00
---

## The problem

One request we receive rather frequently is to create groups containing all users in a specific reporting chain. In other words groups containing all users reporting to *Manager 1* and all their direct reports recursively.

While this kind of reports is usually available through any decent HR software, like *Workday*, getting this kind of information out of Active Directory poses some unique challenges.

While we can get direct reports of a specific user via the *DirectReports* AD property like this

```powershell

# Get user direct reports
$paramGetADUser = @{
    AuthType   = 'Test-User'
    Properties = 'DirectReports'
}

Get-ADUser @paramGetADUser | Select-Object 'DirectReports' -ExpandProperty 'DirectReports'

# Output
CN=Test\, User01,OU=Admin,OU=Users,OU=Admin,DC=automatuon,DC=lab
CN=Test\, User02,OU=Admin,OU=Users,OU=Admin,DC=automatuon,DC=lab
CN=Test\, User03,OU=Admin,OU=Users,OU=Admin,DC=automatuon,DC=lab
```

But this only solves half of the problem as if any of the above users has direct reports it will not be shown in the output. Once solution would be to get all direct reports, cycle through them and recursively get direct reports.

While a valid approach it is inefficient and not so elegant at least for my personal taste.

## Get management report chain - Approach 1

If like me you are lucky enough to have access to *OneIdentity ActiveRoles* (once *Quest*) cmdlets the *Get-QADUser* has a *-Manager* parameter that effectively works like the above example.

What is so special about the cmdlet is that we can turn it in a [recursive function](https://en.wikipedia.org/wiki/Recursion_(computer_science)) to get the full reporting chain (if you're wondering yes I've already done this)

```powershell
function Get-ReportChain
{
    param
    (
        [Parameter(Mandatory = $true)]
        [ValidateNotNullOrEmpty()]
        [string]
        $Identity
    )

    # Find all direct reports for identity
    Get-QADUser -Manager $Identity | ForEach-Object {

        # Output direct report
        $_

        # Call function on each report
        Get-ReportChain -Identity $_
    }
}

Get-ReportChain -Identity $managerUser
```

As you can see function is really simple, it will accept a user identity as input and will use recursion to find all **direct** and **indirect** reports of the user.

This approach served me very well and I've been using it in my automation solutions for quite some time. Only drawback is, as I already mentioned, for this to work is you will need access to Active Roles cmdlets.

## Get management report chain - Approach 2 (The native way)

A little know fact is that *[Active Directory schema](https://docs.microsoft.com/en-us/windows/win32/adschema/active-directory-schema)* has an *[OID](https://en.wikipedia.org/wiki/Object_identifier)* named **LDAP_MATCHING_RULE_IN_CHAIN** which does exactly what you're thinking of giving us the full report chain. Let's see it in action.

```powershell
# Define user DN
[string]$ceoUser = 'CN=ceo\, user,OU=Users,DC=automation,DC=lab'


# Define LDAP filter
[string]$ldapFilter = '(manager:1.2.840.113556.1.4.1941:=$ceoUser)'

# Same as above just with full DN declaration
#[string]$ldapFilter = '(manager:1.2.840.113556.1.4.1941:=CN=ceo\, user,OU=Users,DC=automation,DC=lab)'

# Get report chain
[array]$allUsers = Get-AdUser -LDAPFilter $ldapFilter
```

As you can see we can easily query the OID via standard LDAP filter specifying the user DN using *Get-AdUser* cmdlet which is included with the *Active Directory* module. The best part is yet to come, this is **blazing fast**! In an environment with over 15.000 objects getting the full report chain from the CEO down to the most humble IT peon, me, it took a whopping **20 seconds** to run the command all of this from a VPN connection.

This method works great, but what if you don't have or cannot install the AD module for some reason? Don't despair, there is a third way which we're going to explore in a second.

## Get management report chain - Approach 3 (The poor man's approach)

If for some reason the active directory module is not available on the machine where you need to run your code you can still leverage .Net to achieve the same result

```powershell
# Define user DN
[string]$ceoUser = 'CN=ceo\, user,OU=Users,DC=automation,DC=lab'


# Define LDAP filter
[adsisearcher]$searcherObject = "(manager:1.2.840.113556.1.4.1941:=$ceoUser)"

# Set paging size to return all results
$searcherObject.PageSize = 200

# Get report chain
[array]$allUsers = $searcherObject.FindAll()
```

The above is equivalent to the *Get-AdUser* method as, under the hood, PowerShell is using a [DirectorySearcher](https://docs.microsoft.com/en-us/dotnet/api/system.directoryservices.directorysearcher?view=netcore-3.1) to retrieve results for us.

There is much more to this accelerator but I'll leave exploration to the reader's curiosity. As a reference here's how the object we've just instantiated looks like

```powershell
CacheResults             : True
ClientTimeout            : -00:00:01
PropertyNamesOnly        : False
Filter                   : (manager:1.2.840.113556.1.4.1941:=CN=ceo\, user,OU=Users,DC=automation,DC=lab)
PageSize                 : 200 # This is very important as if not set only fist 1000 results will be returned
PropertiesToLoad         : {}
ReferralChasing          : External
SearchScope              : Subtree
ServerPageTimeLimit      : -00:00:01
ServerTimeLimit          : -00:00:01
SizeLimit                : 0
SearchRoot               : System.DirectoryServices.DirectoryEntry
Sort                     : System.DirectoryServices.SortOption
Asynchronous             : False
Tombstone                : False
AttributeScopeQuery      :
DerefAlias               : Never
SecurityMasks            : None
ExtendedDN               : None
DirectorySynchronization :
VirtualListView          :
Site                     :
Container                :
```

## Closing thoughts - Which method to use

When developing some automation solution I try to use *native* tools as much as possible to make code portable and implementation easier. This does not imply that I consider using modules or external binaries a bad practice, far from it, but we cannot give for granted they will be available on all servers where our code will run and trust me in some environments something as simple as installing a module can be a real pain.

As a personal suggestion I would say if the *AD Module* is available on the system go for [Method 2](https://pscustomobject.github.io/powershell/howto/identity%20management/Active-Directory-Get-Report-Chain/#get-management-report-chain---appraoch-2-the-native-way) which will give you the best performances with the added flexibility of PowerShell native cmdlets.

**Note:** I have created a function called **Get-ReportChain**, about which I will write soon, that you can already find contained in my **[IT-ToolBox Module](https://github.com/PsCustomObject/IT-ToolBox)**
{: .notice--primary}
