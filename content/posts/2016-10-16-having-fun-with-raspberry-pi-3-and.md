---
title: "Having fun with raspberry pi 3 and Astro Hat."
slug: having-fun-with-raspberry-pi-3-and
date: 2016-10-16 18:19 +0200
year: "2016"
description: "Long working notes on Fedora 24 on a Raspberry Pi 3: hand-built SD card, wifi access point with DHCP and bind, Tomcat demo, Sense HAT examples and STOMP to ActiveMQ."
---

## Basic installation

From: https://fedoraproject.org/wiki/Raspberry_Pi

```bash
fdisk /dev/mmcblk0
```

unmount and reinsert.

Looking for the images: https://dl.fedoraproject.org/pub/fedora/linux/releases/24/Spins/armhfp/images/

There are a bunch of them... https://dl.fedoraproject.org/pub/fedora/linux/releases/24/Server/armhfp/images/

Only one, I have picked this one.

```bash
losetup --partscan --find --show /home/jfclere/Downloads/Fedora-Server-armhfp-24-1.2-sda.raw /dev/loop0
mount -r /dev/loop0p3 /tmp/img/
mkfs.vfat /dev/mmcblk0p1
mkswap /dev/mmcblk0p2
mkfs.ext4 /dev/mmcblk0p3
mkdir /tmp/rpi/
mount /dev/mmcblk0p3 /tmp/rpi/
mkdir /tmp/rpi/boot/
mount /dev/mmcblk0p1 /tmp/rpi/boot/
cp -rpv /tmp/img/* /tmp/rpi/
sync    # (takes ages)
losetup -d /dev/loop0
```

```text
[root@jfcpc NOTES]# blkid | grep mmcblk0
/dev/mmcblk0: PTTYPE="dos"
/dev/mmcblk0p1: SEC_TYPE="msdos" UUID="DB85-3C4A" TYPE="vfat"
/dev/mmcblk0p2: UUID="5f07dbe9-2619-481e-b0b3-5f37a7f011bf" TYPE="swap"
/dev/mmcblk0p3: UUID="8df2bd5c-4692-4f80-96ec-da2e9697a713" TYPE="ext4"
```

Edit and adjust /tmp/rpi/etc/fstab.

In /tmp/rpi/boot/cmdline.txt add:

```text
dwc_otg.lpm_enable=0 console=ttyAMA0,115200 console=tty1 root=/dev/mmcblk0p3 rootfstype=ext4 elevator=deadline rootwait
```

https://www.raspberrypi.org/documentation/configuration/config-txt.md (configuration stuff).

/tmp/rpi/boot/config.txt (do we need something?...) empty for the moment...

Copy the boot and modules. Download the firmware: https://github.com/raspberrypi/firmware

What is https://github.com/raspberrypi/firmware/archive/master.zip? It is like https://github.com/modcluster/mod_cluster/archive/master.zip (just a zip repo). Download and unzip.

So https://github.com/raspberrypi/firmware/tree/master/boot

```text
[root@jfcpc NOTES]# cp -r /home/jfclere/Downloads/firmware-master/boot/* /tmp/rpi/boot/
```

So https://github.com/raspberrypi/firmware/tree/master/modules (for + and -v7+ no idea how it is chosen).

Once running, doing more /proc/cpuinfo:

```text
processor : 0
model name : ARMv7 Processor rev 4 (v7l)
```

guessing we use the -v7+ ;-)

```text
[root@jfcpc NOTES]# cp -r /home/jfclere/Downloads/firmware-master/modules/* /tmp/rpi/lib/modules/
```

```bash
sync
umount /tmp/rpi/boot/
umount /tmp/rpi/
```

Network: (trying to check it)...

In fact the fedora24 uses systemd. /tmp/rpi/etc/systemd/ (there is something run at the first boot).

```text
/tmp/rpi/etc/systemd/system/default.target -> /lib/systemd/system/multi-user.target
/tmp/rpi/lib/systemd/system/systemd-firstboot.service ?
```

and it runs /usr/bin/systemd-firstboot --prompt-locale --prompt-timezone --prompt-root-password

```properties
[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/systemd-firstboot --prompt-locale --prompt-timezone --prompt-root-password
```

For the curious:

```text
[jfclere@jfcpc ~]$ rpm -qf /usr/bin/systemd-firstboot
systemd-229-13.fc24.x86_64
```

```text
--locale=en_US.UTF-8
--timezone=Europe/Paris
--root-password=Changeit2016
```

```bash
nmap -sn 192.168.1.0/24
```

found the PI:

```text
Nmap scan report for pc-151.home (192.168.1.111)
Host is up (0.047s latency).
MAC Address: B8:27:EB:C1:44:41 (Raspberry Pi Foundation)
```

