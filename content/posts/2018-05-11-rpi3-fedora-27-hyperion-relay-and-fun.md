---
title: "RPI3 fedora 27, hyperion relay and fun with the latch of street door"
slug: rpi3-fedora-27-hyperion-relay-and-fun
date: 2018-05-11 21:30 +0200
year: "2018"
description: "Fedora 27 on a Raspberry Pi 3 as wifi access point, DHCP and DNS server, then driving a relay on GPIO 18 from httpd to open a street door latch."
---

Use the installer:

```bash
arm-image-installer --image=Fedora-Server-armhfp-27-1.6-sda.raw.xz --media=/dev/mmcblk0 --target=rpi3 --norootpass --addkey=/home/jfclere/.ssh/id_rsa.pub
```

Resize the image with the graphic tool:

```bash
gparted /dev/mmcblk0
```

Boot the PI and find its address (ifconfig to get the laptop address):

```text
nmap -sn 192.168.1.0/24
Nmap scan report for 192.168.1.39
Host is up (-0.057s latency).
MAC Address: B8:27:EB:D4:2E:85 (Raspberry Pi Foundation)
```

`ssh -l root 192.168.1.39` and you are in!!!

Remove the auto configure:

```bash
/bin/systemctl disable initial-setup.service
```

Setting timezone:

```bash
ls -lt /usr/share/zoneinfo/Europe/Zurich
rm /etc/localtime
ln -s /usr/share/zoneinfo/Europe/Zurich /etc/localtime
```

Install wifi (PI3):

```bash
curl https://fedora.roving-it.com/brcmfmac43430-sdio.txt -o /lib/firmware/brcm/brcmfmac43430-sdio.txt
reboot
```

Done:

```text
[root@localhost ~]# nmcli device status
DEVICE  TYPE      STATE      CONNECTION
eth0    ethernet  connected  eth0
lo      loopback  unmanaged  --
wlan0   wifi      unmanaged  --
```

nmtui:

```text
SSID PI2
Mode <Access Point>
Channel <Automatic>
Security <None>
Addresses 10.0.0.201/24
[X] Never use this network for default route
[X] Require IPv4 addressing for this connection
[X] Automatically connect
```

Add the dhcp server and configure it. Use https://github.com/jfclere/tomcatPI/blob/master/conf/master/dhcpd.conf, then enable and start it:

```bash
yum install dhcp-server
/bin/systemctl enable dhcpd
/bin/systemctl start dhcpd
```

Add named and configure it. Use https://github.com/jfclere/tomcatPI/blob/master/conf/master/named.conf and copy the configuration files in /etc/bind (mkdir /etc/bind), then enable and start it:

```bash
/bin/systemctl enable named
/bin/systemctl start named
```

Don't forget the firewall on the PI:

```bash
firewall-cmd --permanent --zone=FedoraServer --add-port=53/tcp
firewall-cmd --permanent --zone=FedoraServer --add-port=53/udp
firewall-cmd --reload
```

**Look to the previous blog and enable the services:**

```bash
/bin/systemctl disable initial-setup.service
/bin/systemctl enable named
/bin/systemctl enable dhcpd
```

## Getting the GPIO working

```text
[root@localhost ~]# yum search gpio
Last metadata expiration check: 2:55:33 ago on Mon 30 Apr 2018 06:50:28 CEST.
=================================================================== Summary & Name Matched: gpio ===================================================================
libgpiod-utils.armv7hl : Utilities for GPIO
sgpio.armv7hl : SGPIO captive backplane tool
libgpiod-devel.armv7hl : Development package for libgpiod
python2-RPi.GPIO.armv7hl : A class to control the GPIO on a Raspberry Pi
python3-RPi.GPIO.armv7hl : A class to control the GPIO on a Raspberry Pi
libgpiod.armv7hl : C library and tools for interacting with linux GPIO char device
```

Install and try:

```text
[root@localhost ~]# python3 jfcgpio.py
Segmentation fault (core dumped)
```

Oops... broken :-(

Install libgpiod-utils, trying...

```text
[root@localhost ~]# /usr/bin/gpioinfo
gpiochip0 - 54 lines:
        line   0:      unnamed       unused   input  active-high
        line   1:      unnamed       unused   input  active-high
...
[root@localhost ~]# /usr/bin/gpiodetect
gpiochip0 [pinctrl-bcm2835] (54 lines)
gpiochip1 [raspberrypi-exp-gpio] (8 lines)
```

Yes:

```bash
gpioset -m time -s 1 gpiochip0 18=1
```

LED ON!!!

## httpd and the door

Install httpd and start it:

```bash
/bin/systemctl enable httpd
/bin/systemctl start httpd
```

Open the firewall for httpd:

```text
[root@localhost ~]# firewall-cmd --get-default-zone
FedoraServer
```

So use FedoraServer ;-)

```bash
firewall-cmd --permanent --zone=FedoraServer --add-port=80/tcp
firewall-cmd --reload
```

Arrange the permissions (selinux = tricky):

```text
[root@localhost ~]# audit2allow -a

#============= httpd_sys_script_t ==============
allow httpd_sys_script_t gpio_device_t:chr_file { ioctl open read write };
allow httpd_sys_script_t initrc_var_run_t:file { lock open read };
allow httpd_sys_script_t pam_var_run_t:dir { add_name write };
allow httpd_sys_script_t pam_var_run_t:file { create getattr lock open read write };
allow httpd_sys_script_t self:capability { audit_write dac_read_search setgid setuid sys_resource };
allow httpd_sys_script_t self:netlink_audit_socket { create nlmsg_relay };
allow httpd_sys_script_t self:process setrlimit;
allow httpd_sys_script_t shadow_t:file { getattr open read };
allow httpd_sys_script_t sudo_db_t:dir getattr;
allow httpd_sys_script_t system_dbusd_t:dbus send_msg;
allow httpd_sys_script_t systemd_logind_t:dbus send_msg;

#============= systemd_logind_t ==============
allow systemd_logind_t httpd_sys_script_t:dbus send_msg;
```

```bash
audit2allow -a -M door
semodule -i door.pp
```

(Not working... Need more time).

Disabling selinux :_(

```text
[root@localhost ~]# sestatus
SELinux status:                 disabled
```

Add apache in sudoers: `sudoedit /etc/sudoers` and add:

```text
apache ALL=NOPASSWD: /usr/bin/gpioset -m time -s 1 gpiochip0 18=1
```

**For the details on httpd configuration, html and cgi** look to https://github.com/jfclere/door
