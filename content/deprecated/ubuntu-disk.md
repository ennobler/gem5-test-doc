---
title: "Full-system Ubuntu Image"
date: 2018-05-13T18:51:37-04:00
draft: false
---
{{% notice warning %}}
This content is out-of-date.
{{% /notice %}}

This page describes how to build a serial-console filesystem of Ubuntu
Linux for ARM ISA simulation after the bare image file is created. An
example Ubuntu Natty ARM image is available on the
[Download](Download "wikilink") page.

## Using Rootstock and Qemu to build the filesystem

One way to create a disk image is to use the rootstock tool provided in
older versions of Ubuntu to build an ARM filesystem. To create a base
2GB filesystem that can be booted to the serial console, run the
following
command:

`%>rootstock --fqdn gem5sim --login gem5 --password 5meg --imagesize 2G --seed build-essential`

Other packages can be added to the --seed option list to be installed by
rootstock. Rootstock will create a tar file containing the file system.
Unpack this tar file into the blank disk image. Packages can also be
installed at a later date by mounting the filesystem to a loop device,
mounting its proc filesystem and chroot'ing into the filesystem. Qemu
will be used to emulate the binaries within the ARM filesystem to
install additional packages using apt-get. For the below to work you
need to have installed the rootstock packages so the various qemu
emulators are available. For example

**Attention: Rootstock has been deprecated by Ubuntu for making ARM disk
images from x86 machines. They now offer core file system image tarballs
for
download.**

## Using Ubuntu CoreFS and Qemu to build an Ubuntu file system for 12.04LTS+

