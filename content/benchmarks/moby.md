---
title: "Moby"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

{{% notice info %}}
AsimBench is renamed as **Moby**\!
{{% /notice %}}


This page describes all the necessary files and modifications to run
Android and [Moby](http://asg.ict.ac.cn/projects/moby) on gem5 using the
ARM ISA. [BBench-gem5](BBench-gem5 "wikilink") introduces more detailed
information about how to set Android running on gem5, and how to build
Android File System and Kernel.

## Android Full-System Files

**These files contain everything you need to get Android, and Moby, up
and running on gem5.**

All these files can be downloaded from
[here](https://bitbucket.org/yongbing_huang/asimbench).

  - **vmlinux.smp.ics.arm.asimbench.2.6.35** -- Pre-compiled Android
    kernel. Compared to the 2.6.35 version Android kernel, this kernel
    only modifies the screen size of simulated mobile device, which is
    specified to 800 x 480.

<!-- end list -->

  - **ARMv7a-ICS-Android.SMP.Asimbench.tar.gz** -- Disk image with a
    pre-compiled ICS file-system. This disk image contains all the files
    for all the applications included in AsimBench. The related files
    are mainly stored in the directories *data/app*, *data/data* of the
    image.

<!-- end list -->

  - **sdcard-1g.tar.gz** -- Disk image with applications' data. This
    disk image simulates the SDcard partition of mobile devices.

<!-- end list -->

  - **{k9mail, adobe, sinaweibo, ...}.rcS** -- Execution scripts for
    each application in gem5.

## Running Moby on Android with gem5

  - Both the disk image and the SDcard image should be loaded by gem5,
    and the SDcard image is automatically mounted under the directory
    */mnt/sdcard* of Android file system by
default.

` Here is an example of loading two disk images in gem5. In the file `*`config/common/FSConfig.py`*
`   def make ArmSystem(...)`
`       ...`
`       self.cf0 = CowIdeDisk(driveID='master')`
`       self.cf2 = CowIdeDisk(driveID='master')`
`       self.cf0.childImage(mdesc.disk())`
`       self.cf2.childImage(disk("sdcard-1g.img"))`
`       # default to an IDE controller rather than a CF one`
`       # assuming we've got one`
`       try:`
`           self.realview.ide.disks = [self.cf0, self.cf2]`
`       except:`
`           self.realview.cf_ctrl.disks = [self.cf0, self.cf2]`
`       ...`

## Publications

If you use Moby (or AsimBench) in your work please cite our paper which
will be appeared in ISPASS'2014.

Yongbing Huang, Zhongbin Zha, Mingyu Chen, Lixin Zhang. Moby: A Mobile
Benchmark Suite for Architectural Simulators. *IEEE International
Symposium on Performance Analysis of Systems and Software (ISPASS)*,
Monterey, CA, March 2014.
