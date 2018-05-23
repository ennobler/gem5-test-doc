---
title: "Disk images"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---


## Background

{{% youtube Oh3NK12fnbg %}}

### Disk image basics

A disk device in gem5 gets its initial contents from a file called a
disk image. This file stores all the bytes present on the disk just as
you would find them on an actual device. Some other systems also use
disk images which are in more complicated formats and which provide
compression, encryption, etc. gem5 currently only supports raw images,
so if you have an image in one of those other formats, you'll have to
convert it into a raw image before you can use it in a simulation. There
are often tools available which can convert between the different
formats.

Because a disk image represents all the bytes on the disk itself, it
contains more than just a file system. For harddrives on most systems,
the image starts with a partition table. Each of the partitions in the
table (frequently only one) is also in the image. If you want to
manipulate the entire disk you'll use the entire image, but if you want
to work with just one partition and/or the file system on it, you'll
need to specifically select that part of the image. The losetup command
(discussed below) has a -o option which lets you specify where to start
in an image.

### Creating an empty image

The recommended method to create a disk image is to use
./util/gem5img.py

It's a good idea to understand how to build an image in case something
goes wrong or you need to do something in an unusual way. However,
gem5img.py script which will go through the process of building and
formatting an image. If you want to understand the guts of what it's
doing see below. You can use the "init" command to create an empty
image, "new", "partition", or "format" to perform those parts of init
independently, and "mount" or "umount" to mount or unmount an existing
image.

### Mounting an image

{{\#ev:youtube|OXH1oxQbuHA|400|center|A youtube video of add file using
mount on Ubuntu 12.04 64bit. Video resolution can be set to 1080}}

To mount a file system on your image file, first find a loopback device
and attach it to your image with an appropriate offset as described in
the "Formatting" section above.

`mount -o loop,offset=32256 foo.img`

### Unmounting

To unmount an image, use the umount command like you normally would.

`umount`

## Image contents

Now that you can create an image file and mount it's file system, you'll
want to actually put some files in it. You're free to use whatever files
you want, but the gem5 developers have found that Gentoo stage3 tarballs
are a great starting point. They're essentially an almost bootable and
fairly minimal Linux installation and are available for a number of
architectures.

If you choose to use a Gentoo tarball, first extract it into your
mounted image. The /etc/fstab file will have placeholder entries for the
root, boot, and swap devices. You'll want to update this file as
apporpriate, deleting any entries you aren't going to use (the boot
partition, for instance). Next, you'll want to modify the inittab file
so that it uses the m5 utility program (described elsewhere) to read in
the init script provided by the host machine and to run that. If you
allow the normal init scripts to run, the workload you're interested in
may take much longer to get started, you'll have no way to inject your
own init script to dynamically control what benchmarks are started, for
instance, and you'll have to interact with the simulation through a
simulated terminal which introduces non-determinism.

### Modifications

By default gem5 does not store modifications to the disk back to the
underlying image file. Any changes you make will be stored in an
intermediate COW layer and thrown away at the end of the simulation. You
can turn off the COW layer if you want to modify the underlying disk.

### Kernel and bootloader

Also, generally speaking, gem5 skips over the bootloader portion of boot
and loads the kernel into simulated memory itself. This means that
there's no need to install a bootloader like grub to your disk image,
and that you don't have to put the kernel you're going to boot from on
the image either. The kernel is provided separately and can be changed
out easily without having to modify the disk image.

### How to create an Ubuntu image for ARM_FS

A howto on creating an Ubuntu based full system image for gem5 can be
found at [Ubuntu Disk Image for ARM Full
System](Ubuntu_Disk_Image_for_ARM_Full_System "wikilink").

## Manipulating images with loopback devices

### Loopback devices

Linux supports loopback devices which are devices backed by files. By
attaching one of these to your disk image, you can use standard Linux
commands on it which normally run on real disk devices. You can use the
mount command with the "loop" option to set up a loopback device and
mount it somewhere. Unfortunately you can't specify an offset into the
image, so that would only be useful for a file system image, not a disk
image which is what you need. You can, however, use the lower level
losetup command to set up a loopback device yourself and supply the
proper offset. Once you've done that, you can use the mount command on
it like you would on a disk partition, format it, etc. If you don't
supply an offset the loopback device will refer to the whole image, and
you can use your favorite program to set up the partitions on it.

## Working with image files

To create an empty image from scratch, you'll need to create the file
itself, partition it, and format (one of) the partition(s) with a file
system.

#### Create the actual file

First, decide how large you want your image to be. It's a good idea to
make it large enough to hold everything you know you'll need on it, plus
some breathing room. If you find out later it's too small, you'll have
to create a new larger image and move everything over. If you make it
too big, you'll take up actual disk space unnecessarily and make the
image harder to work with. Once you've decided on a size you'll want to
actually create the file. Basically, all you need to do is create a file
of a certain size that's full of zeros. One approach is to use the dd
command to copy the right number of bytes from /dev/zero into the new
file. Alternatively you could create the file, seek in it to the last
byte, and write one zero byte. All of the space you skipped over will
become part of the file and is defined to read as zeroes, but because
you didn't explicitly write any data there, most file systems are smart
enough to not actually store that to disk. You can create a large image
that way but take up very little space on your physical disk. Once you
start writing to the file later that will change, and also if you're not
careful, copying the file may expand it to its full size.

#### Partitioning

First, find an available loopback device using the losetup command with
the -f option.

`losetup -f`

Next, use losetup to attach that device to your image. If the available
device was /dev/loop0 and your image is foo.img, you would use a command
like this.

`losetup /dev/loop0 foo.img`

/dev/loop0 (or whatever other device you're using) will now refer to
your entire image file. Use whatever partitioning program you like on it
to set up one (or more) paritions. For simplicity it's probably a good
idea to create only one parition that takes up the entire image. We say
it takes up the entire image, but really it takes up all the space
except for the partition table itself at the beginning of the file, and
possibly some wasted space after that for DOS/bootloader compatibility.

From now on we'll want to work with the new partition we created and not
the whole disk, so we'll free up the loopback device using losetup's -d
option

`losetup -d /dev/loop0`

#### Formatting

First, find an available loopback device like we did in the partitioning
step above using losetup's -f option.

`losetup -f`

We'll attach our image to that device again, but this time we only want
to refer to the partition we're going to put a file system on. For PC
and Alpha systems, that partition will typically be one track in, where
one track is 63 sectors and each sector is 512 bytes, or 63 \* 512 =
32256 bytes. The correct value for you may be different, depending on
the geometry and layout of your image. In any case, you should set up
the loopback device with the -o option so that it represents the
partition you're interested in.

`losetup -o 32256 /dev/loop0 foo.img`

Next, use an appropriate formating command, often mke2fs, to put a file
system on the partition.

`mke2fs /dev/loop0`

You've now successfully created an empty image file. You can leave the
loopback device attached to it if you intend to keep working with it
(likely since it's still empty) or clean it up using losetup -d.

`losetup -d /dev/loop0`

Don't forget to clean up the loopback device attached to your image with
the losetup -d command.

`losetup -d /dev/loop0`
