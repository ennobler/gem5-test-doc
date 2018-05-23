---
title: "Linux kernel"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

Depending on what ISA you're using with gem5 the instructions for
compiling a Linux kernel differ slightly. Ultimately, we'd like to have
a single unified way to compile all these kernels, but it will take some
work to happen. If you'd like to help, please let us know.

### Alpha

Currently we have a patch queue for Alpha that can be applied on top of
a linux mercurial repository. This patch queue adds various debugging,
performance, introspection capabilities and increases the number of
simulated processors the Alpha implementation in gem5 can support from 4
to 64. To compile a new kernel for Alpha follow the directions
below:

`# First clone a copy of the linux repository`
`hg clone `<git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux-2.6.git>` linux-2.6`
`cd linux-2.6`

`# Go to the .hg directory`
`cd .hg`

`#Get a copy of the patches repository`
`hg clone `<http://repo.gem5.org/linux-patches>` patches`

`# Go back to the linux directory`
`cd ..`

`#Update the linux repository to a known working version`
`hg update v2.6.27`
` `
`# Select the patches for 2.6.27`
`hg qselect 2.6.27`
` `
`# Apply the patches to the repository`
`hg qpush -a`

`# Copy the default config file to .config`
`cp .config.m5 .config`

`# Build the kernel`
`make ARCH=alpha CROSS_COMPILE=/path/to/alpha/compiler/alpha-unknown-linux-gnu- vmlinux -j 4`

I few minutes later a kernel will be compiled. You can use a newer
version of m5, however you'll likely need to modify the config file and
some of the patches slightly for it to work.

### ARM

See [ARM Linux Kernel](ARM_Linux_Kernel "wikilink").

### x86

x86 doesn't currently have a patch queue, but a generic kernel works
well since the normal code already has functions that wait for
interrupts in idle loops. Some of the debugging functionality, such as
m5dprintk can be trivially added if you need to by just using the patch
from the alpha linux-patches repository
above.

`# To compile a kernel for x86, first get the kernel source. Here again we use mercurial because that is what we use for gem5`
`# However, you could get the code from kernel.org in a bzip format`
`hg clone `<http://www.kernel.org/hg/linux-2.6>

`# Change directory to the newly cloned directory`
`cd linux-2.6`

`# Update to the version you're interested in; 2.6.28.4 in this case because that is what the config file we've got is for`
`hg update v2.6.28.4`

`# Grab the config file on the download page:`
`wget `<http://www.m5sim.org/dist/current/x86/config-x86.tar.bz2>
`tar jxvf config-x86.tar.bz2`
`cp configs/linux-2.6.28.4 .config`

`# Compile the kernel, assuming you're building on an x86 host`
`make vmlinux -j 4`

