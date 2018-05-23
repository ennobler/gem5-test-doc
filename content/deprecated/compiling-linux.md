---
title: "Compiling Linux"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

If you're interested in ARM Linux kernels, please see this page: [ARM
Linux Kernel](ARM_Linux_Kernel "wikilink")

We supply a repository of patches against the Linux kernel that enables
some M5 features and provides default configuration files that work with
M5. The repository is a Mercurial Queue (MQ) that is intended to be
applied on top of a Linux repository. To compile a kernel using our
patches repository you'll need to get a copy of the linux-2.6 repository
first. Once you have the repository, you'll need to select the version
of Linux you wish to compile, select the appropriate version of patches
for that version of Linux, apply the patches, and then compile.
Step-by-step instructions are provided below.

Note that only certain specific versions of Linux are supported (as of
this writing, the list includes 2.6.13, 2.6.16, 2.6.18, 2.6.22, and
2.6.27). To see a full list, type `hg qguard -l` in the linux-2.6
repository with the m5 patch queue, and look for the version number to
appear on the right side of one of the patches. There is nothing
preventing the patches from working with other kernel versions. However,
no one has gone through the effort of verifying that the patches apply
cleanly to other versions. (If you do make this effort, please let us
know so we can update the patch queue.)

The correct way to use the patches repository is the following (assuming
2.6.27):

    # Get a copy of the linux-2.6 mercurial repository
    hg clone http://www.kernel.org/hg/linux-2.6/

    # Get a copy of our patches to linux
    cd linux-2.6/.hg
    hg clone http://repo.m5sim.org/linux-patches/ patches

    # Return to the root linux directory
    cd ..

    # Update the linux source to the desired version (can take 5 minutes)
    # (see discussion above for supported versions)
    hg update v2.6.27

    # Select tho appropriate patches for the version of linux you selected
    hg qselect 2.6.27

    # Apply the patches
    hg qpush -a

    # Copy the default configuration file, so it's used
    cp .config.m5 .config

    # Compile the kernel (assuming the cross compiler is in $PATH, otherwise full path would need to be specified)
    # The dash after gnu is required.
    make ARCH=alpha CROSS_COMPILE=alpha-unknown-linux-gnu- vmlinux

