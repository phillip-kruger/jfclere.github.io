---
title: "Upgrading from F6 to F7 on Dell Latitude D620"
slug: upgrading-from-f6-to-f7-on-dell
date: 2007-06-24 16:43 +0200
year: "2007"
description: "The Fedora 6 to Fedora 7 upgrade on a Dell Latitude D620 hung on boot, so I booted a borrowed kernel, reconfigured X and updated to the shipping kernel."
---

I have booted from the DVD and chosen "upgrade system". After a long while the install process asks for a reboot... But the box doesn't boot. It hangs here:

```text
TCP established hash table entries: 131072 (order: 8, 1572864 bytes)
TCP bind hash table entries: 65536 (order: 7, 524288 bytes)
TCP: Hash tables configured (established 131072 bind 65536)
```

I have rebooted from the DVD and chosen recovery boot. Once booted I have copied the kernel/initrd/System.map and /lib/modules of another machine (2.6.21-1.3228.fc7) to the box, edited the boot/grub/menu.lst to add a new entry for the copied kernel, and rebooted.

The reboot doesn't have X running, so I accepted the reconfiguration and got X running (800x600 instead of my expected 1440x900). From there I upgraded to the last version (using the Package Updater). Reboot and the expected resolution is now working (I have not yet tested the dual head). The improvement is that I don't need the nvidia driver.

Trying Suspend, the machine suspends but never comes up (that was working in Fedora 6) ;-( Trying Hibernate, the machine stops correctly and comes up with problems (that wasn't working with fedora 6).

Network ethernet and wifi are working like in Fedora 6.

Strange thing: the Package Updater wants to upgrade to kernel-2.6.21-1.3228.fc7 even though I have already done that.

In the Application menu I have a .hidden entry that contains k3b entries (2).

Evolution hangs like in Fedora 6 (I am using imap) but also hangs the box (by eating all the memory).
