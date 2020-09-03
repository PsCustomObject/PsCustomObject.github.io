---
title: "PowerShell - Get all members of a group recursively"
excerpt: "In this article we are going to explore how we can use PowerShell to get all members of a group even if nested"
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

In my other article [Get-ADGroupMember : The operation returned because the timeout limit was exceeded](https://pscustomobject.github.io/powershell/powershell%20tips/Get-AdGroupMember-Timeout/) a user commented

> Nice n fast, as long as you don't want recursive members? How to make this include recursive members?

This is a rather common requirement in every environment as getting members of a group is rather straightforward but *discovering* indirect membership is not that obvious. Funnily enough answer lies in something I posted couple of days ago when explaining how to get all users reporting to a specific manager OID **1.2.840.113556.1.4.1941**.

Let's see this in action

```powershell
# Define LDAP Filter to use
[string]$ldapFilter = '(memberOf:1.2.840.113556.1.4.1941:=CN=TestGroup,OU=Groups,DC=automation,DC=lab)'

# Get all members of the group recursively
Get-ADUser -LDAPFilter $filter -Properties mail

# Output
DistinguishedName : CN=user1,OU=Users,OU=Test,DC=automation,DC=lab
mail              : user1@automation.lab
Name              : user1
ObjectClass       : user
ObjectGUID        : 75c199bf-4a20-4e2a-82b8-80796f7f39cf

DistinguishedName : CN=user2,OU=Users,OU=Test,DC=automation,DC=lab
Name              : user2
ObjectClass       : user
ObjectGUID        : e3153344-be4d-409f-afba-4569b36c9922

# <snip>
```

If you also want to return groups members of the specified group simply use the *Get-AdObject* cmdlet

```powershell
# Define LDAP Filter to use
[string]$ldapFilter = '(memberOf:1.2.840.113556.1.4.1941:=CN=TestGroup,OU=Groups,DC=automation,DC=lab)'

# Get all members of the group recursively
Get-ADObject -LDAPFilter $filter -Properties mail

# Output
DistinguishedName : CN=user1,OU=Users,OU=Test,DC=automation,DC=lab
mail              : user1@automation.lab
Name              : user1
ObjectClass       : user
ObjectGUID        : 75c199bf-4a20-4e2a-82b8-80796f7f39cf

DistinguishedName : CN=user2,OU=Users,OU=Test,DC=automation,DC=lab
Name              : user2
ObjectClass       : user
ObjectGUID        : e3153344-be4d-409f-afba-4569b36c9922

DistinguishedName : CN=TestGroup,OU=Distribution Groups,DC=automation,DC=lab
mail              : GRoup1@automation.lab
Name              : Group1
ObjectClass       : group
ObjectGUID        : e715c1c2-d596-4fc4-975c-7e30ed9c2c8b
```

Again this approach is lightning fast and can return thousand of results in couple of seconds making it a suitable candidate for larger deployments.

As I did with [Get-ReportChain](https://pscustomobject.github.io/powershell/howto/identity%20management/PowerShell-Get-Users-Manager/) I will wrap this up as a function and make it available in my [IT-ToolBox module](https://github.com/PsCustomObject/IT-ToolBox)
