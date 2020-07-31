---
title: "CentOS 8 - No updates available"
excerpt: "After deploying a new CentOS server and running a dnf update you could receive a Nothing to do, Complete! message despite updates being available"
categories:
  - Linux
tags:
  - Tips

toc: false
---

Today I deployed a new CentOS 8.1 machine, yes I'm still doing some Linux, and immediately after installing the OS I ran the update command which did not yield any result

```bash
    dnf update

    Dependencies resolved.
    Nothing to do.
    Complete!
```

It turns out this a documented bug affecting even recent build, I generally use the *minimal* or *boot* ISOs, builds [you can read of it here](https://wiki.centos.org/Manuals/ReleaseNotes/CentOSStream#Stream_1905_users_need_to_take_action_after_install).

Solution is really simple issue the following command:

```bash
dnf install -y centos-release-stream

dnf update
```

This time updates will be correctly downloaded and installed on the system if you choose so.
