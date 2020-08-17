---
title: "Exchange Online - Data returned by the remote Get-FormatData command is not in the expected format"
excerpt: "When opening an Exchange Online session via PowerShell error 'Data returned by the remote Get-FormatData command is not in the expected format' is returned and no connection is opened - Let's see how to solve it"
categories:
  - PowerShell
  - Office365
  - Troubleshooting
tags:
  - PowerShell
  - Office365
  - Exchange Online
  - TroubleShooting

toc: false
header:
    teaser: "/assets/images/ExchangeLogo.png"
---

Today I started my day with my inbox filled with exceptions notifications of one of our workflow scripts taking of *Office 365* services provision. Error message was rather *generic* and not very helpful

> Data returned by the remote Get-FormatData command is not in the expected format.

I was able to easily pinpoint this to he function opening a connection to Exchange online, I will publish this soon, but the error message was not really helpful to know exactly what's going on.

It turns out this is a *[known](https://answers.microsoft.com/en-us/msoffice/forum/all/cannot-connect-to-exchange-online-via-powershell/25ca1cc2-e23a-470e-9c73-e6c56c4fbb46?page=1)* and unfortunately **semi random** issue *Microsoft* is aware of and, at the time of writing, there is no ETA on a final solution.

The official workaround from Microsoft is connecting to *Exchange Online* via the following code:

```powershell
# Connect to Exchange online with modified endpoing URL
Connect-ExchangeOnline -ConnectionUri "https://outlook.office365.com/powershell-liveid?SerializationLevel=Full"
```

The above, as per Microsoft statement, has a *very low* performance impact in the process of selecting the optimal endpoint for interaction with the service and I can confirm it works reliably in solving the issue.

Microsoft also made available a [feedback form](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR2iCyV8tWU1Kh494jBzFAxdUN1pLTlVNTkRWWFc4UTBIWjFKUlpEOTdTMy4u) you can use to report the issue which should help them track down the issue and hopefully solve it.

Hope this will help you as well!
