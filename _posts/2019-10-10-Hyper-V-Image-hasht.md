---
title: "The image hash and certificate are not allowed"
excerpt: "The image hash and certificate are not allowed error is displayed when creating a new Hyper-V machine"
last_modified_at: 2019-11-03T08:55:48-05:00
categories:

  - Windows
  - SysAdmin
  - PowerShell
  
tags:
  - HyperV
  - Virtualization
  - Tips
---

Today I was deploying a new Linux machine in my home lab, part of which is running on Hyper-V,to host a *Docker* cluster base on Linux CentOS.

After creating the VM and connecting the required ISO File, CentOS 7 in my case, I booted it just to be greeted by the **The image hash and certificate are not allowed** error message with machine failing to boot as no PXE server is configured for network boot.

The error message is caused by having *Secure Boot* enabled with an incorrect configuration for guest OS, you can easily display the configuration with the following command:

```powershell
Get-VMFirmware -VMName $vmName

VMName    SecureBoot SecureBootTemplate PreferredNetworkBootProtocol BootOrder
------    ---------- ------------------ ---------------------------- ---------
SRV-TEST  On         MicrosoftWindows   IPv4                         {File, Network, Drive, Drive}
```

As you can see in the above example the *SEcureBootTemplate* is set to *MicrosoftWindows* which is the default value for new VMs.

To change this setting use the **Set-VMFirmware** like this

```powershell
Set-VMFirmware -VMName $vmName -SecureBootTemplate MicrosoftUEFICertificateAuthority

# Very correct settings
VMName    SecureBoot SecureBootTemplate                PreferredNetworkBootProtocol BootOrder
------    ---------- ------------------                ---------------------------- ---------
SRV-TEST  On         MicrosoftUEFICertificateAuthority IPv4                         {File, Network, Drive, Drive}
```

If you want to disable *SecureBoot* all together for a specific VM, for example if it will still not boot, you can easily do that with the following command:

```powershell
Set-VMFirmware -VMName $vmName -EnableSecureBoot Off
```
