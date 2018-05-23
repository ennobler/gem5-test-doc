---
title: "Software not in git"
date: 2018-05-12T21:19:58-04:00
draft: false
---
### Full-System Stuff

You would need one or more of the following files to full system
simulations using gem5. If you download these files, read
[this](Introduction#Getting_Additional_Tools_and_Files "wikilink") page
for instructions on how to install these files.

  - ARM
      - [ARM Full-System Files](http://www.gem5.org/dist/current/arm/)
        -- Pre-compiled kernel and disk images for 32-bit and 64-bit ARM
        simulation. Updated October 2014. There kernels all support PCIe
        devices and the 64-bit kernels support \>2GB of DRAM.
      - [Legacy ARM Full System
        Files](Legacy_ARM_Full_System_Files "wikilink") -- A collection
        of previous ARM files that have been distributed. Anyone getting
        started with ARM and gem5 should use the above link.
      - [BBench for gem5](BBench-gem5 "wikilink") -- Full-system Android
        files and [BBench](http://bbench.eecs.umich.edu), a web-browser
        benchmark.
      - [AsimBench for gem5](AsimBench "wikilink") -- Full-system
        Android files for
        [AsimBench](http://asg.ict.ac.cn/projects/asimbench), a
        benchmark suite containing various types of mobile applications.
  - X86
      - [Full System
        Files](http://www.m5sim.org/dist/current/x86/x86-system.tar.bz2)
        -- The kernel used for regressions, an SMP version of it, and a
        disk image
      - [config
        files](http://www.m5sim.org/dist/current/x86/config-x86.tar.bz2)
        -- Config files for both of the above kernels, 2.6.25.1 and
        2.6.28.4
  - (The `mkblankimage.sh` script to create a blank disk image that used
    to be downloadable here is now included in the m5 repository, in the
    `util` directory.)
  - Alpha
      - [Full System
        Files](http://www.m5sim.org/dist/current/m5_system_2.0b3.tar.bz2)
        -- Pre-compiled Linux kernels, PALcode/Console code, and a
        filesystem
          - Unchanged since M5 2.0 beta 3. If you already have these you
            don't need them again.
      - [linux-dist](http://www.m5sim.org/dist/current/linux-dist.tgz)
        -- Everything you need to create your own disk image and compile
        everything in it from scratch

### Benchmarks

  - For information about running Android on gem5 and using the web
    browser benchmark, see [BBench-gem5](BBench-gem5 "wikilink").
  - For information about running the AsimBench benchmark on Android
    with gem5, see [AsimBench](AsimBench "wikilink") for more
    information.
  - For information about using the [DaCapo
    benchmarks](http://www.dacapobench.org) on gem5 see the [DaCapo
    benchmarks](DaCapo_benchmarks "wikilink") page for more information.
  - [SPLASH
    benchmarks](http://www.gem5.org/dist/m5_benchmarks/v1-splash-alpha.tgz)
    -- See the [Splash benchmarks](Splash_benchmarks "wikilink") page
    for more information.

### Pre-compiled Cross-compilers

Externally supplied cross compilers:

  - Ubuntu users can simply install ARM compilers with the
    crossbuild-essential-armhf and libc6-dev-armhf-armel-cross packages
    for 32-bit ARM and crossbuild-essential-arm64 and
    libc6-dev-arm64-cross for 64-bit ARM.
  - [MIPS cross compilers from
    CodeSourcery](http://www.codesourcery.com/sgpp/lite/mips/portal/subscription3130)

All generated with [crosstool](http://www.kegel.com/crosstool/) for x86
linux hosts/linux targets

  - Alpha:
    [gcc-3.4.3](http://www.m5sim.org/dist/current/alpha_crosstool.tar.bz2),
    [gcc-4.3.2, glibc-2.6.1
    (NPTL,x86/64)](http://www.m5sim.org/dist/current/alphaev67-unknown-linux-gnu.tar.bz2),
    [gcc-4.3.2, glibc-2.6.1
    (NPTL,x86/32)](http://www.m5sim.org/dist/current/alphaev67-unknown-linux-gnu-x86-32.tar.bz2)
  - [SPARC64](http://www.m5sim.org/dist/current/sparc64_crosstool.tar.bz2)


