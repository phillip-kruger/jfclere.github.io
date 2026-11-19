---
title: "hostap with Siemens I-GATE 11M PCI"
slug: hostap-with-siemens-i-gate-11m-pci
date: 2005-07-06 00:08 +0200
year: "2005"
description: "Swapping the orinoco driver for hostap on a PrismII based Siemens I-GATE 11M card in a PLX9052 PCI to PCMCIA adapter, and bringing the interface up."
---

That is a PrismII based PCMCIA card in the PLX9052 PCI-PCMCIA adapter. Follow the README of the hostap driver:

```bash
make
make install    # (as root)
```

Remove the orinoco modules:

```bash
ifconfig eth1 down
rmmod orinoco_plx
rmmod orinoco
```

Try the new one:

```bash
modprobe hostap_plx
```

Check it:

```text
ifconfig -a
wifi0     Link encap:UNSPEC  HWaddr 00-90-D1-00-76-4E-00-00-00-00-00-00-00-00-00-00
         BROADCAST MULTICAST  MTU:1500  Metric:1
         RX packets:0 errors:0 dropped:0 overruns:0 frame:0
         TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
         collisions:0 txqueuelen:1000
         RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)
         Interrupt:11 Base address:0x3000

wlan0     Link encap:Ethernet  HWaddr 00:90:D1:00:76:4E
         BROADCAST MULTICAST  MTU:1500  Metric:1
         RX packets:0 errors:0 dropped:0 overruns:0 frame:0
         TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
         collisions:0 txqueuelen:0
         RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)
         Interrupt:11 Base address:0x3000
```

Configure it as below:

```bash
ifconfig wlan0 inet 10.170.254.37
ifconfig wlan0 netmask 255.255.255.0
ifconfig wlan0 broadcast 10.170.254.255

iwconfig wlan0 essid ATMEL
iwconfig wlan0 channel 1
iwconfig wlan0 mode managed
```

ping wexpert doesn't work, why?
