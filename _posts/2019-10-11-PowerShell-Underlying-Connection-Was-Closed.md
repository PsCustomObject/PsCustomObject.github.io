---
title: "The underlying connection was closed - An unexpected error occurred"
excerpt: "When using PowerShell Invoke-RestMethod you get the Invoke-RestMethod : The underlying connection was closed: An unexpected error occurred Error message"
last_modified_at: 2019-10-27T21:35:02-05:00
categories:
  - PowerShell
  - HowTo
tags:
  - PowerShell
  - TLS
  - Invoke-RestMethod
  - PowerShell TLS 1.0
  - PowerShell TLS 1.2
---

## Problem statement

When issuing the **Invoke-RestMethod** to interact with a web API you get the following error message:

```powershell
Invoke-RestMethod : The underlying connection was closed: An unexpected error occurred on a send
```

## Root cause and solution

Issue is PowerShell, by default, uses TLS version 1.0 which is disabled on any modern platform due to security risks associated with the use of it.

To solve this simply issue the following command:

```powershell
# Enable TLS 1.2 as Security Protocol
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
```

If you need to support multiple versions you can do so like this

```powershell
# Enable TLS 1.2 and TLS 1.1 as fallback
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12,
[Net.SecurityProtocolType]::Tls11
```

Keep in mind the above will be effective **only** for the active PowerShell session where command is executed and it is not a system wide configuration.

What I usually do is setting the TLS version as part of my scripts so it automatically configured when script is executed.

I also put this in my PowerShell profile so don't have to manually set this when I open a new PowerShell session.
