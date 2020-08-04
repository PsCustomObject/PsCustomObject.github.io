---
title: "Upgrade PowerShell 7 version"
excerpt: "In this post we will explore how to easily and effectively upgrade PowerShell 7 when a new version is released"
categories:
  - PowerShell
tags:
  - PowerShell
  - PowerShell 7
  - HowTo

toc: false
---

When Microsoft releases a new PowerShell 7 version the following message is displayed in terminal/console:

![PowerShell upgrade available](/assets/images/PowerShell-Upgrade.png)

To upgrade to the latest version you can either go to the [PowerShell Releases Page](https://github.com/PowerShell/PowerShell/releases) or use the following command to upgrade directly from the console:

```powershell
Invoke-Expression "& { $(Invoke-Restmethod https://aka.ms/Install-PowerShell.ps1) } -UseMSI"
```

This will start the download of latest PowerShell 7 version and start the installer directly from the current console:

![PowerShell Installer Page](/assets/images/PowerShell-7-Installer.png)

The use of the *-UseMSI* parameter will cause installation to immediately start, when you will be asked chose to close application locking required files this will close the current PowerShell session but installation will progress and once it is over latest PowerShell version will be installed.
