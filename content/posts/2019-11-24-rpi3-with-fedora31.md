---
title: "RPI3 with Fedora31"
slug: rpi3-with-fedora31
date: 2019-11-24 18:51 +0100
year: "2019"
description: "Install Fedora Server 31 on a Raspberry Pi 3 with arm-image-installer, wire up a DS1307 clock with chrony and hwclock, and get the onboard wifi working."
---

## Get the image and install it

Get the image: https://dl.fedoraproject.org/pub/fedora/linux/releases/31/Server/aarch64/images/Fedora-Server-31-1.9.aarch64.raw.xz

Copy to SD card:

```bash
arm-image-installer --image=/home/jfclere/Downloads/Fedora-Server-31-1.9.aarch64.raw.xz --target=rpi3 --media=/dev/mmcblk0 --norootpass --addkey=/home/jfclere/.ssh/id_rsa.pub --resizefs --relabel --selinux=ON
```

Easy but see https://bugzilla.redhat.com/show_bug.cgi?id=1785109

Boot it and use ssh to connect to it.

## DS1307 on RPI3 with fedora 31

```bash
yum install i2c-tools
```

Add in /boot/efi/config.txt:

```text
dtoverlay=i2c-rtc,ds1307
```

Check dmesg:

```text
[root@pc-75 ~]# dmesg | grep rtc
[    7.407923] hctosys: unable to open rtc device (rtc0)
[   15.057637] rtc-ds1307 1-0068: registered as rtc0
```

Adjust /etc/chrony.conf:

```text
# Enable kernel synchronization of the real-time clock (RTC).
rtcsync
# Serve time even if not synchronized to any NTP server.
local stratum 10
```

Create a /etc/systemd/system/hwclock.service to get the hard clock set at the RPI3 start time:

```properties
[Unit]
Description=Setup date and time via hwclock
[Service]
Type=oneshot
ExecStart=/usr/sbin/hwclock --hctosys
[Install]
WantedBy=multi-user.target
```

Enable it and check it:

```text
[root@pc-75 ~]# systemctl enable hwclock
Created symlink /etc/systemd/system/multi-user.target.wants/hwclock.service -> /etc/systemd/system/hwclock.service.
[root@pc-75 ~]# timedatectl set-local-rtc 0
[root@pc-75 ~]# timedatectl status
               Local time: Sun 2019-11-24 16:08:08 CET
           Universal time: Sun 2019-11-24 15:08:08 UTC
                 RTC time: Sun 2019-11-24 15:08:07
                Time zone: Europe/Zurich (CET, +0100)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

Set the firewall so we can send the date :D

```text
[root@pc-75 ~]# firewall-cmd --get-default-zone
FedoraServer
[root@pc-75 ~]# firewall-cmd --permanent --zone=FedoraServer --add-port=123/udp
success
[root@pc-75 ~]# firewall-cmd --reload
success
```

## Prepare the wifi

All looked good but not working...

```text
[root@pc-75 ~]# cd /lib/firmware/brcm/
[root@pc-75 brcm]# ls -lt brcmfmac43430-sdio.txt
ls: cannot access 'brcmfmac43430-sdio.txt': No such file or directory
[root@pc-75 brcm]# ls -lt brcmfmac43430-sdio.raspberrypi,3-model-b.txt
-rw-r--r--. 1 root root 872 Sep 24 15:53 brcmfmac43430-sdio.raspberrypi,3-model-b.txt
[root@pc-75 brcm]# cp brcmfmac43430-sdio.raspberrypi,3-model-b.txt brcmfmac43430-sdio.txt
```

init 0 and wait...

```text
[root@pc-75 ~]# nmcli dev show wlan0
GENERAL.DEVICE:                         wlan0
GENERAL.TYPE:                           wifi
GENERAL.HWADDR:                         E2:65:23:25:88:06
GENERAL.MTU:                            1500
GENERAL.STATE:                          20 (unavailable)
GENERAL.CONNECTION:                     --
GENERAL.CON-PATH:                       --
```

weird, looks broken...

Looking with another usb wifi, the same for nmcli dev show wlan1:

```text
Nov 24 18:04:13 pc-75.home NetworkManager[832]:   [1574615053.7791] device (wlan1): driver supports Access Point (AP) mode
Nov 24 18:04:13 pc-75.home NetworkManager[832]:   [1574615053.7846] manager: (wlan1): new 802.11 Wi-Fi device (/org/freedesktop/NetworkManager/Devices/4)
Nov 24 18:04:13 pc-75.home NetworkManager[832]:   [1574615053.8954] device (wlan1): state change: unmanaged -> unavailable (reason 'managed', sys-iface-state: 'external')
Nov 24 18:04:14 pc-75.home NetworkManager[832]:   [1574615054.5274] device (wlan1): set-hw-addr: set MAC address to 5A:2F:BD:3A:A0:D4 (scanning)
```

Looking to wpa_supplicant: it is stopped, not even installed!!! Problem fixed :D

```text
[root@pc-75 ~]# journalctl -b -u NetworkManager | grep wpa
Nov 24 17:35:54 localhost.localdomain NetworkManager[832]:   [1574613354.9928] supplicant: wpa_supplicant stopped
```

Install it with "yum install wpa_supplicant" and start it:

```text
[root@pc-75 ~]# systemctl enable wpa_supplicant
Created symlink /etc/systemd/system/multi-user.target.wants/wpa_supplicant.service -> /usr/lib/systemd/system/wpa_supplicant.service.
[root@pc-75 ~]# systemctl start wpa_supplicant
[root@pc-75 ~]# journalctl -b -u NetworkManager | grep wpa
Nov 24 17:35:54 localhost.localdomain NetworkManager[832]:   [1574613354.9928] supplicant: wpa_supplicant stopped
Nov 24 18:23:53 pc-75.home NetworkManager[832]:   [1574616233.5877] supplicant: wpa_supplicant running
[root@pc-75 ~]# nmtui
```

Working!!!
