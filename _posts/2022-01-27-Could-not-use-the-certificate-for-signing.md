---
title: "Exchange Online Management Module - Could not use the certificate for signing"
excerpt: "When trying to establish a connection to Exchange Online Could not use the certificate for signing error message is displayed."
categories:
  - PowerShell
  - Exchange
  - Office 365

tags:
  - PowerShell
  - Office365
  - Exchange

toc: true
header:
  teaser: "/assets/images/PowerShell_Logo.png"
---

## Exchange Online Certificate Based authentication

As I have written in my [previous post about TokenExpiry error message Microsoft is retiring ability to connect to Exchange Online via basic authentication](https://pscustomobject.github.io/powershell/exchange/office%20365/Cannot-bind-argument-to-parameter-Token-Expiry/).

You can read my article on how to implement _Certificate Based authentication_ for Exchange Online [here](https://pscustomobject.github.io/powershell/office365/exchange/Exchange-Online-Certificate-Based-Authentication/).

## Could not use the certificate for signing error message

Today while I was updating code for one of our automations I created a request for a new certificate to use for authentication purposes. 

Once I deployed code to our test environment automation was failing the connection to Exchange Online with the following error

```powershell
[System.Management.Automation.RuntimeException] One or more errors occurred.
[Microsoft.Identity.Client.MsalClientException] Could not use the certificate for signing. See inner exception for details. Possible cause: this may be a known issue with apps build against .NET Desktop 4.6 or lower. Either target a higher version of .NET desktop - 4.6.1 and above, or use a different certificate type (non-CNG) or sign your own assertion as described at aka.ms/msal-net-signed-assertion.
[System.Security.Cryptography.CryptographicException] Invalid provider type specified.
````

Funnily enough the same certificate and cmdlets were working fine with PowerShell 7. 

After quite some troubleshooting I've found out the problem was caused by the certificate's private key using [*Cryptography Next Generate (CNG)*](https://docs.microsoft.com/en-us/mem/configmgr/core/plan-design/network/cng-certificates-overview) template rather than RSA. 

Not having direct access to the CA releasing the certificate I could not change this so I had to resort on either running the automation in PowerShell 7 or update the certificate itself. 

Luckily this is easily done via OpenSSL. Let's see how.

## Convert Certificate private key from CNG to RSA

If you have installed Git, cygwin or Windows Subsystem for Linux you just need to fire a bash prompt and use the following commands:

```bash
# Extract the public key from the cert 
OpenSSL pkcs12 -in "CNGCertificate.pfx" -nokeys -out "temp.cer"

# Extract the private key
OpenSSL pkcs12 -in "CNGCertificate.pfx" -nocerts -out "temp.pem"

# Convert key to RSA
OpenSSL rsa -inform PEM -in "temp.pem" -out "temp.rsa"

# Finally create a new pfx file
OpenSSL pkcs12 -export -in "temp.cer" -inkey "temp.rsa" -out "RSACertificate.pfx"
````

**Note:** In the above commands I am not using a password for the certificate as everything is local to my machine but a password is definitely *required* when exporting a certificate together with the private key.
{: .notice--warning}

Once the new pfx file has been created all *temporary* certificates can be safely removed form the system and connection to Exchange Online will go through just fine. 

Again if you can use PowerShell 7 you will not face this issue but in case you're stuck with version 5.1 and facing this error message hopefully this post can save you some headaches. 

Full credit for the solution goes to this [StackOverFlow thread](https://stackoverflow.com/questions/22581811/invalid-provider-type-specified-cryptographicexception-when-trying-to-load-pri/34103154#34103154)