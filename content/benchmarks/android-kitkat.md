---
title: "Android KitKat"
date: 2018-05-13T18:51:37-04:00
draft: false
---

## Building Android KitKat for gem5

### Overview

The easiest way to build Android for gem5 is to base the system on a the
emulated Goldfish device. The main difference between the actual
qemu-based Goldfish model and gem5 is a small number of gem5-specific
configuration files. These files are mainly related to differences in
block device naming and scripts to start use gem5's pseudo-op interface
to start experiments.

### Build Instructions

Follow the directions on [Android Open
Source](http://source.android.com/source/building.html) to download and
build Android. Make sure to create a local mirror (see the [download
page](http://source.android.com/source/downloading.html)) to speed up
things if you ever need to create a new working directory. The following
instructions are based on our experience from bringing up Android
4.4.4r2 (KitKat).

Make sure to quickly read the AOSP build instructions. In particular the
section about Java dependencies. KitKat and earlier all depend on
particular Sun/Oracle JDK versions. Lollipop and newer seem to be able
to build with OpenJDK.

If you are building on a dedicated build machine or virtual machine, I'd
recommend making sure OpenJDK is not installed at all on your machine
for building KitKat. Sometimes having both versions can cause issues
where some tools are invoked from OpenJDK while others get invoked from
the Oracle JDK.

Either way, make sure that the Oracle bin directory is first in your
PATH (and make sure it doesn't have a trailing slash):

`export PATH=/path/to/java-6-jdk/bin:$PATH`

When setting up your build directories, I'd suggest a structure as
follows:

  - `/work/android` (or some other directory with plenty of free space)
      - `.../repos/aosp-mirro` - Mirror of the upstream Android
        repository
      - `.../gem5kitkat/`
          - `.../aosp` - Clone of the Android repository

Before you start the AOSP build, you will need to make one change to the
build system to enable building libion.so, which is used by the Mali
driver. Edit the file `aosp/system/core/libion/Android.mk` to change
`LOCAL_MODULE_TAGS` for libion from 'optional' to 'debug'. Here is the
output of `repo diff`:

` project system/core/`
` diff --git a/libion/Android.mk b/libion/Android.mk`
` index 5121fee..1a0af0e 100644`
` --- a/libion/Android.mk`
` +++ b/libion/Android.mk`
` @@ -3,7 +3,7 @@ LOCAL_PATH:= $(call my-dir)`
`  include $(CLEAR_VARS)`
`  LOCAL_SRC_FILES := ion.c`
`  LOCAL_MODULE := libion`
` -LOCAL_MODULE_TAGS := optional`
` +LOCAL_MODULE_TAGS := debug`
`  LOCAL_SHARED_LIBRARIES := liblog`
`  include $(BUILD_SHARED_LIBRARY)`

Build a vanilla AOSP KitKat distribution using the following command:

`. build/envsetup.sh`
`lunch aosp_arm-userdebug`
`make -j8`

This builds a plain Android for the Goldfish device (an Android specific
qemu version). We are going to use this as a base for our gem5
distribution.

Useful
commands:

|            |                                                                        |
| ---------- | ---------------------------------------------------------------------- |
| `hmm`      | Show a list of build system commands                                   |
| `mm`       | Build the Android module in the current directory                      |
| `mma`      | Build the Android module in the current directory and its dependencies |
| `emulator` | Launch Android in qemu                                                 |

The `mm` command is especially useful since just running `make` in a
directory with an existing build of Android (i.e., make doesn't need to
build anything) can take several minutes.

### Preparing a filesystem for gem5

First, create an empty disk (this example creates a 2GiB image) image:

`dd if=/dev/zero of=myimage.img bs=1M count=2048`

As root, hook up the disk image to a loopback device (the following
assumes /dev/loop0 is free).

`losetup /dev/loop0 myimage.img`

Using fdisk, create the following partitions:

| Part. No | Usage  | Approximate Size |
| -------- | ------ | ---------------- |
| 1        | /      | 500MB            |
| 2        | /data  | 1GB              |
| 3        | /cache | 500MB            |

Use common sense when setting up the partitions. The root partition will
contain both Android's root file system and the system file system and
should be big enough for both of them. The data partition will contain
any apps that are not a part of the system (i.e., anything you install).

As root, tell the kernel about the partitions and format them to ext4:

`partprobe /dev/loop0`
`mkfs.ext4 -L AndroidRoot /dev/loop0p1`
`mkfs.ext4 -L AndroidData /dev/loop0p2`
`mkfs.ext4 -L AndroidCache /dev/loop0p3`

Mount the filesystem and extract the root file system:

`mkdir -p /mnt/android`
`mount /dev/loop0p1 /mnt/android`
`cd /mnt/android`
`zcat AOSP/out/target/product/generic/ramdisk.img  | cpio -i`
`mkdir cache`

`mkdir -p /mnt/tmp`
`mount -oro,loop AOSP/out/target/product/generic/system.img /mnt/tmp`
`cp -a /mnt/tmp/* system/`

At this point, the new disk image is a (mostly) vanilla Goldfish image.
Add the gem5 pixie dust by copying all the files in the attached
tar-ball into the new root file system and adding an `m5` binary (see
`util/m5` in your gem5 work directory) to `/sbin`. This directory
contains a gem5-specific init.rc that is based on the original Goldfish
device, with additional tweaks. Specifically, it runs
`/gem5/postboot.sh` when Android has booted. This script is responsible
for disabling the screen lock and downloading and executing a run script
from gem5. Double check to make sure that both the `init.gem5.rc` and
`postboot.sh` file have the executable permission set.

At this point, everything should just work. Unmount everything and
disconnect the loop back device:

`umount /mnt/android`
`losetup -d /dev/loop0`

Android systems generally does a lot of initialization (JIT compilation
etc.) on the first boot. Since gem5 normally mounts the root file system
as CoW and stores the file system differences in memory. To speed up
future experiments, make sure to follow the guide in
[BBench-gem5](BBench-gem5 "wikilink") to make these changes permanent.

### Building the kernel

You will need a recent ARM cross compiler to build the kernel. If you're
using Ubuntu 10.04, install it by running:

`sudo apt-get install gcc-arm-linux-gnueabihf`

Checkout the gem5 kernel from the following git repository:

<git://linux-arm.org/linux-linaro-tracking-gem5.git>

Alternatively, you can use the github mirror of this same repo (See:
<http://permalink.gmane.org/gmane.comp.emulators.m5.devel/27631>):

`git clone `<https://github.com/gem5/linux-arm-gem5.git>

If you are not planning on doing kernel development, you can add the
option `--depth 1` to significantly speed up your clone.

Configure and build the kernel
using:

`make CROSS_COMPILE=arm-linux-gnueabihf- ARCH=arm vexpress_gem5_defconfig`
`make CROSS_COMPILE=arm-linux-gnueabihf- ARCH=arm vmlinux -j8`

Note: if you want to use Workload Automation, this may be a good time to
add the extra kernel options to your .config file. See
[WA-gem5](WA-gem5 "wikilink") for a description and which kernel options
should be enabled.

## Android Runtime Configuration

Simulation-specific kernel boot options
options:

| Kernel Option             | Function                                                                          |
| ------------------------- | --------------------------------------------------------------------------------- |
| qemu=1                    | Enable emulation support                                                          |
| qemu.gles=1               | Don't use software rendering (usually enables GLES pass through)                  |
| androidboot.hardware=NAME | Overrides the device's platform name (used to select configuration files at boot) |
|  |

A simulated system should normally set `qemu=1` on the kernel command
line. This enables software rendering and some emulation specific
services. In order to load the right boot configuration, set
`androidboot.hardware` to `gem5`.

Kernel command line to properties mapping:

| Kernel Option          | Android Property                          |
| ---------------------- | ----------------------------------------- |
| androidboot.\*         | ro.boot.\*                                |
| androidboot.serialno   | ro.serialno                               |
| androidboot.mode       | ro.bootmode                               |
| androidboot.baseband   | ro.baseband                               |
| androidboot.bootloader | ro.bootloader                             |
| androidboot.hardware   | ro.hardware & ro.boot.hardware            |
| \*                     | ro.kernel.\* if running in emulation mode |

\== Emulation Mode Specifics (qemu=1) ==

### OpenGL & Graphics

  - Setting the ro.kernel.qemu property forces libGLES_android to be
    used unless ro.kernel.qemu.gles is also set.

<!-- end list -->

  - qemu.sf.lcd_density instead of ro.sf.lcd_density is used to
    specify display
density.

## Resources

[kitkat-overlay.tar.bz2](http://www.gem5.org/dist/current/arm/kitkat-overlay.tar.bz2)

## Running workloads with Kitkat

To run workloads we advise the use of [Workload
Automation](https://github.com/ARM-software/workload-automation). For
detailed instructions on how to use this see:
[WA-gem5](WA-gem5 "wikilink")

## NoMali

To use a NoMali model in gem5, you need to perform the following steps:

  - Add it to the configuration file
  - Add the user-space drivers to your Android image
  - Add the kernel-side driver to your kernel
  - Inform the kernel of the device by adding it to the device tree

These steps are described in detail in the sections below.

Once you have completed the steps to build the kernel-side driver and
modified the DTB, test the kernel and DTB by starting it in gem5. Ensure
that you get the following message from the kernel:

`mali 2d000000.gpu: Continuing without Mali clock control`
`mali 2d000000.gpu: GPU identified as 0x0750 r0p0 status 1`
`mali 2d000000.gpu: Probed as mali0`

Once the user-space driver has been added, start Android and check the
output of the `logcat` command. You should see something like this when
`surfaceflinger`
starts:

`I/[Gralloc](  885): using (fd=14)`
`I/[Gralloc](  885): id           = hdlcd`
`I/[Gralloc](  885): xres         = 1920 px`
`I/[Gralloc](  885): yres         = 1080 px`
`I/[Gralloc](  885): xres_virtual = 1920 px`
`I/[Gralloc](  885): yres_virtual = 1080 px`
`I/[Gralloc](  885): bpp          = 16`
`I/[Gralloc](  885): r            = 11:5`
`I/[Gralloc](  885): g            =  5:6`
`I/[Gralloc](  885): b            =  0:5`
`I/[Gralloc](  885): width        = 305 mm (159.895081 dpi)`
`I/[Gralloc](  885): height       = 171 mm (160.421051 dpi)`
`I/[Gralloc](  885): refresh rate = 59.28 Hz`
`E/SurfaceFlinger(  885): hwcomposer module not found`
`W/SurfaceFlinger(  885): getting VSYNC period from fb HAL: 16868241`
`W/SurfaceFlinger(  885): no suitable EGLConfig found, trying a simpler query`
`I/SurfaceFlinger(  885): EGL informations:`
`I/SurfaceFlinger(  885): vendor    : Android`
`I/SurfaceFlinger(  885): version   : 1.4 Android META-EGL`
`I/SurfaceFlinger(  885): extensions: EGL_KHR_get_all_proc_addresses EGL_ANDROID_presentation_time EGL_KHR_image EGL_KHR_image_base EGL_KHR_gl_texture_2D_image EGL_KHR_gl_texture_cubemap_image EGL_KHR_gl_renderbuffer_image EGL_KHR_fence_sync EGL_KHR_create_context EGL_EXT_create_context_robustness EGL_ANDROID_image_native_buffer EGL_KHR_wait_sync EGL_ANDROID_recordable `
`I/SurfaceFlinger(  885): Client API: OpenGL_ES`
`I/SurfaceFlinger(  885): EGLSurface: 5-6-5-0, config=0x78f1f4e0`
`I/SurfaceFlinger(  885): OpenGL ES informations:`
`I/SurfaceFlinger(  885): vendor    : ARM`
`I/SurfaceFlinger(  885): renderer  : Mali-T760`
`I/SurfaceFlinger(  885): version   : OpenGL ES 3.1 v1.r8p0-02rel0.785fbb28b57044ab28a90a55a7b06f3f`

### Configuration Changes

The NoMali GPU is not instantiated by default in any of the example
configurations. To instantiate it, add the following code to your
configuration script:

`my_system.gpu = NoMaliGpu(`
`    gpu_type="T760",`
`    ver_maj=0, ver_min=0, ver_status=1,`
`    int_job=114, int_mmu=115, int_gpu=116,`
`    pio_addr=0x2d000000,`
`    pio=membus.master)`

The code above instantiates a NoMali model of a Mali T760 r0p0-1 at
address `0x2d000000`. The GPU type and revision must match the driver.

### Android User Space Drivers

Download the Mali drivers for gem5 from the [ARM Mali Midgard User Space
Drivers](http://malideveloper.arm.com/resources/drivers/arm-mali-midgard-gpu-user-space-drivers/)
section on [MaliDeveloper](http://malideveloper.arm.com). Make sure that
you download a driver that matches your Android version. Make a note of
the driver version number, you'll need a matching kernel-side driver of
the same version.

The downloaded file will contain at least the following files:

  - `lib/egl/libGLES_mali.so`
  - `lib/hw/gralloc.default.so`

Copy the entire `lib` directory into `/system/vendor` in the target
system. The resulting disk image should contain (at least) the following
files:

  - `/system/vendor/lib/egl/libGLES_mali.so`
  - `/system/vendor/lib/hw/gralloc.default.so`

Make sure that everyone can access the files (i.e., files have
permission 0644 and directories 0755), or you'll end up with permission
errors when booting the system.

### Kernel Drivers

Download the Mali kernel-side drivers from the [ARM Mali Midgard Kernel
Drivers](http://malideveloper.arm.com/resources/drivers/open-source-mali-midgard-gpu-kernel-drivers/)
section on [MaliDeveloper](http://malideveloper.arm.com). Make sure that
you download a driver version that matches your user space driver
version.

Copy the directory `driver/product/kernel/drivers/gpu/arm` into
`drivers/gpu/` in your target kernel. You now need to wire up the driver
to the kernel's build system. Do that by adding `arm/` to the list of
subdirectories in `drivers/gpu/Makefile`. To add it to the configuration
system, add `source "drivers/gpu/arm/Kconfig"` to
`drivers/video/Kconfig`.

Enable the driver in the kernel configuration (set `MALI_MIDGARD` /
`Device Drivers -> Graphics support -> ARM GPU Configuration -> Mali
Midgard series support`). The default options should work and enable
device tree support. Do *not* set the NoMali option in the kernel.

You will also need to enable the Mali DDK sysfs configuration (set
`MALI_MIDGARD_DEBUG_SYS` / `Device Drivers -> Graphics support -> ARM
GPU Configuration -> Mali Midgard series support -> Enable sysfs for the
Mali Midgard DDK`).

You will also need to enable the kernel config option
`CONFIG_ION_DUMMY=y` / `Device Drivers -> Staging drivers -> Android ->
Android Drivers -> Ion Memory Manager -> Dummy Ion driver`. This will
create the /dev/ion device node that the Mali driver will use.

In short, you need to set following configurations in your `.config`
file.

`CONFIG_MALI_MIDGARD=y`
`CONFIG_MALI_MIDGARD_DEBUG_SYS=y`
`CONFIG_ION=y`
`CONFIG_ION_DUMMY=y`

### Device Tree

Add the following node to your device tree:

`gpu@0x2d000000 {`
`   compatible = "arm,mali-midgard";`
`   reg = <0 0x2d000000 0 0x4000>;`
`   interrupts = <0 82 4>, <0 83 4>, <0 84 4>;`
`   interrupt-names = "JOB", "MMU", "GPU";`
`};`

Note that there is an offset between interrupts in gem5 and in the
device tree. Linux starts counting SPIs from 0, while gem5 starts from
32.

