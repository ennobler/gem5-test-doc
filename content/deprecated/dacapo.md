---
title: "DaCapo"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

This page describes how to get the Oracle-Sun JRE version 7 working on a
disk image for use with ARM gem5. This will allow you to use the DaCapo
benchmarks, which are on the [Ubuntu image for gem5
ARM](http://www.gem5.org/dist/current/arm/arm-system-dacapo-2011-08.tgz)
in the `/benchmarks/dacapo` directory.

## Installing Oracle-Sun Java on an Ubuntu Image for ARM gem5

Download the standard edition(SE) of Java 7; the SE version is the free
version of Java available from Oracle. Make sure to get the headless,
SoftFP version of the JRE for ARMv6/7 from
[here](http://www.oracle.com/technetwork/java/embedded/embedded-se/downloads/index.html).
You may need to register with Oracle before being allowed to download.
Once you have this, untar
it.

## Setup the Java Directory

`  `
`//mount the image`
`sudo mkdir /mnt/ubuntu-gem5`
`sudo mount -o loop,offset=32256 arm-ubuntu-natty-headless-java.img /mnt/ubuntu-gem5`
`  `
`//setup the Java install directory and copy the Java directory over to your image`
`cd /mnt/ubuntu-gem5/usr/lib`
`sudo mkdir jvm`
`cd ./jvm`
`sudo cp -a /path_to_jre/jre_dir .`
`sudo ln -s ./jre_dir java-7-sun`

## Install Java

`  `
`cd /mnt/ubuntu-gem5`
`sudo mount -o bind /proc ./proc`
`sudo mount -o bind /dev ./dev`
`sudo mount -o bind /sys ./sys`
`sudo chroot .`
`exec ./bin/bash`
`  `
`//now install Java`
`cd /usr/lib/jvm`
`update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-7-sun/bin/java 1`
`update-alternatives --install /usr/bin/keytool keytool /usr/lib/jvm/java-7-sun/bin/keytool 1`
`  `
`//setup .jinfo file`
`echo "name=java-7-sun" > .java-7-sun.jinfo`
`echo "alias=java-7-sun" >> .java-7-sun.jinfo`
`echo "priority=1" >> .java-7-sun.jinfo`
`echo "section=non-free" >> .java-7-sun.jinfo`
`echo "jre java /usr/lib/jvm/java-7-sun/bin/java" >> .java-7-sun.jinfo`
`echo "jre keytool /usr/lib/jvm/java-7-sun/bin/keytool" >> .java-7-sun.jinfo`
`  `
`//update java alternatives`
`update-java-alternatives -s java-7-sun`

Once you have unmounted the image, you should be able to run Java with
this disk image. Boot this image on gem5 and, from the terminal, type
`java -version` to verify the Java version on your image. E.g., the
version tested on gem5
is:

`  `
`root@gem5sim:~# java -version`
`java version "1.7.0_04-ea"`
`Java(TM) SE Runtime Environment for Embedded (build 1.7.0_04-ea-b20, headless)`
`Java HotSpot(TM) Embedded Client VM (build 23.0-b21, mixed mode)`
