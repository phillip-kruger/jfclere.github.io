---
title: "Build and testing harmony jchevm JVM"
slug: build-and-testing-harmony-jchevm-jvm
date: 2006-02-26 11:40 +0100
year: "2006"
description: "Building the Apache Harmony jchevm JVM on Debian with Classpath, and the glibj.zip boot classpath trick needed to actually run a class with it."
---

To build it on debian:

1. Install classpath, autoconf, automake, libtool, libpopt, libz and Classpath.
2. Run `autogen.sh --with-classpath=/usr`
3. `configure`
4. `make`

To test it write a small java program and start it using the JVM:

```text
jfclere@jfcexpert:~/harmony/enhanced/jchevm$ jc/jc toto
java.lang.NoClassDefFoundError: org.dellroad.jc.vm.DebugThread
       at java.lang.VMThrowable.fillInStackTrace(Native Method)
       at java.lang.Throwable.fillInStackTrace(Throwable.java:498)
       at java.lang.Throwable.<init>(Throwable.java:159)
       at java.lang.Error.<init>(Error.java:81)
       at java.lang.LinkageError.<init>(LinkageError.java:72)
       at java.lang.NoClassDefFoundError.<init>(NoClassDefFoundError.java:74)
jc: failed to create VM
```

Do the following (toto.class is a small java program):

```bash
jc/jc -b /usr/share/classpath/glibj.zip:./java/jc.zip toto
```
