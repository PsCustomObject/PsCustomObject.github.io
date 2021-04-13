---
title: "Store Credentials in PowerShell Script"
excerpt: "How to securely store credentials in a PowerShell script writing them in plain sight"
categories:

  - PowerShell
  - HowTo
tags:
  - PowerShell
  - PowerShell Core
  - Passwords
  - new-object system.management.automation.pscredential
  - PowerShell create credential
  - PowerShell store credentials in file
---

## Get-Credential

One  common tasks, when dealing with different servers and services, is the requirement of storing username and passwords in a script to carry on the designed task.  Despite being something so common I still do see this being implemented the wrong way partially caused by the lack of a standard method of or built-in function.

When creating an interactive script we can easily use the *Get-Credential* cmdlet which will ask us for a username and a password creating the required object in the background

```powershell
# Output from PowerShell core on MacOs
Get-Credential

PowerShell credential request
Enter your credentials.
User: admin
Password for user admin: **********


UserName                     Password
--------                     --------
admin    System.Security.SecureString
```

As I said this works very well when writing script that will be run interactively and a human being is available to input required fields.

## Plain Text Credentials in Script

First of all a disclaimer **never do this** unless you are testing something in  a lab or protected environment. I cannot count the times I have seen something like this in production

```powershell
# Define Credentials
[string]$userName = 'admin'
[string]$userPassword = 'mySuperSecurePassword'

# Crete credential Object
[SecureString]$secureString = $userPassword | ConvertTo-SecureString -AsPlainText -Force 
[PSCredential]$credentialObejct = New-Object System.Management.Automation.PSCredential -ArgumentList $userName, $secureString
```

It is pretty obvious why this should never be used in production, anybody who has access to our code would be able to easily read username password which are conveniently written in the code.

## Store Encrypted password in an external file

As seen in the previous example when creating a *PSCredential* object we need to specify a *SecureString* object as argument created with the *ConvertTo-SecureString* cmdlet.

Inspecting content of the *$secureString* variable after converting it with the *ConvertFrom-SecureString* we will see PowerShell create long string of random characters similar to this *1204890d04c9ddf0115d1…..d3d491bb6d740864117a090d11*, shortened for convenience, which is what internally PowerShell uses.

Taking this a step further we can save the *secure string*  to a file that is then sourced in our script, here's an example:

```powershell
# Define clear text password
[string]$userPassword = 'mySuperSecurePassword'

# Crete credential Object
[SecureString]$secureString = $userPassword | ConvertTo-SecureString -AsPlainText -Force 

# Get content of the string
[string]$stringObject = ConvertFrom-SecureString

# Save Content to file
$stringObject | Set-Content -Path 'C:\SomeDir\Secure.txt'
```

The above will create a secure string object from our password, extract the encrypted string and save it to a file which can now be used in our scripts. Previous example could be rewritten like this

```powershell
# Define Credentials
[string]$userName = 'admin'
[string]$passwordText = Get-Content 'C:\SomeDir\Secure.txt'

# Convert to secure string
[SecureString]$securePwd = $pwdTxt | ConvertTo-SecureString 

# Create credential object
[PSCredential]$credObject = New-Object System.Management.Automation.PSCredential -ArgumentList $userName, $securePwd
```

There are couple of points to keep in mind when using this approach

- A malicious user can still run the script and *authenticate* as the user *admin* if he gets physical access to the machine
- If we need to run the same script on multiple machines we will need to create multiple *Secure.txt* files one for each machine on which the script will run

Last point is specifically important as it can become impractical if the same script needs to run on multiple machines. Additionally, as credentials are encrypted using DAPI, **only the user who created the encrypted credentials** on the original machines will be able to access/decrypt them.

**Note:** If you are curious about the last remark I suggest reading more about [Data Protection API](https://en.wikipedia.org/wiki/Data_Protection_API) or for something more technical you can refer to [this article](https://docs.microsoft.com/en-us/dotnet/standard/security/how-to-use-data-protection)

## Use an Encrypted String in Script

**Full Disclosure:** Before I describe this method let me clarify I am not the original author of the code used to do the encryption/decryption part, I simply downloaded it, refined a bit and adapted to my needs. Original code can be found [here](https://gallery.technet.microsoft.com/scriptcenter/PowerShell-Script-410ef9df#content).

This is what I usually use in my automation scripts to avoid having too many files lying around and it implies storing the password directly in the script code, yes that's not a typo, as it is *encrypted*.

First of all encrypt the string with the following command after importing the **New-StringEncryption** function

```powershell
# Create encrypted string
New-StringEncryption -StringToEncrypt 'MySecurePassword123'

# Output string
kvKHaoJytKOhWVSLRwpaAb4jBDz0i/s4yUdlhFKpNi0=
```

This will generate a an encrypted string that can only be decrypted on the same machine that performed the original encryption. Once we have the encrypted string we can input it in script code like this:

```powershell
# Decrypt string and convert it to a Secure.String object
ConvertTo-SecureString -String (New-StringDecryption -EncryptedString 'kvKHaoJytKOhWVSLRwpaAb4jBDz0i/s4yUdlhFKpNi0=') -AsPlainText -Force
```

The **New-StringDecryption** is a companion function to the encrypting one that does exactly what the name implies, takes a *Base64* encoded string and will output the clear text version of it.

Putting all together we would have something like this in our script

```powershell
# Define Credentials
[string]$userName = 'admin'
[SecureString]$securePwd = ConvertTo-SecureString -String (New-StringDecryption -EncryptedString 'kvKHaoJytKOhWVSLRwpaAb4jBDz0i/s4yUdlhFKpNi0=') -AsPlainText -Force

# Create the credential object
[pscredential]$credentialObject = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $userName, $securePwd

# Code here
```

## Closing Notes

In the article we have seen different approaches to store passwords in PowerShell scripts while not saving them in clear text. It has to be noted none of the illustrated methods is bullet proof but, once again, it is better than storing sensitive information in clear text.

Among the methods illustrated I find last one easier to implement as encrypted passwords can be centrally stored on a file and sourced from multiple scripts using the correct one for the application or service that script needs to connect to.

All the functions used in the post are available through my **[IT-ToolBox](https://github.com/PsCustomObject/IT-ToolBox)** module.
