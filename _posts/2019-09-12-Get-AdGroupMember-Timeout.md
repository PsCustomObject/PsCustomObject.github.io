---
title: "Get-ADGroupMember : The operation returned because the timeout limit was exceeded"
excerpt: "When using Get-AdGroupMember cmdlet with a group with a large number of members the The operation returned because the timeout limit was exceeded error could be shown. Let's see how to solve this."
last_modified_at: 2020-08-14T12:02:02-01:00
categories:
  - PowerShell
  - PowerShell Tips
tags:
  - PowerShell
  - PowerShell from the field
  - PowerShell Active Directory
  - 

toc: true
---

## Get-ADGroupMember - Timeout limit was exceeded

**Note: [I have published an updated article providing a more performant approach to the issue with added benefit of returning results in a recurisive way](https://pscustomobject.github.io/powershell/howto/identity%20management/Get-User-Group-Membership/)**.
{: .notice--danger}

The *Get-ADGroupMember* cmdlet returns  the members of an Active Directory group. Members can be users, other groups or computers.

It is an handy tool to quickly check if a user has been already been made member of a specific group like for example:

```powershell
# Get all members of the Dl-GroupSame
Get-ADGroupMember -Identity DL-GroupSample

distinguishedName : CN=Domain Admins,CN=Users,DC=Lab,DC=com
name              : Domain Admins
objectClass       : group
objectGUID        : 5ccc6037-c2c9-42be-8e92-c8f98afd0011
SamAccountName    : Domain Admins
SID               : S-1-5-21-41432690-3719764436-1984117282-512

distinguishedName : CN=Enterprise Admins,CN=Users,DC=Lab,DC=com
name              : Enterprise Admins
objectClass       : group
objectGUID        : 0215b0a5-aea1-40da-b598-720efe930ddf
SamAccountName    : Enterprise Admins
SID               : S-1-5-21-41432690-3719764436-1984117282-302

distinguishedName : CN=LabAdmin,CN=Users,DC=Lab,DC=com
name              : LabAdmin
objectClass       : user
objectGUID        : ab7c269d-aec5-4fcc-aebe-6cd1a2e6cd53
SamAccountName    : LabAdmin
SID               : S-1-5-21-41432690-3719764436-1984117282-1015
<snip>
```

Issue is when working with large groups with lot of members Domain Controller will cut off the operation the following error will be displayed

```powershell
Get-ADGroupMember : The operation returned because the timeout limit was exceeded.
At line:1 char:1
+ Get-ADGroupMember DL-LargeGroup
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OperationTimeout: (DL-LargeGroup:ADGroup) [Get-ADGroupMember], TimeoutException
    + FullyQualifiedErrorId : ActiveDirectoryCmdlet:System.TimeoutException,Microsoft.ActiveDirectory.Management.Comma
   nds.GetADGroupMember
```

There are multiple solutions to this which is a by design behavior and not some obscure error message.

## Solution 1 - Modify Maximum Number of Return Objects

As I said the above error message is a by design feature, more specifically one imposed by Active Directory Web Service (ADWS) used by Active Directory module to query AD.

Limit is defined in the **c:\Windows\ADWS\Microsoft.ActiveDirectory.WebServices.exe.config** configuration file stored on each domain controller.

Default limit for returned objects is set to **5000** but this can be changed under the **<appSettings>** section and adding the following line

```xml
<add key=”MaxGroupOrMemberEntries” value=”25000”/>
```

The above will raise the limit of returned objects to 25000.

**Note on updating WebServices.exe.config file:** Changing the setting can negatively affect performances of the domain controller. Also note file need to be updated on each domain controller separately.
{: .notice--primary}

## Solution 2 - Get-ADGroup Cmdlet

Modifying inner Active Directory configuration files is not always a good idea and sometimes not possible at all, for example where I work such a change is not permitted, for this reason I usually workaround the problem not using the *Get-ADGroupMember* cmdlet at all.

The following command will return DNs of all users member of any given group:

```powershell
Get-ADGroup -Identity 'DL-LargeGroup' -Properties Member | Select-Object -Property 'Member' -ExpandProperty 'Member'
```

Output of the above command can be assigned to a variable and easily manipulated.

This approach has two advantages.

**1.** Modification to Active Directory built-in limits is not required

**2.** Command runs *extremely* fast, for example on a group with over 14k members it took 3 seconds in my environment.
