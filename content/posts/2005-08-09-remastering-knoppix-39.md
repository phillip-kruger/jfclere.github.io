---
title: "remastering knoppix 3.9"
slug: remastering-knoppix-39
date: 2005-08-09 20:29 +0200
year: "2005"
description: "Remastering a Knoppix 3.9 live CD on a 2.6 kernel: building cloop with a small patch, mounting the compressed filesystem, editing it and rebuilding the iso."
---

You will need cloop. I have used http://developer.linuxtag.net/knoppix/sources/cloop_2.01-5.tar.gz

To build it on linux 2.6.x, arrange the Makefile, comment out:

```text
include $(KERNEL_DIR)/conf.vars
```

and add -O2 to CKERNOPS.

In advancecomp-1.9_create_compressed_fs/advfs.cc comment out:

```text
#include <netinet/in.h>
```

compressed_loop.c needs the following patch:

```text
--- compressed_loop.c.org       2005-08-09 18:23:27.000000000 +0200
+++ compressed_loop.c   2005-08-09 18:24:44.000000000 +0200
@@ -202,7 +202,7 @@
                           unsigned long offset, unsigned long size)
 {
  char *kaddr;
- struct clo_read_data *p = (struct clo_read_data*)desc->buf;
+ struct clo_read_data *p = (struct clo_read_data*)desc->arg.buf;
  unsigned long count = desc->count;
  if (size > count) size = count;
  kaddr = kmap(page);
@@ -228,7 +228,7 @@
    read_descriptor_t desc;
    desc.written = 0;
    desc.count   = size;
-   desc.buf     = (char*)&cd;
+   desc.arg.buf     = (char*)&cd;
    desc.error   = 0;
 #ifdef REDHAT_KERNEL /* Greenshoe Linux */
    do_generic_file_read(f, &pos, &desc, clo_read_actor, 0);
```

Build it:

```bash
make KERNEL_DIR=/home/jfclere/KERNEL/linux-2.6.10
```

Insert it in the system:

```bash
/sbin/modprobe zlib_inflate
/sbin/modprobe zlib_deflate
/sbin/insmod cloop.o file=/home/jfclere/tmp/knx/master/KNOPPIX/KNOPPIX
```

Mount it:

```bash
mount -r -t iso9660 /dev/cloop /mnt/test2
```

Copy the KNOPPIX/KNOPPIX contents (from the KNOPPIX_V3.9-2005-05-27-EN.iso) into one directory (for example use `cp -Rp /mnt/test2/* KNOPPIX`). That takes a while ;-) Arrange the files (remove and add) in the directory and rebuild the compressed file system with mkisofs for example:

```bash
mkisofs -R -U -V "KNOPPIX.net filesystem" -publisher "KNOPPIX www.knoppix.net" -hide-rr-moved -cache-inodes -no-bak -pad /home/jfclere/tmp/KNOPPIX | nice -5 /home/jfclere/tmp/cloop-2.01/create_compressed_fs - 65536 > /home/jfclere/tmp/KNOPPIX/KNOPPIX
```

Copy the KNOPPIX from the original iso image and replace the file KNOPPIX/KNOPPIX by the new one (/home/jfclere/tmp/KNOPPIX/KNOPPIX here).

Build the iso image:

```bash
(cd /home/jfclere/tmp/knx/master
mkisofs -pad -l -r -J -v -V "KNOPPIX" -no-emul-boot -boot-load-size 4 -boot-info-table -b boot/isolinux/isolinux.bin -c boot/isolinux/boot.cat -hide-rr-moved -o /home/jfclere/jfc.iso /home/jfclere/tmp/knx/master
)
```
