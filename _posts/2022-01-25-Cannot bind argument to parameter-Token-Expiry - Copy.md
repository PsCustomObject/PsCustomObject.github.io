---
title: "Cannot bind argument to parameter TokenExpiryTime because it is null - Error Message"
excerpt: "When opening a PowerShell session to both on-prem and Exchange online in the same window cannot bing argument to parameter TokenExpiryTime because is null error message could be displayed. Let's explore how to solve this."
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

Microsoft is, _finally_, disabling **basic authentication** (read username and password) in Exchange Online in favor of **Certificate Based authentication**.

Once this change is fully implemented, around mid February at least for some tenants, connecting via username and passwords to Exchange Online will not be possible anymore.

You can read my article on how to implement _Certificate Based authentication_ for Exchange Online [here](https://pscustomobject.github.io/powershell/office365/exchange/Exchange-Online-Certificate-Based-Authentication/).

As a result of this change I started updating one of our automations, responsible for the whole life-cycle of our mailboxes, to ditch old credential objects in favor of the more secure Certificate Authentication.

This is when I encountered the _‌cannot bind argument to parameter 'token expiry time' because it is null._ error message.

## Multiple PowerShell Exchange Sessions

When operating an hybrid environment it is pretty common to open, in the same window/session, a PowerShell connection to both Exchange on-prem and Exchange online.

This is required as part of the configuration, usually creation of the mailbox, takes place in on-prem for example via the _New-RemoteMailbox_ cmdlet while other parts of the configuration are performed directly online.

This morning while debugging my workflow I’ve noticed that while trying to retrieve mailbox information from the on-prem server an exception was being thrown

```powershell
# Cmdlet I was running
Get-RemoteMailbox -Identity $userUpn

# Part of the exception message
Cannot bind argument to parameter 'token expiry time' because it is null.
```

It took me quite a bit to figure this out as no exception during the connection phase was being thrown by either module.

When I was about to give up and open a ticket with Microsoft, which is usually as helpful as freezer in the North Pole, I discovered by chance that opening connection to Exchange Online **first** and **only afterwards** to the Exchange on-prem was working as intended, allowing me to interact with both the Online service and my local Exchange.

**Note:** I have experienced/tested this with version 2.0.4 and 2.0.5 of Exchange Online PowerShell module but other versions could be affected as well.
{: .notice--warning}

I did not dig deep into the root cause of the issue but plan to do this tomorrow and already sent my feedback to exocmdletpreview {at} service {fullstop} microsoft {fullstop} com but I doubt I will hear anything from that channel. I plan to open a ticket anyhow to at least have an official statement on this.

As soon as I have any news I will update the post until then I hope you can find the information useful.
