---
title: "Convert a secure string to plain text"
toc: false
categories:
  - PowerShell
  - Functions
tags:
  - PowerShell
  - PowerShell Functions
---

In my [previous post](2019-11-07-PowerShell-Create-Credential-Object.md) I have described how we can create a *credential object* part of the process involved creating a *secure string* object like this

```powershell
[string]$userPassword = 'MySuperSecurePassword'

# Convert to SecureString
[securestring]$secStringPassword = ConvertTo-SecureString $userPassword -AsPlainText -Force
```

But what if we need to reverse the process? PowerShell has a handy cmdlet aptly named *ConvertFrom-SecureString* with the following syntax

```powershell
ConvertFrom-SecureString -SecureString $secStringPassword
```

The above will produce **an encrypted** standard string, I have described this process in my post [about storing credentials in a script](https://pscustomobject.github.io/powershell/howto/Store-Credentials-in-PowerShell-Script/#store-encrypted-password-in-an-external-file), which is not what we're looking for.

To get the *unencrypted* string we will to leverage built-in .Net methods which, while not as straightforward as a cmdlet, will get the job done here's the syntax

```powershell
[System.Runtime.InteropServices.Marshal]::PtrToStringAuto([System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($secStringPassword))

# Output
MySuperSecurePassword
```

The above will leverage the *[Marshall Class](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.marshal?view=netframework-4.8)* to *convert* a secure string to its *standard and unencrypted* equivalent.

While not very common there are situations, for example when calling an external exe, where *secure string* are not supported and this is an handy way to use them without storing them in clear text in your code.

As usual Microsoft has pretty extensive documentation about the method that you can find [here](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.marshal.securestringtobstr?view=netframework-4.8).
