---
title: "Arranging broken F8"
slug: arranging-broken-f8
date: 2007-11-30 20:24 +0100
year: "2007"
description: "A quick checklist for repairing a Fedora 8 system after a hardware disk problem: boot the rescue DVD, bring up the LVM volumes and force an fsck."
---

Arranging broken F8 (after hardware disk problems):

- Boot the DVD (rescue system).
- Don't mount the disk once the shell is started.
- Check the lvm (LVM): `lvm vgck`
- Create the /dev/mapper specials: `lvm vgchange -a y`
- Check the disk and force the correction: `fsck /dev/mapper/VolGroup00-LogVol00 -f -y -c -c`
