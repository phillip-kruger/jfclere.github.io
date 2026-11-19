---
title: "Using RTL8812AU (AWUS036ACH) on Fedora 31"
slug: using-rtl8812au-awus036ach-on-fedora-31
date: 2019-12-21 08:28 +0100
year: "2019"
description: "Build and install the out-of-tree rtl8812au driver with dkms on an aarch64 Fedora 31 box so an Alfa AWUS036ACH USB wifi adapter shows up as wlan1."
---

Check via lsusb:

```text
[root@pc-86 ~]# lsusb
Bus 001 Device 004: ID 0bda:8812 Realtek Semiconductor Corp. RTL8812AU 802.11a/b/g/n/ac 2T2R DB WLAN Adapter
Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp. SMSC9512/9514 Fast Ethernet Adapter
Bus 001 Device 002: ID 0424:9514 Standard Microsystems Corp. SMC9514 Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

Clone the source (you need make, dkms, git etc):

```bash
git clone https://github.com/gnab/rtl8812au.git
dkms add ./rtl8812au
```

```text
[root@pc-86 ~]# dkms add ./rtl8812au
Creating symlink /var/lib/dkms/8812au/4.2.3/source ->
                 /usr/src/8812au-4.2.3
```

check it:

```text
[root@pc-86 ~]# dkms status
8812au, 4.2.3: added
```

build it...

```bash
ln -s /usr/src/kernels/5.3.16-300.fc31.aarch64/arch/arm64 /usr/src/kernels/5.3.16-300.fc31.aarch64/arch/aarch64
dkms build 8812au/4.2.3
```

check it:

```text
[root@pc-86 ~]# dkms status
8812au, 4.2.3, 5.3.16-300.fc31.aarch64, aarch64: built
```

install it...

```bash
dkms install 8812au/4.2.3
```

check it:

```text
[root@pc-86 ~]# dkms status
8812au, 4.2.3, 5.3.16-300.fc31.aarch64, aarch64: installed
```

and:

```text
[root@pc-86 ~]# lsmod | grep 8812au
8812au               1150976  0
[root@pc-86 ~]# iwconfig
wlan0     IEEE 802.11  ESSID:off/any
          Mode:Managed  Access Point: Not-Associated   Tx-Power=31 dBm
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Encryption key:off
          Power Management:on

wlan1     unassociated  Nickname:""
          Mode:Auto  Frequency=2.412 GHz  Access Point: Not-Associated
          Sensitivity:0/0
          Retry:off   RTS thr:off   Fragment thr:off
          Encryption key:off
          Power Management:off
          Link Quality:0  Signal level:0  Noise level:0
          Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
          Tx excessive retries:0  Invalid misc:0   Missed beacon:0

eth0      no wireless extensions.

lo        no wireless extensions.
```

Then:

```bash
yum install wpa_supplicant NetworkManager-tui
systemctl start wpa_supplicant
nmtui
```

DONE!!!
