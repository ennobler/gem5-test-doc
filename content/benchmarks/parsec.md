---
title: "PARSEC"
date: 2018-05-13T18:51:37-04:00
draft: false
---

## PARSEC v2.1

The PARSEC benchmarks have been built to run in the gem5 full-system
simulation mode. The following section details references and the
process for building and running the suite.

### Download Pre-built M5 Disk Image

PARSEC has been built to run on gem5 with the ALPHA ISA, and disk images
are available at [Running PARSEC 2.1 on M5 from The University of
Texas](http://www.cs.utexas.edu/~parsec_m5/). You can download a disk
image there and unzip it.

The original system files that can be downloaded from
<http://www.m5sim.org/Download> will have the following directory
structure upon extracting:

    system/
           binaries/
                  console
                  ts_osfpal
                  vmlinux
           disks/
                  linux-bigswap2.img
                  linux-latest.img

Now, if you are going to run for the ALPHA ISA, then you need to swap
the ts_osfpal, vmlinux, linux-latest.img files in the above directory
structure with the ones provided at
<http://www.cs.utexas.edu/~parsec_m5/>

After unzipping the file, you will need to specify where gem5 should
look for the disk image. In the file ./configs/common/SysPaths.py,
specify the path to your disk image in the
    line:

    path = [ ’/dist/m5/system’, ’<complete path to your disks and binaries directory>’ ]

Next, in ./configs/common/Benchmarks.py, specify the name of your disk
image. For example:

    return env.get(’LINUX_IMAGE’, disk(’linux-parsec-2-1-m5.img’))

You should now be set up to run with benchmarks that are on the disk
image. See the section below about running gem5 for more details on how
to execute the simulation.

### Running PARSEC in gem5

To run PARSEC in gem5, you can specify a run script to gem5 on the
command
    line

    ./build/ALPHA_FS/m5.opt ./configs/example/fs.py --script=./path/to/runscript.rcS

Where the script, runscript.rcS is a shell script that contains commands
to execute the benchmark. For example:

    #!/bin/sh
    # File to run the blackscholes benchmark
    cd /parsec/install/bin
    /sbin/m5 dumpresetstats
    ./blackscholes 64 /parsec/install/inputs/blackscholes/in_64K.txt /parsec/install/inputs/blackscholes/prices.txt
    echo "Done :D"
    /sbin/m5 exit

This example changes directories to the location where the binary exists
on the disk image, resets the gem5 statistics, runs the blackscholes
benchmark and exits.

### References

For more details on the process for building and running PARSEC for the
ALPHA ISA, see [Running PARSEC 2.1 on M5 from The University of
Texas](http://www.cs.utexas.edu/~parsec_m5/).

## PARSEC 3.0

### ARM

In the following setting, 12 out of the 13 PARSEC benchmarks can be
built (and run\!) successfully:

  - PARSEC 3.0-beta-20150206 with this
    [patch](https://n.ethz.ch/~pfistchr/download/parsec-arm-static.diff)
    [Mirror
    \#1](https://mega.nz/#!Fpk03BRQ!qCKp7Tpy-OJJH7XcKfDB9o64SHR_jpRJQNA0ZqfR2Mk)
    [Mirror \#2](http://pastie.org/10844282)
  - native compilation on Debian Wheezy armhf (e.g. on qemu-system-arm
    -M vexpress-a9)
  - the resulting static binaries work in gem5 (if you copy them, do not
    forget to copy the input files as well)

The following benchmarks work:

  - blackscholes
  - bodytrack
  - canneal
  - dedup
  - facesim
  - ferret
  - fluidanimate
  - freqmine
  - streamcluster
  - swaptions
  - vips
  - x264

The following benchmark does not work:

  - raytrace (LRT/render.cxx uses SSE intrinsics)

