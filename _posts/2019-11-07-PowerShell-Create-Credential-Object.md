---
title: "PowerShell Create Credential Object"
toc: false
categories:
  - PowerShell
  - HowTo
tags:
  - PowerShell
  - PowerShell Basics
  - PowerShell Core
  - PowerShell create credential
  - new-object system.management.automation.pscredential
---

There are many cmdlets that take a **[PSCredential](https://docs.microsoft.com/en-us/dotnet/api/system.management.automation.pscredential?view=pscore-6.2.0)** object to run under the context of that specific user.

When running a script interactively this can easily be solved using similar syntax (using *SharePoint Online* cmdlets in the example)

```powershell

# Open connection to SharePoint Online admin portal
Connect-SPOService -Url $adminUrl -Credential (Get-Credential)
```

Unfortunately while this method is very handy it won't work when implementing a script as part of an unattended solution so we have to *instantiate* credential object.

A *credential* object is made up of two parts a **string** representing the username and a **[SecureString](https://docs.microsoft.com/en-us/dotnet/api/system.security.securestring?view=netframework-4.8)** representing the user password.

To create the *SecureString* object the following syntax is used

```powershell
# Define clear text string for username and password
[string]$userName = 'MyUserName'
[string]$userPassword = 'MySuperSecurePassword'

# Convert to SecureString
[securestring]$secStringPassword = ConvertTo-SecureString $userPassword -AsPlainText -Force
```

Once the SecureString object has been created the *PSCredential* object can be created with the following syntax

```powershell
[pscredential]$credObject = New-Object System.Management.Automation.PSCredential ($userName, $secStringPassword)
```

The resultant *pscredential* object can be passed to any cmdlet requiring/using the **-PSCredential** parameter.

**Note:** This is just an example so I use a plain text password in my script this is never a good idea or practice in a production environment. If storing credentials in a script is a requirement follow instructions in [this article](https://pscustomobject.github.io/powershell/howto/Store-Credentials-in-PowerShell-Script/) where I describe alternatives methods which are way more secure than a plain text string. Functions mentioned in the article can be found in my [IT-Toolbox Module](https://github.com/PsCustomObject/IT-ToolBox)
{: .notice--primary}
