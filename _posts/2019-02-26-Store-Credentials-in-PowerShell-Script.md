---
title: "Store Credentials in PowerShell Script"
excerpt: "Handle PowerShell credentials without hard-coded passwords, with a Windows DPAPI example and clear limits for unattended automation."
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
last_modified_at: 2026-10-23
---

## Choose the authentication model first

For unattended automation, prefer a workload identity, managed identity or certificate-based application authentication when the target service supports it. When a password is unavoidable, retrieve it from a secret store whose access controls and unattended authentication fit the execution account.

An encrypted password is still a credential. Code running as an identity authorised to decrypt it can recover and use it. Encryption at rest does not replace file permissions, least privilege, rotation or control of the automation host.

## Get-Credential

For an interactive script, prompt rather than placing the password in source code:

```powershell
$credential = Get-Credential
# Pass $credential to a command that accepts -Credential.
```

A `PSCredential` packages a username and a `SecureString`; it does not determine how the destination service authenticates. Confirm that the target command supports this authentication method.

## Plain Text Credentials in Script

Do not put passwords in scripts, command-line arguments or source control. Converting a hard-coded string with `ConvertTo-SecureString -AsPlainText` cannot protect the original literal or remove it from repository history.

## Store Encrypted password in an external file

The following example is **Windows-only** and uses Windows Data Protection API (DPAPI) without an explicit encryption key. Create the file as the same Windows account, on the same machine, that will read it. Use an existing directory with restrictive permissions; the example path is a placeholder, not a directory created by these commands.

```powershell
# One-time interactive setup on the automation host, as its execution account.
$credentialPath = 'C:\Automation\Secrets\service-password.txt'
$credential = Get-Credential
$credential.Password |
    ConvertFrom-SecureString |
    Set-Content -LiteralPath $credentialPath
```

Read the file under that same account and machine context:

```powershell
$credentialPath = 'C:\Automation\Secrets\service-password.txt'
# Replace with the username supplied during setup.
$userName = 'CONTOSO\svc-automation'
$securePassword = (Get-Content -LiteralPath $credentialPath -Raw).Trim() |
    ConvertTo-SecureString
$credential = [System.Management.Automation.PSCredential]::new(
    $userName, $securePassword
)
```

DPAPI binds this example to a Windows user and machine context. It is not a portable deployment format. Anyone able to run code as that account may be able to decrypt the password; physical access is not required. Restrict the file and execution account, and recreate the encrypted file when changing the deployment context or rotating the password.

The protection described here must not be assumed on Linux or macOS. Microsoft documents that `SecureString` contents are not encrypted on non-Windows systems. See [ConvertTo-SecureString](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.security/convertto-securestring) and [ConvertFrom-SecureString](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.security/convertfrom-securestring) for the platform and key-handling details.

## Use an Encrypted String in Script

The original article recommended custom `New-StringEncryption` and `New-StringDecryption` helpers from a now-retired TechNet Gallery page. That recommendation and its sample ciphertext have been removed: the original post did not establish the implementation’s security boundary, so its machine-only decryption claim should not be relied on.

Base64 is an encoding, not encryption. If an AES key is used with the built-in secure-string cmdlets, the key must be protected separately from the ciphertext. Distributing both together defeats the intended separation.

## Closing Notes

For a secret store integration, review authentication, auditability, rotation and the capabilities of the vault provider. PowerShell [SecretManagement](https://learn.microsoft.com/en-us/powershell/utility-modules/secretmanagement/overview) provides a vault abstraction; Microsoft now describes those modules as feature complete, maintained for security and critical fixes. Do not assume any particular vault configuration will work unattended.

For Exchange Online automation, see the [certificate authentication guide]({% post_url 2020-09-21-Exchange-Online-Certificate-Based-Authentication %}) and its current Microsoft references.
