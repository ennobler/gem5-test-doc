---
title: "Android Marshmallow"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

{{% notice info %}}
These instructions expect that the user will be using ARM support of gem5
{{% /notice %}}

## Overview

To successfully run Android in gem5, an image, a compatible kernel and a
device tree blob .dtb) file configured for the simulator are necessary.
This guide shows how to build Android Marshmallow 32bit version using a
3.14 kernel with Mali support. An extra section will be added in the
future on how to build the 4.4 kernel with Mali.

## Pre-requisites

This guide assumes a 64-bit system running 14.04 LTS Ubuntu. Before
starting it is important first to set up our system correctly. To do
this the following packages need to be installed through
shell.

```
 Tip: Always check for the up-to-date prerequisites at the Android build page.
```

Update and install all the dependencies. This can be done with the
following commands:

<code>sudo apt-get update

sudo apt-get install openjdk-7-jdk git-core gnupg flex bison gperf
build-essential zip curl zlib1g-dev gcc-multilib g++-multilib
libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev
ccache libgl1-mesa-dev libxml2-utils xsltproc unzip</code>

Also, make sure to have repo correctly installed (instructions
[here](https://source.android.com/source/downloading.html#installing-repo)).

Ensure that the default is OpenJDK 1.7

`javac -version`

To cross-compile the kernel (32bit) and for the device tree we will need
the following packages to be installed:

`sudo apt-get install gcc-arm-linux-gnueabihf device-tree-compiler`

Before getting started, as a final step make sure to have the gem5
binaries and busybox for 32-bit ARM.

  - For the gem5 binaries just do the following starting from your gem5
    directory:

<!-- end list -->

  -
    `.cd util/m5`
    `make -f Makefile.arm`
    `cd ../term`
    `make`
    `cd ../../system/arm/simple_bootloader/`
    `make`

<!-- end list -->

  - For busybox you can find the guide
    [here](http://wiki.beyondlogic.org/index.php?title=Cross_Compiling_BusyBox_for_ARM)

## Building Android

We build Android Marshmallow using an AOSP running build based on the
release for the Pixel C. The AOSP provides [other
builds](https://source.android.com/source/build-numbers.html#source-code-tags-and-builds),
which are untested with this guide.

    Tip: Synching with repo will take a long time.

    Use -jN to speed up the make process, where N number of parallel jobs to run.

  - Make a directory and pull the android repository. Type in the
    following:

<!-- end list -->

  -
    `mkdir android`
    `cd android`
    `repo init --depth=1 -u
    `<https://android.googlesource.com/platform/manifest>` -b
    android-6.0.1_r63`
    `repo sync -c -jN`

<!-- end list -->

  - Before you start the AOSP build, you will need to make one change to
    the build system to enable building libion.so, which is used by the
    Mali driver. Edit the file `aosp/system/core/libion/Android.mk` to
    change `LOCAL_MODULE_TAGS` for libion from 'optional' to 'debug'.
    Here is the output of `repo
diff`:

` --- a/system/core/libion/Android.mk`
` +++ b/system/core/libion/Android.mk`
` @@ -3,7 +3,7 @@ LOCAL_PATH := $(call my-dir)`
` include $(CLEAR_VARS)`
` LOCAL_SRC_FILES := ion.c`
` LOCAL_MODULE := libion`
` -LOCAL_MODULE_TAGS := optional`
` +LOCAL_MODULE_TAGS := debug`
` LOCAL_SHARED_LIBRARIES := liblog`
` LOCAL_C_INCLUDES := $(LOCAL_PATH)/include $(LOCAL_PATH)/kernel-headers`
` LOCAL_EXPORT_C_INCLUDE_DIRS := $(LOCAL_PATH)/include`
` $(LOCAL_PATH)/kernel-headers`

  - Source the environment setup and build Android.
    <span style="color:orange"> Make sure to do this in a bash shell.
    </span>

> Tip: For root access and "debuggability" \[sic\] we choose userdebug.
> Build can be done in different modes as seen
> [here](https://source.android.com/source/building.html#choose-a-target)

    Tip: Making Android will take a long time.

    Use -jN to speed up the make process, where N number of parallel jobs to run.

  -
    `source build/envsetup.sh`

<!-- end list -->

  -
    `lunch aosp_arm-userdebug`

<!-- end list -->

  -
    `make -jN`

## Creating an Android image

After a successful build, we create an image of Android and add the init
files and binaries that configure the system for gem5.

  - Create an empty image to flash the Android build and attach the
    image to a loopback device. The commands below create a 3GB
    image.

<!-- end list -->

    Tip: If you want to add applications or data, make the image large enough to fit the build and anything else that is meant to be written into it.

  -
    `dd if=/dev/zero of=myimage.img bs=1M count=2560`

<!-- end list -->

  -
    `sudo losetup /dev/loop0 myimage.img`

<!-- end list -->

  - Make three partitions AndroidRoot (1.5GB), AndroidData (1GB),
    AndroidCache (512MB):

<!-- end list -->

  -
    Partition the device:

<!-- end list -->

  -
    `sudo fdisk /dev/loop0`

<!-- end list -->

  -
    Update the partition table:

<!-- end list -->

  -
    `sudo partprobe /dev/loop0`

<!-- end list -->

  -
    Name the partitions / Define filesystem as ext4:

<!-- end list -->

  -
    `sudo mkfs.ext4 -L AndroidRoot /dev/loop0p1`

<!-- end list -->

  -
    `sudo mkfs.ext4 -L AndroidData /dev/loop0p2`

<!-- end list -->

  -
    `sudo mkfs.ext4 -L AndroidCache /dev/loop0p3`

<!-- end list -->

  - Mount the Root partition to a directory:

<!-- end list -->

  -
    `sudo mkdir -p /mnt/androidRoot`

<!-- end list -->

  -
    `sudo mount /dev/loop0p1 /mnt/androidRoot`

<!-- end list -->

  - Load the build to the partition:

<!-- end list -->

  -
    `cd /mnt/androidRoot`

<!-- end list -->

  -
    `sudo zcat
    `<path/to/build/android>`/out/target/product/generic/ramdisk.img |
    sudo cpio -i`

<!-- end list -->

  -
    `sudo mkdir cache`

<!-- end list -->

  -
    `sudo mkdir /mnt/tmp`

<!-- end list -->

  -
    `sudo mount -oro,loop
    `<path/to/build/android>`/out/target/product/generic/system.img
    /mnt/tmp`

<!-- end list -->

  -
    `sudo cp -a /mnt/tmp/* system/`

<!-- end list -->

  -
    `sudo umount /mnt/tmp`

<!-- end list -->

  - Download and unpack the
    [overlays](http://www.gem5.org/dist/current/arm/kitkat-overlay.tar.bz2)
    that are necessary from the [gem5 Android KitKat
    page](Android_KitKat#Resources "wikilink") and make the following
    changes to the init.gem5.rc file. Here is the output of `repo
diff`:

` --- /kitkat_overlay/init.gem5.rc`
` +++ /m_overlay/init.gem5.rc`
` @@ -1,21 +1,13 @@`
` +`
`  on early-init`
`      mount debugfs debugfs /sys/kernel/debug`
` `
`  on init`
` -    export LD_LIBRARY_PATH ${LD_LIBRARY_PATH}:/vendor/lib/egl`
` -`
` -    # See storage config details at `<http://source.android.com/tech/storage/>
` -    mkdir /mnt/media_rw/sdcard 0700 media_rw media_rw`
` -    mkdir /storage/sdcard 0700 root root`
` +    # Support legacy paths`
` +    symlink /sdcard /mnt/sdcard`
`      chmod 0666 /dev/mali0`
`      chmod 0666 /dev/ion`
` -`
` -    export EXTERNAL_STORAGE /storage/sdcard`
` -`
` -    # Support legacy paths`
` -    symlink /storage/sdcard /sdcard`
` -    symlink /storage/sdcard /mnt/sdcard`
` `
`  on fs`
`      mount_all /fstab.gem5`
` @@ -60,7 +52,6 @@`
`      group root`
`      oneshot`
` `
` -# fusewrapped external sdcard daemon running as media_rw (1023)`
` -service fuse_sdcard /system/bin/sdcard -u 1023 -g 1023 -d`
` /mnt/media_rw/sdcard /storage/sdcard`
` +service fingerprintd /system/bin/fingerprintd`
`      class late_start`
` -    disabled`
` +    user system`

  - Add the Android overlays and configure their permissions:

<!-- end list -->

  -
    `sudo cp -r `<path/to/android/overlays>`/* /mnt/androidRoot/`

<!-- end list -->

  -
    `sudo chmod ug+x /mnt/androidRoot/init.gem5.rc`

<!-- end list -->

  -
    `/mnt/androidRoot/gem5/postboot.sh`

<!-- end list -->

  - Add the m5 and busybox binaries under the sbin directory and make
    them executable:

<!-- end list -->

  -
    `sudo cp `<path/to/gem5>`/util/m5/m5 /mnt/androidRoot/sbin`

<!-- end list -->

  -
    `sudo cp `<path/to/busybox>`/busybox /mnt/androidRoot/sbin`

<!-- end list -->

  -
    `sudo chmod a+x /mnt/androidRoot/sbin/busybox
    /mnt/androidRoot/sbin/m5`

<!-- end list -->

  - Make the directories readable and searchable:

<!-- end list -->

  -
    `sudo chmod a+rx /mnt/androidRoot/sbin/ /mnt/androidRoot/gem5/`

<!-- end list -->

  - Remove the boot animation:

<!-- end list -->

  -
    `sudo rm /mnt/androidRoot/system/bin/bootanimation`

<!-- end list -->

  - Download and unpack the Mali drivers (for gem5 Android 4.4) from
    [here](http://malideveloper.arm.com/resources/drivers/arm-mali-midgard-gpu-user-space-drivers/).
    Add the drivers and adjust their permissions:

<!-- end list -->

  -
    Make the directories for the drivers and copy them:

<!-- end list -->

  -
    `sudo mkdir -p /mnt/androidRoot/system/vendor/lib/egl`

<!-- end list -->

  -
    `sudo mkdir -p /mnt/androidRoot/system/vendor/lib/hw`

<!-- end list -->

  -
    `sudo cp `<path/to/userspace/Mali/drivers>`/lib/egl/libGLES_mali.so
    /mnt/androidRoot/system/vendor/lib/egl`

<!-- end list -->

  -
    `sudo cp
    `<path/to/userspace/Mali/drivers>`/lib/hw/gralloc.default.so
    /mnt/androidRoot/system/vendor/lib/hw`

<!-- end list -->

  -
    Change the permissions

<!-- end list -->

  -
    `sudo chmod 0755 /mnt/androidRoot/system/vendor/lib/hw`

<!-- end list -->

  -
    `sudo chmod 0755 /mnt/androidRoot/system/vendor/lib/egl`

<!-- end list -->

  -
    `sudo chmod 0644
    /mnt/androidRoot/system/vendor/lib/egl/libGLES_mali.so`

<!-- end list -->

  -
    `sudo chmod 0644
    /mnt/androidRoot/system/vendor/lib/hw/gralloc.default.so`

<!-- end list -->

  - Unmount and remove loopback device:

<!-- end list -->

  -
    `cd /..`

<!-- end list -->

  -
    `sudo umount /mnt/androidRoot`

<!-- end list -->

  -
    `sudo losetup -d /dev/loop0`

## Building the Kernel (3.14)

After successfully setting up the image, a compatible kernel needs to be
built and a .dtb file generated.

  - Clone the repository containing the gem5 specific kernel:

<!-- end list -->

  -
    `git clone -b ll_20140416.0-gem5
    `<https://github.com/gem5/linux-arm-gem5.git>

<!-- end list -->

  - Make the following changes to the kernel gem5 config file at
    <path/to/kernel/repo>/arch/arm/configs/vexpress_gem5_defconfig.
    Here is the output of `repo diff`:

` --- a/arch/arm/configs/vexpress_gem5_defconfig`
` +++ b/arch/arm/configs/vexpress_gem5_defconfig`
` @@ -200,4 +200,15 @@ CONFIG_EARLY_PRINTK=y`
` CONFIG_DEBUG_PREEMPT=n`
` # CONFIG_CRYPTO_ANSI_CPRNG is not set`
` # CONFIG_CRYPTO_HW is not set`
` +CONFIG_MALI_MIDGARD=y`
` +CONFIG_MALI_MIDGARD_DEBUG_SYS=y`
` +CONFIG_ION=y`
` +CONFIG_ION_DUMMY=y`
` CONFIG_BINARY_PRINTF=y`
` +CONFIG_NET_9P=y`
` +CONFIG_NET_9P_VIRTIO=y`
` +CONFIG_9P_FS=y`
` +CONFIG_9P_FS_POSIX_ACL=y`
` +CONFIG_9P_FS_SECURITY=y`
` +CONFIG_VIRTIO_BLK=y`
` +CONFIG_VMSPLIT_3G=y`
` +CONFIG_DNOTIFY=y`
` +CONFIG_FUSE_FS=y`

  - For the device tree, add the Mali GPU device and increase the memory
    to 1.8GB. Do this with the following changes at
    <path/to/kernel/repo>/arch/arm/boot/dts/vexpress-v2p-ca15-tc1-gem5.dts.
    Here is the output of `repo diff`:

` --- a/arch/arm/boot/dts/vexpress-v2p-ca15-tc1-gem5.dts`
` +++ b/arch/arm/boot/dts/vexpress-v2p-ca15-tc1-gem5.dts`
` @@ -45,7 +45,7 @@`
` `
`          memory@80000000 {`
`                  device_type = "memory";`
` -                reg = <0 0x80000000 0 0x40000000>;`
` +                reg = <0 0x80000000 0 0x74000000>;`
`          };`
` `
`         hdlcd@2b000000 {`
` @@ -59,6 +59,14 @@`
` //                mode = "3840x2160MR-16@60"; // UHD4K mode string`
`                   framebuffer = <0 0x8f000000 0 0x01000000>;`
`           };`
` +`
` +    gpu@0x2d000000 {`
` +        compatible = "arm,mali-midgard";`
` +        reg = <0 0x2b400000 0 0x4000>;`
` +        interrupts = <0 86 4>, <0 87 4>, <0 88 4>;`
` +        interrupt-names = "JOB", "MMU", "GPU";`
` +    };`
` +`
` /*`
`         memory-controller@2b0a0000 {`
`                   compatible = "arm,pl341", "arm,primecell";`

  - Download and unpack the userspace matching Mali kernel drivers for
    gem5 from
    [here](http://malideveloper.arm.com/resources/drivers/open-source-mali-midgard-gpu-kernel-drivers/).
    Copy them to the gpu driver directory:

<!-- end list -->

  -
    `cp -r
    `<path/to/kernelspace/Mali/drivers>`/driver/product/kernel/drivers/gpu/arm/
    drivers/gpu`

<!-- end list -->

  - Change the following in
    <path/to/kernelspace/Mali/drivers>/drivers/video/Kconfig and
    <path/to/kernelspace/Mali/drivers>/drivers/gpu/Makefile:

Here is the output of the Kconfig `repo diff`:

` --- a/drivers/video/Kconfig`
` +++ b/drivers/video/Kconfig`
` @@ -23,6 +23,8 @@ source "drivers/gpu/host1x/Kconfig"`
` `
` source "drivers/gpu/drm/Kconfig"`
` `
` +source "drivers/gpu/arm/Kconfig"`
` +`
`  config VGASTATE`
`         tristate`
`         default n`

Here is the output of the drivers/gpu/Makefile `repo diff`:

` --- a/drivers/gpu/Makefile`
` +++ b/drivers/gpu/Makefile`
` @@ -1,2 +1,2 @@`
` -obj-y                += drm/ vga/`
` +obj-y                += drm/ vga/ arm/`

  - Finally, build the kernel and the .dtb file:

<!-- end list -->

  -
    Build the
    kernel

<!-- end list -->

    Tip: Use -jN to speed up the make process, where N number of parallel jobs to run.

  -
    `make CROSS_COMPILE=arm-linux-gnueabihf- ARCH=arm
    vexpress_gem5_defconfig`

<!-- end list -->

  -
    `make CROSS_COMPILE=arm-linux-gnueabihf- ARCH=arm vmlinux -jN`

<!-- end list -->

  -
    Create the .dtb file:

<!-- end list -->

  -
    `dtc -I dts -O dtb arch/arm/boot/dts/vexpress-v2p-ca15-tc1-gem5.dts
    > vexpress-v2p-ca15-tc1-gem5.dtb`

## Testing the build

Make the following changes to example/fs.py. Here is the output `repo
diff`:

` --- a/configs/example/fs.py Thu Jun 02 20:34:39 2016 +0100`
` +++ b/configs/example/fs.py Fri Jun 10 15:37:29 2016 -0700`
` @@ -144,6 +144,13 @@`
`      if is_kvm_cpu(TestCPUClass) or is_kvm_cpu(FutureClass):`
`          test_sys.vm = KvmVM()`
` `
` +    test_sys.gpu = NoMaliGpu(`
` +        gpu_type="T760",`
` +        ver_maj=0, ver_min=0, ver_status=1,`
` +        int_job=118, int_mmu=119, int_gpu=120,`
` +        pio_addr=0x2b400000,`
` +        pio=test_sys.membus.master)`
` +`
`     if options.ruby:`
`         # Check for timing mode because ruby does not support atomic accesses`
`         if not (options.cpu_type == "detailed" or options.cpu_type == "timing"):`

And the changes to FS config to either enable or disable software
rendering.

` --- a/configs/common/FSConfig.py Thu Jun 02 20:34:39 2016 +0100`
` +++ b/configs/common/FSConfig.py Thu Jun 16 10:23:44 2016 -0700`
` @@ -345,7 +345,7 @@`
` `
`            # release-specific tweaks`
`            if 'kitkat' in mdesc.os_type():`
` -                cmdline += " androidboot.hardware=gem5 qemu=1 qemu.gles=0 " + \`
` +                cmdline += " androidboot.hardware=gem5 qemu=1 qemu.gles=1 " + \`
`                           "android.bootanim=0"`
` `
`        self.boot_osflags = fillInCmdline(mdesc, cmdline`

Set the following M5_PATH:

`M5_PATH=. build/ARM/gem5.opt configs/example/fs.py --cpu-type=atomic
--mem-type=SimpleMemory --os-type=android-kitkat
--disk-image=myimage.img --machine-type=VExpress_EMM
--dtb-filename=vexpress-v2p-ca15-tc1-gem5.dtb -n 1 --mem-size=1800MB`

