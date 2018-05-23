---
title: "Compiling Workloads"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

## Cross Compilers

A cross compiler is a compiler set up to run on one ISA but generate
binaries which run on another. You may need one if you intend to
simulate a system which uses a particular ISA, Alpha for instance, but
don't have access to actual Alpha hardware. There are various sources
for cross compilers, listed here in roughly recommended order:

1.  Some architectures have professionally build cross-compilers
    available from Code Sourcery. These are updated frequently and a
    good starting point:
    [ARM](http://www.codesourcery.com/sgpp/lite/arm/portal/subscription3057),
    [MIPS](http://www.codesourcery.com/sgpp/lite/mips/portal/subscription3130)
2.  You can build your own cross compiler using crosstools-ng: Download
    it from
    [here](http://ymorin.is-a-geek.org/dokuwiki/projects/crosstool#download)
    and follow the instructions on that page.
3.  We have some available on our [Download](Download "wikilink") page.

Alternatively, you can use QEMU and a disk image to run the desired ISA
in emulation. See

{{\#ev:youtube|Oh3NK12fnbg|400|center|A youtube video of working with
image files using qemu on Ubuntu 12.04 64bit. Video resolution can be
set to 1080}}

Note: The video uses an Ubuntu Natty image. Since Natty is now quite
old, you will need to update your /etc/apt/sources.list file, on the
mounted image. Use, e.g., `sudo vi /etc/apt/sources.list`, and replace
<http://old-releases.ubuntu.com/ubuntu/> in place of
<http://ports.ubuntu.com/>. Also, for things to work as show in the
video, you will need to have have sshd running on the host machine.

## Syscall Emulation Mode

SE mode workloads must be statically linked. Gem5 doesn't yet support
dynamic linking, so all benchmarks run in SE mode must be statically
linked in order to be started properly by the simulator. In FS mode the
simulated operating system takes care of any dynamic linking, so this
restriction only applies to SE mode. If you're using gcc you can pass it
the --static option.

