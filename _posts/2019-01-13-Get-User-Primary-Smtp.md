---
title: "Get User Primary SMTP Address without Exchange module"
excerpt: "How to get user primary SMTP Address when you don't have the Exchange module installed on your machine"

categories:
  - PowerShell Tips
tags:
  - PowerShell
  - Exchange
  - Tips

toc: false
---

Have you ever found yourself in need to find user primary SMTP without having access to Exchange cmdlets? Usually checking the *email* built-in attribute is enough but there are situations where field is either empty or not properly aligned with user's primary SMTP address.

Here's a way to do it:

```powershell
$testUser = Get-AdUser -Identity 'Test.User' -Properties 'ProxyAddresses'
[string]$primaryAddress = $testUser.'ProxyAddresses' -clike 'SMTP:*'
```

A user can only have a primary address denoted by the **SMTP** prefix. In the above example we are performing a *case sensitive* like operation on the *ProxyAddresses* attribute which will return only the primary address.

Of course the same approach can be used when dealing with multiple users

```powershell
[array]$matchingUsers = Get-AdUser -Filter * -Properties 'ProxyAddresses'

foreach ($user in $matchingUsers)
{
  [string]$primaryAddress = $testUser.'ProxyAddresses' -clike 'SMTP:*'

  # Take any required action
}
```