[Ubuntu
Core](http://cdimage.ubuntu.com/ubuntu-core/releases/12.04/release/)
offers pre-compiled base file systems in a tarball to use instead of
rootstock. Since v12.04 of Ubuntu, cannonical now offers hardfloat (hf)
ABI compiled binaries to make better use of VFP and NEON along with the
usual softfloat binaries (armel). Make sure to install the
qemu-arm-static package to be able to install packages to your
file-system after you have unpacked the tarball to a blank disk image.
After unpacking the tarball to the blank image copy the qemu-arm-static
binary from /usr/bin from your host system to the usr/bin directory of
your disk image. This allows ARM emulation for installing packages
directly to the disk image, and even for compiling source packages using
an ARM version of GCC. Perform the following operations to get a
functional Ubuntu file system that can run hardfloat and softfloat
binaries.

` `
` // if you have not mounted the disk image, do so now, make sure to unpack the core`
` // fs to this disk image before continuing`
` mount -oloop,offset=32256 /tmp/Ubuntu-arm.img /mnt`
` cd /mnt `
` mount -o bind /proc /mnt/proc`
` mount -o bind /dev /mnt/dev`
` mount -o bind /sys /mnt/sys`
` cp /etc/resolv.conf /mnt/etc/`
` chroot .`
` `
` // enable the universe repo in etc/apt/sources.list `
` apt-get update`
` apt-get install ubuntu-minimal`
` apt-get install build-essential`
` apt-get install vim`
` apt-get install gcc-multilib`
` apt-get install g++-multilib`
` apt-get install <what ever other packages you want>`
` `

If you transfer the file system created here to another computer make
sure resolv.conf in /etc of the created filesystem is updated to match
the resolv.conf of the host system, otherwise apt-get will fail to
function properly.

## Setting up Upstart to be Gem5 friendly

Instead of the old SysV and init.d, Ubuntu uses Upstart for mounting
filesystems and loading the various daemons upon boot. To speed up the
process in simulation the following upstart scripts should be removed
from the /etc/init folder in the new filesystem as they are not needed:

  - console.conf
  - console-setup.conf
  - container-detect.conf
  - cron.conf
  - dmesg.conf
  - hwclock.conf
  - hwclock-save.conf
  - mounted-debugfs.conf
  - mounted-tmp.conf
  - mountall-net.conf
  - network-interface-container.conf
  - network-interface-security.conf
  - plymouth.conf
  - plymouth-log.conf
  - plymouth-splash.conf
  - plymouth-stop.conf
  - plymouth-upstart-bridge.conf
  - rsyslog.conf
  - setvtrgb.conf
  - tty\[x\].conf (only need tty1.conf)
  - udev-fallback-graphics.conf
  - ureadahead.conf
  - ureadahead-other.conf

The following changes should be made to the mountall.conf script:

  - Remove the fsck checks at the beginning of the script declaration
  - Remove the fsck checks in the post-script declaration
  - Remove the exec mountall --daemon declaration so total control of
    what filesystems are mounted and when is retained

To the script field on the mountall.conf file, add at least the
following:

`  `
` # Mount appropriate file systems from fstab and remount root.`
` mount /proc`
` mount /tmp`
` mount /sys`
` mount -o remount,rw /dev/sda1 /`
` swapon /swapfile # if present`
`  `
` # Make sure to emit all events that mountall would have`
` initctl emit virtual-filesystems`
` initctl emit local-filesystems`
` initctl emit remote-filesystems`
` initctl emit all-swaps`
` initctl emit filesystem`
` initctl emit mounting`
` initctl emit mounted`
` `

Additionally, one of the tty.conf scripts should be modified like below
to get a login prompt within m5term or load an .rcS job script with
multi-user and job-control
enabled:

`  `
` script`
`   if [ ! -c /dev/ttyAMA0 ]`
`   then`
`     mknod /dev/ttyAMA0 c 204 64`
`   fi`
`   if [ ! -c /dev/ttySA0 ]`
`   then`
`     if [ ! -L /dev/ttySA0 ]`
`     then`
`       ln -s /dev/ttyAMA0 /dev/ttySA0`
`     fi`
`   fi `
`  `
`   /sbin/m5 readfile > /tmp/script`
`   chmod 755 /tmp/script`
`   if [ -s /tmp/script ]`
`   then`
`     exec su root -c '/tmp/script' # gives script full privileges as root user in multi-user mode`
`     exit 0`
`   else`
`     exec /sbin/getty -L ttySA0 38400 vt100 # login prompt`
`   fi`
` end script`
` `

By default a modules.dep file will not be created by rootstock, this
file is needed to prevent certain upstart scripts from failing. To
create this file, add the following to the beginning of the script
declaration in module-init-tools.conf, or simply add your own custom
modules.dep file in the appropriate directory for your
kernel.

`  `
`` if [ ! -e /lib/modules/`uname -r`/modules.dep ]``
` then`
``   mkdir /lib/modules/`uname -r` ``
``   echo "#No modules for this run of Gem5" > /lib/modules/`uname -r`/modules.dep``
` fi`
` `

To allow passwordless login to the filesystem, edit the /etc/shadow file
to have the gem5 and root entries look like the following.

` `
` root::15201:0:99999:7:::`
` gem5::15201:0:99999:7:::`
` `

Ensure that the tty that will be used as the login tty is contained
within the /etc/securetty file. In most cases this will probably be
ttyAMA0. Go through the mounted-proc.conf, mounted-dev.conf,
mounted.tmp.conf and mounted-varrun.conf and verify all use the clause

`  `
` start on mounted `
` `

so the scripts start running. Also make sure to remove dependencies on
variables that contain the word "container". This appears to be an
addition for virtualization but is unnecessary for Gem5.

The rc-sysinit.conf file should have most of the script body commented
out except for these lines:

`  `
` [ -n "${FROM_SINGLE_USER_MODE}" ] || /etc/init.d/rcS`
` telinit "${DEFAULT_RUNLEVEL}"`
` `

Finally the /etc/hosts file must contain the following to get proper
behavior for programs that use sockets to do RPC:

`  `
` 127.0.0.1 localhost`
` ::1 localhost ip6-localhost ip6-loopback`
` fe00::0 ip6-localnet`
` ff00::0 ip6-mcastprefix`
` ff02::1 ip6-allnodes`
` ff02::2 ip6-allrouters`
` ff02::3 ip6-allhosts`
` `

