---
title: "Getting DS1307 on RPI3 with fedora 24"
slug: getting-ds1307-on-rpi3-with-fedora-24
date: 2017-09-10 18:47 +0200
year: "2017"
description: "Make a DS1307 real-time clock work on a Raspberry Pi 3 under Fedora 24: device tree overlay, chrony, a hwclock unit and serving time to the other Pis."
---

1 - Add in the /boot/config.txt:

```text
dtoverlay=i2c-rtc,ds1307
```

In dmesg:

```text
[    5.536541] rtc-ds1307 1-0068: rtc core: registered ds1307 as rtc0
[    5.540786] rtc-ds1307 1-0068: 56 bytes nvram
```

The modules are loaded at boot now...

```text
[root@pc-8 ~]# i2cdetect -y 1
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- UU -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
[root@pc-8 ~]# hwclock -r
2017-09-09 17:58:49.868784+1:00
```

(well not too bad after 2 months (+~1 minute))

2 - Get chrony to resynchronize my hardware clock (if I have a connection!)

In /etc/chrony.conf:

```text
# Enable kernel synchronization of the real-time clock (RTC).
rtcsync
# Serve time even if not synchronized to any NTP server.
local stratum 10
```

3 - Get the hard clock set at the RPI3 start time:

Create a /etc/systemd/system/hwclock.service:

```properties
[Unit]
Description=Setup date and time via hwclock
[Service]
Type=oneshot
ExecStart=/usr/sbin/hwclock --hctosys
[Install]
WantedBy=multi-user.target
```

and do:

```bash
systemctl enable hwclock
```

System time is set correctly...

```text
[root@pc-8 ~]# timedatectl status
      Local time: Sat 2017-09-09 19:35:58 CEST
  Universal time: Sat 2017-09-09 17:35:58 UTC
        RTC time: Sat 2017-09-09 17:35:58
       Time zone: Europe/Zurich (CEST, +0200)
 Network time on: yes
NTP synchronized: no
 RTC in local TZ: no
```

Don't forget the firewall...

```bash
firewall-cmd --permanent --zone=FedoraServer --add-port=123/udp
firewall-cmd --reload
```

4 - Make sure the clients (HypriotOS in my case) use the time server:

Add the RPI3 address in /etc/systemd/timesyncd.conf (here 10.0.0.201 is the RPI3 with the hardware clock):

```properties
[Time]
Servers=10.0.0.201
```

```bash
service systemd-timesyncd restart
```
