---
title: "Write RaspBerry image to SD card via command line"
excerpt: "In this post we will explore how to use bash to write an OS image to an SD card for use with RaspBerryPI"
categories:
  - Linux
  - Azure

tags:
  - Linux
  - Lab

toc: false
header:
    teaser: "/assets/images/Kubernetes_Logo.png"
---

I am in the process of rebuilding my *Docker/Kubernetes* portable cluster which I build using a couple of RaspberryPi 4 and as part of this I needed to *reflash* the various SD cards where operating system for each node is installed.

Usually [Balena Etcher](https://www.balena.io/etcher/) is my go-to tool for such endeavours but being in a rush and not easy way to download the tool on my Linux box I simply used the good old command line, here is how this is done.

First of all we need to locate the device mapped to our SD card, in my case I'm using a microSD to USB adapter, which can be done with the following command:

```bash
sudo fdisk -l

Disk /dev/sda: 931.51 GiB, 1000204886016 bytes, 1953525168 sectors
Disk model: WDC WD1003FZEX-0
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0x806d3748

Device     Boot      Start        End   Sectors   Size Id Type
/dev/sda1  *          2048    1126399   1124352   549M  7 HPFS/NTFS/exFAT
/dev/sda3       1024002048 1953519615 929517568 443.2G  7 HPFS/NTFS/exFAT


Disk /dev/sdb: 465.76 GiB, 500107862016 bytes, 976773168 sectors
Disk model: Samsung SSD 850 
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xde0a016a

Device     Boot   Start       End   Sectors   Size Id Type
/dev/sdb1  *       2048   2099199   2097152     1G 83 Linux
/dev/sdb2       2099200 976773119 974673920 464.8G 83 Linux

<snip for brevity>

Device     Boot  Start     End Sectors  Size Id Type
/dev/sdd1  *      2048  526335  524288  256M  c W95 FAT32 (LBA)
/dev/sdd2       526336 6819619 6293284    3G 83 Linux
```

In my case the SD card is mapped to device **/dev/sdd**, once we have found the device to use the following command:

```bash
xzcat ./ubuntu-21.04-preinstalled-server-arm64+raspi.img.xz | sudo dd bs=4M of=/dev/sdd conv=fsync
0+425226 records in
0+425226 records out
3491662848 bytes (3.5 GB, 3.3 GiB) copied, 99.9634 s, 34.9 MB/s
```

In the above example I used **Ubuntu Server 21.04** as the operating system with the image file being stored in the same path where the command is being run but the same can be used with any other image file.

I hope this can be useful and probably in the future I will post how to easily build a portable Docker/Kubernetes cluster lab.
