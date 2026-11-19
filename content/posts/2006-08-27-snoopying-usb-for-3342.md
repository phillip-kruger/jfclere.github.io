---
title: "snoopying usb for 3342"
slug: snoopying-usb-for-3342
date: 2006-08-27 13:20 +0200
year: "2006"
description: "Trying to reverse engineer a Netopia 3342 USB ADSL modem under Linux by snooping the USB traffic from a Windows guest in VMware with usbfs_snoop."
---

Trying to get my netopia 3342 working on linux.

- install vmware + windoze + modem (manually).
- vmware uses /dev/usb
- `echo 1 > /sys/module/usbcore/parameters/usbfs_snoop`

Get a trace like:

```text
Aug 27 13:05:03 wexpert last message repeated 3 times
Aug 27 13:05:03 wexpert kernel: usb 3-1: usbdev_ioctl: SUBMITURB
Aug 27 13:05:03 wexpert kernel: usb 3-1: usbdev_ioctl: SUBMITURB
Aug 27 13:05:03 wexpert kernel: usb 3-1: usbdev_ioctl: REAPURBDELAY
Aug 27 13:05:03 wexpert last message repeated 3 times
Aug 27 13:05:03 wexpert kernel: usb 3-1: usbdev_ioctl: SUBMITURB
Aug 27 13:05:03 wexpert kernel: usb 3-1: usbdev_ioctl: SUBMITURB
Aug 27 13:05:03 wexpert kernel: usb 3-1: usbdev_ioctl: REAPURBDELAY
Aug 27 13:05:03 wexpert last message repeated 3 times
```

Not very helpful... But printk() at the right places will be quicker.