scanning it:

```text
[root@jfcpc Downloads]# nmap 192.168.1.111

Starting Nmap 7.12 ( https://nmap.org ) at 2016-09-07 22:15 CEST
Nmap scan report for pc-151.home (192.168.1.111)
Host is up (0.0044s latency).
Not shown: 998 filtered ports
PORT     STATE SERVICE
22/tcp   open  ssh
9090/tcp open  zeus-admin
MAC Address: B8:27:EB:C1:44:41 (Raspberry Pi Foundation)

Nmap done: 1 IP address (1 host up) scanned in 4.60 seconds
```

Somehow the password stuff failed... The stuff still asks :-( Copying my fedora laptop password helps...

```text
[root@pc-151 ~]# ps -ef | grep tty1
root       343     1  0 18:23 tty1     00:00:00 /bin/bash /usr/libexec/initial-setup/run-initial-setup
root       348   343 12 18:23 tty1     00:00:23 /usr/bin/python3 /usr/libexec/initial-setup/initial-setup-text --no-stdout-log
root       720   696  0 18:26 pts/0    00:00:00 grep --color=auto tty1
```

In it: `/bin/systemctl disable initial-setup.service` (so it removes itself), and it is a python script...

trying... mount

```text
/dev/mmcblk0p3 on /run/media/jfclere/a440011a-a5ea-4eb7-ae01-95c8cd981673
[root@jfcpc Downloads]# grep ENCRYPT_METHOD /tmp/img/etc/login.defs
ENCRYPT_METHOD SHA512
```

```bash
echo -n root:Changeit2016 | chpasswd -c SHA512 -R /run/media/jfclere/a440011a-a5ea-4eb7-ae01-95c8cd981673
```

selinux problems :-( and doesn't work.

```bash
echo -n root:Changeit2016 | chpasswd -c SHA512
```

in the PI works...

Setting timezone:

```bash
ls -lt /usr/share/zoneinfo/Europe/Zurich
rm /etc/localtime
ln -s /usr/share/zoneinfo/Europe/Zurich /etc/localtime
```

easy :D

Don't forget to remove the base fedora kernel components (yum might get funny otherwise)...

```bash
dnf remove kernel kernel-core kernel-modules
```

## Configure wifi

The goal is to get a sort of captive portal to run an easy demo: the raspberry is acting as an AP and dhcp server and name server.

```text
[root@pc-111 ~]# nmcli device status
DEVICE  TYPE      STATE      CONNECTION
eth0    ethernet  connected  eth0
lo      loopback  unmanaged  --
```

no wifi. From https://fedoraproject.org/wiki/Raspberry_Pi:

```bash
cd /lib/firmware/brcm/
curl -O https://raw.githubusercontent.com/RPi-Distro/firmware-nonfree/master/brcm80211/brcm/brcmfmac43430-sdio.bin
curl -O https://raw.githubusercontent.com/RPi-Distro/firmware-nonfree/master/brcm80211/brcm/brcmfmac43430-sdio.txt
```

reboot?

```text
iw list
Supported interface modes:
* IBSS
* managed
* AP
```

it can do AP, cool...

```bash
dnf install NetworkManager-tui NetworkManager-wifi
nmtui
```

```text
SSID=PI
Mode <Access Point>
Channel <Automatic>
Security <None>
Addresses 10.0.0.201/24
[X] Never use this network for default route
[X] Require IPv4 addressing for this connection
[X] Automatically connect
```

from https://fedoraproject.org/wiki/Raspberry_Pi

From the laptop:

```text
wlp4s0    IEEE 802.11  ESSID:"PI"
Mode:Managed  Frequency:2.412 GHz  Access Point: B8:27:EB:94:11:14
```

in PI:

```text
wlan0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
ether b8:27:eb:94:11:14  txqueuelen 1000  (Ethernet)
```

hm, laptop doesn't get IP?

try dhcp (server?) 10.0.0.0 (give 10.0.0.201 to the PI). Look to /etc/dhcp/dhcpd.conf. The important:

```text
subnet 10.0.0.0 netmask 255.255.255.0 {
```

so it listens on 10.0.0.201

```text
option domain-name "mysrv.world";
option domain-name-servers 10.0.0.201;
```

so the names are resolved by the bind of PI.

```text
option routers 10.0.0.201
```

so it is the router for the interface (we can use the PI as router!!!).

Looks nearly OK...

```text
Listening on LPF/wlan0/b8:27:eb:94:11:14/10.0.0.0/24
Sending on   LPF/wlan0/b8:27:eb:94:11:14/10.0.0.0/24
```

ifconfig on the laptop:

```text
wlp4s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
inet 10.0.0.200  netmask 255.255.255.0  broadcast 10.0.0.255
```

nmap -sn 10.0.0.0/24 (it finds the PI):

```text
[root@jfcpc APACHE-2.4.x]# nmap -sn 10.0.0.0/24

Starting Nmap 7.12 ( https://nmap.org ) at 2016-09-14 23:41 CEST
Nmap scan report for 10.0.0.201
Host is up (0.014s latency).
MAC Address: B8:27:EB:94:11:14 (Raspberry Pi Foundation)
Nmap scan report for 10.0.0.200
Host is up.
Nmap done: 256 IP addresses (2 hosts up) scanned in 32.09 seconds
```

easy... 192.168.1.109 (connected).

To find the address: `nmap -sn 192.168.1.0/24` (doesn't help... well run it as root = better...)

```text
Nmap scan report for pc-111.home (192.168.1.108)
Host is up (0.0067s latency).
MAC Address: B8:27:EB:7A:A6:98 (Raspberry Pi Foundation)
Nmap scan report for pc-111.home (192.168.1.109)
Host is up (0.0067s latency).
```

Name server (based on http://serverfault.com/questions/396958/configure-dns-server-to-return-same-ip-for-all-domains):

```bash
yum install bind
```

/etc/named.conf:

```text
listen-on port 53 { 10.0.0.201; };
allow-query     { 10.0.0.0/24; };
recursion no;
zone "." IN {
type master;
file "/etc/bind/db.fakeroot";
};
```

And in /etc/bind/db.fakeroot:

```text
@ IN SOA ns.domain.com. hostmaster.domain.com. ( 1 3h 1h 1w 1d )
IN NS 10.0.0.201
* IN A  10.0.0.201
```

Don't forget the firewall on the PI:

```bash
firewall-cmd --permanent --zone=FedoraServer --add-port=53/tcp
firewall-cmd --permanent --zone=FedoraServer --add-port=53/udp
firewall-cmd --reload
```

## Getting tomcat running

yum install java (should be ready for tomcat).

```bash
yum install java maven
git clone https://github.com/jfclere/tomcatPI
cd tomcatPI/demo
java -jar ./target/demo-1.0-SNAPSHOT-jar-with-dependencies.jar
```

Don't forget the firewall (again).

```bash
firewall-cmd --permanent --zone=FedoraServer --add-port=80/tcp
firewall-cmd --reload
```

## Getting Astro Hat examples running

HAT: https://github.com/RPi-Distro/python-sense-hat and https://github.com/raspberrypi/hats. https://github.com/RPi-Distro/python-sense-hat looks the good one...

A lot needs to be installed: gcc/python-devel/gcc-gfortran/redhat-rpm-config (hard to guess this one!) and building takes ages... libz-devel, jpeg-devel.

Testing... /root/python-sense-hat/examples

Missing https://github.com/RPi-Distro/RTIMULib, cmake/gcc-c++.

- in RTIMULib/RTIMULib/build (mkdir) cmake ... make, make install.
- in RTIMULib/Linux need qt-devel
- in RTIMULib/Linux/python (probably the only one really needed!).

./python-sense-hat/examples/colour_cycle.py (works...)

./RTIMULib/Linux/build/RTIMULibCal/RTIMULibCal (to get the RTIMULib.ini to put in /etc according to the doc).

### Playing with the 8X8 frame buffer

```text
[root@jfcpc java]# pwd
/home/jfclere/tomcatPI/demo/src/main/java
[root@jfcpc java]# java -cp . org.jfclere.tomcatPI.PIFrameBuffer
file is 4096Exception in thread "main" java.io.IOException: No such device
at sun.nio.ch.FileChannelImpl.map0(Native Method)
at sun.nio.ch.FileChannelImpl.map(FileChannelImpl.java:926)
at org.jfclere.tomcatPI.PIFrameBuffer.main(PIFrameBuffer.java:22)
```

OK, it can't read it :-(

According to the Python code (`from sense_hat import SenseHat`) the LED matrix is an RGB565 framebuffer. More or less working, now committed: https://github.com/jfclere/tomcatPI/blob/master/demo/src/main/java/org/jfclere/tomcatPI/PIFrameBuffer.java

## Sending information to ActiveMQ and consuming it

```bash
yum install stomppy
```

Install activemq on the laptop and check the stomp port:

```text
INFO | Listening for connections at: stomp://jfcpc:61613?maximumConnections=1000&wireFormat.maxFrameSize=104857600
INFO | Connector stomp started
```

http://127.0.0.1:8161/admin/ (we receive something binary UTF8 encoded string).

Write a STOMP logic to send messages to ActiveMQ (running on my laptop): https://github.com/jfclere/tomcatPI/tree/master/python

Write a small jms client to consume the messages from the raspberry: https://github.com/jfclere/tomcatPI/tree/master/reader
