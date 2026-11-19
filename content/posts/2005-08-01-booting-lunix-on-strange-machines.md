---
title: "booting lunix on 'strange' machines (using one floppy)"
slug: booting-lunix-on-strange-machines
date: 2005-08-01 23:53 +0200
year: "2005"
description: "Booting Linux on machines without a usable CD drive by building a small modular kernel and a single grub boot floppy that chainloads the real disk."
---

Boot knoppix (on a machine with a bootable CD drive). In a console, format and create the boot floppy using the following commands:

```bash
mkfs.ext2 /dev/fd0
fsck -f -t ext2 /dev/fd0
mkboot -d /dev/fd0
```

That doesn't work.... Too big!!!

Build a small kernel by removing drivers and configuring most as modules.

```text
ls -lt ./arch/i386/boot/bzImage
-rw-r--r--    1 jfclere  users      835540 2005-08-01 22:40
./arch/i386/boot/bzImage
```

Do the job:

```bash
mkfs.ext2 /dev/fd0
fsck -f -t ext2 /dev/fd0
mount /dev/fd0 /mnt/test1
mkdir -p /mnt/test1/boot/grub
cp /home/jfclere/KERNEL/linux-2.6.10/arch/i386/boot/bzImage /mnt/test1
cp /home/jfclere/tmp/initrd /mnt/test1
cp /boot/grub/menu.lst /mnt/test/boot/grub
cp /boot/grub/stage1 /mnt/test/boot/grub
cp /boot/grub/stage2 /mnt/test/boot/grub
```

menu.lst contains:

```text
color white/blue black/light-gray
timeout 8

title linuxtest
  kernel (fd0)/bzImage root=/dev/hda1 vga=791
  initrd (fd0)/initrd
title floppy
  root (fd0)
  chainloader +1
```
