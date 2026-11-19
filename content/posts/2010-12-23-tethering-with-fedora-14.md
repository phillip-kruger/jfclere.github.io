---
title: "Tethering with Fedora 14 with iphone"
slug: tethering-with-fedora-14
date: 2010-12-23 10:33 +0100
year: "2010"
description: "Tethering an iPhone on Fedora 14 turned out to be easy: the ipheth driver loads on its own and dhclient on wwan0 brings the connection up."
---

It looks very easy now:

```bash
yum install libimobiledevice    # (really needed?)
```

dmesg tells:

```text
[ 3202.355929] ipheth 1-8.4:4.2: Apple iPhone USB Ethernet device attached
[ 3202.355982] usbcore: registered new interface driver ipheth
```

In network manager I am able to configure it but not to activate it (maybe I have something wrong in my box).

Using

```bash
dhclient wwan0
```

gets it working :D

Note that the iphone is running 4.2.1 and my provider is swisscom.
