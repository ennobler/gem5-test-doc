---
title: "BBench"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

Note that the ICS images are deprecated. Please see the instructions for
[running Android on gem5](Android_KitKat "wikilink") and [how to use
workload automation](WA-gem5 "wikilink").

## Tips for Making Your Disk Image gem5 Friendly

#### Speeding Up the Boot Process

When a fresh Android image is booted it generates a lot of files and
does a lot of JIT compiling; this can slow down the boot process
significantly. gem5 uses a copy-on-write (COW) layer between the
simulator and the actual disk image, because of this COW layer none of
the changes are stored to the disk image. To make these changes
permanent, and avoid having to repeat them in the future, you can remove
the COW layer during the first boot of an image. This will significantly
speedup future runs. To do so make the following changes to
`configs/common/FSConfig.py`:

<code>

`   class RawIdeDisk(IdeDisk):`
`       image = RawDiskImage(read_only=False)`
`       def childImage(self, ci):`
`           self.image.image_file=ci`

</code>

Then, inside of `makeArmSystem()`, change from:

<code>

`   self.c0 = CowIdeDisk(driveID='master')`

</code>

to

<code>

`   self.c0 = RawIdeDisk(driveID='master')`

</code>

Be careful when doing this. Any changes made to the disk image will be
permanent when using the raw ide disk. To ensure that all the changes
are written to the disk image properly you can use the `sync` and `halt`
Linux commands. These are not available on Android so it is recommended
that you use something like [BusyBox](http://busybox.net). Once the
image has finished booting and has settled, you can run `busybox sync`
and `busybox halt -f` from a gem5 terminal to write all changes to the
disk and halt it properly. You will likely get a panic when the
simulator exits regarding an unrecognized byte, however, this doesn't
seem to cause any problems. Remember to re-enable the COW layer once
you've finished setting up your disk image.

#### BusyBox

[BusyBox](http://busybox.net) is a useful tool providing many common
Linux utilities for embedded systems. To build busybox, download the
source and compile statically using the following commands:

1.  `make CROSS_COMPILE=arm-none-linux-gnueabi- defconfig`
2.  `LDFLAGS="--static" make CROSS_COMPILE=arm-none-linux-gnueabi- -jn`

Place the busybox binary into the `/sbin/` directory of your Android
file-system. Run `busybox --help` to see a full list of the available
utilities.

#### Android Init Process

To have your benchmarks start automatically you will likely need to
modify the `init.rc` script located in the file-system's root directory.
You can see the changes made to the `init.rc` script for BBench by
looking at the `init.rc` script in the images we distribute with BBench,
this may be a useful template. Another useful resource for understanding
the init process is located
[here](http://www.kandroid.org/online-pdk/guide/bring_up.html).

#### m5 Utility

You will likely need the `m5` utility, whose source is located in
`util/m5/`, to do anything useful with your disk image. Build this
(statically) and place it in the `/sbin/` directory of your Android
file-system.

## Publications

If you use BBench in your work please cite our [IISWC 2011
paper](http://dx.doi.org/10.1109/IISWC.2011.6114205):

A. Gutierrez, R.G. Dreslinski, T.F. Wenisch, T. Mudge, A. Saidi, C.
Emmons, and N. Paver. Full-System Analysis and Characterization of
Interactive Smartphone Applications. *IEEE International Symposium on
Workload Characterization*, pages 81-90, Austin, TX, November 2011.

