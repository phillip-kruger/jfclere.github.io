---
title: "fedora9 x86_64 with nvidia and centrino (d620)"
slug: fedora9-x8664-with-nvidia-and-centrino
date: 2008-05-25 10:37 +0200
year: "2008"
description: "Moving a Dell Latitude D620 from Fedora 8 i386 to Fedora 9 x86_64: clearing conflicting packages, a working nvidia driver and the right centrino firmware."
---

In fact I am trying to move from F8-386 to F9-x86_64.

## Upgrading

- Via upgrade installation... It doesn't work (F8 i386 packages are not replaced).
- Via install: if you use the custom partitioning and don't change the partition layout and don't format, you will keep your data but the F8 i386 packages are conflicting.
- Remove the conflicting packages (via rescue installed system):

```bash
rpm --root /mnt/sysimage/ -qa | grep i386 | xargs rpm --root /mnt/sysimage/ --nodeps --erase
rpm --root /mnt/sysimage/ -qa | grep fc8 | xargs rpm --root /mnt/sysimage/ --nodeps --erase
```

avahi (fc7.i386) has a broken postun scriptlet, use --nopostun for it.

- Redo the install as previous.
- yum update (to use a 2.6.25.3-18 kernel).

## Fixing remaining problems

Remaining problems (now investigating with a 4 Gbytes USB key as F9 disk)...

Dual screen doesn't work (solution: use nvidia 173.14.05):

- default looks ok, but no dual screen.
- xorg-drv-nouveau: gives a bad resolution and strange (green colours) when randr12 on.
- nvidia: doesn't work (X doesn't start, not the right ABI from http://www.nvidia.com/object/linux_display_amd64_173.08.html).
- nouveau from git (080525): see http://nouveau.freedesktop.org/wiki/InstallNouveau , use the Fedora location (drm and xf86-video-nouveau): `./configure --prefix=/usr --libdir=/usr/lib64`. Restore the libglx.so if you tried nvidia before (`yum reinstall xorg-x11-server-Xorg`).
- nvidia try 2: it works with http://www.nvidia.com/object/linux_display_amd64_173.14.05.html

wifi (centrino) doesn't work (solution: use intel firmware 2.14.1.5 and stop NetworkManager):

- default doesn't work (well, not well enough to use it).
- remove the firmwares that are installed and don't correspond to the 3945 board:

```bash
yum erase ipw2100-firmware
yum erase ipw2200-firmware
yum erase iwl4965-firmware
```

normally something like the following should be done:

```bash
yum search 3945
yum install iwl3945-firmware
```

But in fact that doesn't help, because with the iwl3945 driver it complains (do `dmesg | grep iw`) that the firmware seems to be the same version (2.14.1.5-2?) as the F8, and I am using the iwl3945 driver too. After retest it seems that it also doesn't work now on F8. It is working using the 2.14.1.5 firmware from http://intellinuxwireless.org/?p=iwlwifi&n=downloads on both Fedora versions (in /lib/firmware/iwlwifi-3945-1.ucode). It seems that stopping the NetworkManager service helps (even with 2.14.1.5-2).

PC speaker makes loud beeps. In /etc/modprobe.d/blacklist add:

```text
install pcspkr /bin/true
```
