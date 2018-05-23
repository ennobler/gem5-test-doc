---
title: "Linux-dist"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

This documentation is relevant only for those desiring to use the
full-system aspect of M5. When running in full-system, M5 reads a raw
disk image as the hard disk. This tells you how to compile both the
linux binary and the disk image for full-system simulation.

There is a general 3 step process for doing this:

1.  compile a cross-compiler capable of building alpha binaries.
2.  compile a kernel using this cross-compiler
3.  use linux-dist to build binaries for the M5 disk image, and create
    the image.

## Compiling a Cross Compiler using Crosstool-NG

Go to
[here](http://ymorin.is-a-geek.org/dokuwiki/projects/crosstool#download),
download crosstool-ng and follow the directions listed on the page. This
is a new tool based on Dan Kegel's cross tool and is more up to date.

## Compiling the Kernel

First compile the kernel using the instructions at [Compiling a Linux
Kernel](Compiling_a_Linux_Kernel "wikilink"). Then, update
m5/configs/common/FSConfig.py to use the new kernel. You basically need
to change the part the points to the kernel binary to point to the new
kernel (or just replace the binary file for the old kernel with the new
kernel's
file).

## Compiling benchmarks for the image and creating the image with linux-dist

'''Note that this process is quite deprecated and not well supported. We
suggest that you get your disk images either from the downloads page, or
if that doesn't satisfy your needs, with a Gentoo stage 3 image. While
we're not saying the below will not work, it may require a lot of
massaging on your part. '''

Linux-dist is bootstrapped off of the Pengutronix PTX-dist tool for
building disk images for embedded processors (i.e. need to be smaller).
Be sure you have downloaded the linux-dist.tgz tarball from
[here](http://www.m5sim.org/dist/current/linux-dist.tgz). Untar it, and
there is a standard compile/install procedure.

1.  Configure the package with `./configure
    --prefix=/where/you/want/the/installation`
2.  `make`
3.  `make install`

You will notice that now, in the place where you have designated, there
is now a bin/ and lib/ directory pertaining to linux-dist. Now, you will
need to create your m5 image workspace.

1.  set your path to point to `/where/you/want/the/installation/bin.`
2.  wherever you want your workspace, type: `ptxdist clone m5-alpha
    `<your workspace name>. This will create a workspace directory with
    everything you need to make an image.
3.  cd into that directory, and type: `ptxdist menuconfig`. The default
    ptxconfig ought to be sufficient for everything, but you do need to
    set one value. Find the Image Creation Options menu and ensure that
    the "Path to kernel src" value points to your linux directory that
    you got from us.
4.  type: `ptxdist toolchain /path/to/your/toolchain/bin` (e.g.
    `/opt/crosstool/gcc-3.4.3-glibc-2.3.5/alpha-unknown-linux-gnu/bin`).
    This ensures that in building all of the binaries you are using the
    appropriate toolchain.
5.  type: `ptxdist menuconfig`. Note that you will have to go to the
    `Image Creation Options` option and set the path to the kernel code
    that you will be using for your headers and iscsi benchmark module
    compilation.
6.  type: `ptxdist go`. This will compile everything you need to run
    existing M5 full-system benchmarks.
7.  type: ptxdist images. This actually creates the image for you and is
    somewhat interactive. You'll need to tell it how big you want the
    image to be (currently 200MB is sufficient), and where you want to
    put it. Be sure you have sudo privileges.

This is all you need to create the image. To run benchmarks, you merely
need to ensure that m5/configs/common/Benchmarks.py points to the image
you just created, and m5/configs/common/FSConfig.py points to the
vmlinux you created. To run these benchmarks, see [Running M5 in
Full-System Mode](Running_M5_in_Full-System_Mode "wikilink").
