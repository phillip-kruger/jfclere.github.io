---
title: "Build harmony on linux 386"
slug: build-harmony-on-linux-386
date: 2005-12-03 18:16 +0100
year: "2005"
description: "An Apache Harmony build fails on an old i386 box because the libc has no SHM_HUGETLB, and the one-line patch to hyvmem.c that gets it compiling."
---

```text
ant
BUILD FAILED
/home/jfclere/harmony/enhanced/trunk/sandbox/contribs/ibm_core/build.xml:41: The following error occurred while executing this line:
/home/jfclere/harmony/enhanced/trunk/sandbox/contribs/ibm_core/native-src/build.xml:97: exec returned: 2

Total time: 31 seconds
```

Look in ./native-src/linux.IA32/build.log

```text
X86 -DHYPORT_LIBRARY_DEFINE -I../include     -c -o hyvmem.o hyvmem.c
hyvmem.c: In function `hyvmem_reserve_memory':
hyvmem.c:311: `SHM_HUGETLB' undeclared (first use in this function)
hyvmem.c:311: (Each undeclared identifier is reported only once
hyvmem.c:311: for each function it appears in.)
make[1]: Leaving directory `/home/jfclere/harmony/enhanced/trunk/sandbox/contribs/ibm_core/native-src/linux.IA32/port'
make[1]: *** [hyvmem.o] Error 1
make: *** [_port] Error 2
```

Oops, my system does not support SHM_HUGETLB (libc is too old!).

To fix, use the patch:

```text
Index: native-src/linux.IA32/port/hyvmem.c
===================================================================
--- native-src/linux.IA32/port/hyvmem.c (revision 351912)
+++ native-src/linux.IA32/port/hyvmem.c (working copy)
@@ -35,6 +35,10 @@
 #include
 #include

+#if !defined(SHM_HUGETLB)
+#define SHM_HUGETLB     04000   /* segment will use huge TLB pages */
+#endif
+
 #if !defined(MAP_FAILED)
 #define MAP_FAILED -1
 #endif
```
