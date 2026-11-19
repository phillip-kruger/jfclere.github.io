---
title: "Wifi on Latitude D810 (Intel 2200BG)"
slug: wifi-on-latitude-d810-intel-2200bg
date: 2006-10-05 10:17 +0200
year: "2006"
description: "Getting the Intel PRO/Wireless 2200BG working on a Dell Latitude D810 under RHEL4 and Fedora: install the firmware, mind the hardware kill switch and reload ipw2200."
---

On RHEL4 (and Fedora) only the firmware needs to be updated. To check you have the right board do `/sbin/lspci`, the output should contain the following entry:

```text
03:03.0 Network controller: Intel Corporation PRO/Wireless 2200BG Network Connection (rev 05)
```

Get the firmware from http://ipw2200.sourceforge.net/firmware.php?fid=6 and extract it in /lib/firmware:

```bash
cd /lib/firmware
sudo tar zxvf /media/usbdisk/ipw2200-fw-2.4.tgz
```

For example. Reboot the machine, the new firmware will be in use. Configure it using System Settings + Network.

If it does not work check dmesg for:

```text
ipw2200: Detected Intel PRO/Wireless 2200BG Network Connection
divert: allocating divert_blk for eth1
ipw2200: Radio Frequency Kill Switch is On:
Kill switch must be turned off for wireless networking to work.
divert: freeing divert_blk for eth1
```

Do `/sbin/rmmod ipw2200` and press the blue key Fn and the normal F2 (where there is a logo like an antenna). Do `/sbin/modprobe ipw2200` and check again:

```text
ipw2200: Intel(R) PRO/Wireless 2200/2915 Network Driver, git-1.0.10
ipw2200: Copyright(c) 2003-2005 Intel Corporation
ACPI: PCI interrupt 0000:03:03.0[A] -> GSI 17 (level, low) -> IRQ 177
ipw2200: Detected Intel PRO/Wireless 2200BG Network Connection
divert: allocating divert_blk for eth1
```

note: @myhome the wifi id is: ATMEL and the mac: 00:16:6F:60:AE:67
