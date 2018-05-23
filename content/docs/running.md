---
title: "Running gem5"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

## Usage

The gem5 command line has four parts, the gem5 binary, options for the
binary, a simulation script, and options for the script. The options
that are passed to the gem5 binary and those passed to the script are
handled separately, so be sure any options you use are being passed to
the right component.

    % <gem5 binary> [gem5 options] <simulation script> [script options]

## gem5 Options

Running gem5 with the "-h" flag prints a help message that includes all
of the supported simulator options. Here's a snippet:

    % build/ALPHA/gem5.debug -h
    Usage
    =====
      gem5.debug [gem5 options] script.py [script options]

     Copyright (c) 2001-2008 The Regents of The University of Michigan All Rights
    Reserved
    gem5 is copyrighted software; use the --copyright option for details.

    Options
    =======
    --version               show program's version number and exit
    --help, -h              show this help message and exit
    --build-info, -B        Show build information
    --copyright, -C         Show full copyright information
    --readme, -R            Show the readme
    --outdir=DIR, -d DIR    Set the output directory to DIR [Default: m5out]
    --redirect-stdout, -r   Redirect stdout (& stderr, without -e) to file
    --redirect-stderr, -e   Redirect stderr to file
    --stdout-file=FILE      Filename for -r redirection [Default: simout]
    --stderr-file=FILE      Filename for -e redirection [Default: simerr]
    --interactive, -i       Invoke the interactive interpreter after running the
                            script
    --pdb                   Invoke the python debugger before running the script
    --path=PATH[:PATH], -p PATH[:PATH]
                            Prepend PATH to the system path when invoking the
                            script
    --quiet, -q             Reduce verbosity
    ...

The default options that gem5 uses to run can be set by creating an
`~/.m5/options.py` file and placing options that you are interested in
there. For example, if you would like to always redirect standard error
and out to a file you could add: `options.stdout_file=simout` to
`options.py`.

## Script Options

The script section of the command line begins with a path to your script
file and includes any options that you'd like to pass to that script.
Most Example scripts allow you to pass a '-h' or '--help' flag to the
script to see script specific options. An example is as follows:

    gem5 compiled Apr  2 2011 00:57:11
    gem5 started Apr  3 2011 21:16:02
    gem5 executing on zooks
    command line: build/ALPHA/gem5.opt configs/example/se.py -h
    Usage: se.py [options]

    Options:
      -h, --help            show this help message and exit
      -c CMD, --cmd=CMD     The binary to run in syscall emulation mode.
      -o OPTIONS, --options=OPTIONS
                            The options to pass to the binary, use " " around the
                            entire string
      -i INPUT, --input=INPUT
                            Read stdin from a file.
      --output=OUTPUT       Redirect stdout to a file.
      --errout=ERROUT       Redirect stderr to a file.
      --ruby
      -d, --detailed
      -t, --timing
      --inorder
      -n NUM_CPUS, --num-cpus=NUM_CPUS
      --caches
      --l2cache
      --fastmem
      --clock=CLOCK
      --num-dirs=NUM_DIRS
      --num-l2caches=NUM_L2CACHES
      --l1d_size=L1D_SIZE
      --l1i_size=L1I_SIZE
      --l2_size=L2_SIZE
      --l1d_assoc=L1D_ASSOC
      --l1i_assoc=L1I_ASSOC
      --l2_assoc=L2_ASSOC
    ...

The script file documentation page ([Configuration / Simulation
Scripts](Configuration_Simulation_Scripts "wikilink")) describes how
to write your own simulation scripts, and the
[Options](Configuration_Simulation_Scripts#Options "wikilink") section
explains how to add your own command line options. The simulation
scripts that are most commonly used are se.py and fs.py. These scripts
are present in configs/examples directory. se.py is meant for simulation
using the system call emulation mode, while fs.py is for full-system
simulations. In most cases, it should be possible to use either of these
two scripts without any modifications. Understanding how these two
scripts work can help you decide on what modifications are required for
your particular case.

## System Call Emulation (SE) Mode

In this mode, one only needs to specify the binary file to be simulated.
This binary file can be statically/dynamically linked.
configs/examples/se.py is used for configuring and running simulations
in this mode. What follows is probably the simplest example of how to
use se.py. The binary file to simulated is specified with option
    **-c**.

    $ ./build/ALPHA/gem5.opt ./configs/example/se.py -c ./tests/test-progs/hello/bin/alpha/linux/hello
    gem5 Simulator System.  http://gem5.org
    gem5 is copyrighted software; use the --copyright option for details.

    gem5 compiled Mar  2 2014 00:06:39
    gem5 started Mar  4 2014 10:52:10
    gem5 executing on $
    command line: ./build/ALPHA/gem5.opt ./configs/example/se.py -c ./tests/test-progs/hello/bin/alpha/linux/hello
    Global frequency set at 1000000000000 ticks per second
    0: system.remote_gdb.listener: listening for remote gdb #0 on port 7000
    **** REAL SIMULATION ****
    info: Entering event queue @ 0.  Starting simulation...
    info: Increasing stack size by one page.
    Hello world!
    Exiting @ tick 3233000 because target called exit()

#### Specifying Command-Line Arguments

In order to pass command line arguments to a binary you can use
`--options="arg1 arg2 ..."` to specify them as a script option in your
simulation command.

## Full System (FS) Mode

This mode simulates a complete system which provides an operating system
based simulation environment. For full system mode, you can use the file
configs/example/fs.py for configuration and simulation. Sensible default
values have been set for the options that this script uses. We provide
examples for ALPHA and ARM based full system simulations.

{{\#ev:youtube|gd_DtxQD5kc|400|center|Example video showing gem5 full
system simulation for ARM. Host system is x86 64bit Ubuntu 12.04. Video
resolution can be set to 1080}}

### Booting Linux

We'll assume that you've already [built](Build_System "wikilink") an
ALPHA version of the gem5 simulator, and [downloaded and
installed](Introduction#Getting_Additional_Tools_and_Files "wikilink")
the full-system binary and disk image files. Then you can run the fs.py
configuration file in the gem5/configs/examples directory. For example:

    % build/ALPHA/gem5.debug -d /tmp/output configs/example/fs.py
    gem5 Simulator System

    Copyright (c) 2001-2006
    The Regents of The University of Michigan
    All Rights Reserved


    gem5 compiled Aug 16 2006 18:51:57
    gem5 started Wed Aug 16 21:53:38 2006
    gem5 executing on zeep
    command line: ./build/ALPHA/gem5.debug configs/example/fs.py
          0: system.tsunami.io.rtc: Real-time clock set to Sun Jan  1 00:00:00 2006
    Listening for console connection on port 3456
    0: system.remote_gdb.listener: listening for remote gdb #0 on port 7000
    warn: Entering event queue @ 0.  Starting simulation...
    <...simulation continues...>

#### Basic Operation

By default, the fs.py script boots Linux and starts a shell on the
system console. To keep console traffic separate from simulator input
and output, this simulated console is associated with a TCP port. To
interact with the console, you must connect to the port using a program
such as `telnet`, for example:

` % telnet localhost 3456`

Telnet's echo behavior doesn't work well with gem5, so if you are using
the console regularly, you probably want to use
[M5term](M5term "wikilink") instead of telnet. By default gem5 will try
to use port 3456, as in the example above. However, if that port is
already in use, it will increment the port number until it finds a free
one. The actual port number used is printed in the gem5 output.

In addition to loading a Linux kernel, gem5 mounts one or more disk
images for its filesystems. At least one disk image must be mounted as
the root filesystem. Any application binaries that you want to run must
be present on these disk images. To begin running benchmarks without
requiring an interactive shell session, gem5 can load .rcS files that
replace the normal Linux boot scripts to directly execute from after
booting the OS. These .rcS files can be used to configure ethernet
interfaces, execute special gem5 instructions, or begin executing a
binary on the disk image. The pointers for the linux binary, disk
images, and .rcS files are all set in the simulation script. (To see how
these files work, see [Configuration / Simulation
Scripts](Configuration_Simulation_Scripts "wikilink").) Examples:
Going into / of root filesystem and typing ls will show:

```
  benchmarks  etc     lib         mnt      sbin  usr
  bin         floppy  lost+found  modules  sys   var
  dev         home    man         proc     tmp   z
```

Snippet of an .rcS file:

    echo -n "setting up network..."
    /sbin/ifconfig eth0 192.168.0.10 txqueuelen 1000
    /sbin/ifconfig lo 127.0.0.1
    echo -n "running surge client..."
    /bin/bash -c "cd /benchmarks/surge && ./Surge 2 100 1 192.168.0.1 5.
    echo -n "halting machine"
    m5 exit

#### m5term

The m5term program allows the user to connect to the simulated console
interface that full-system gem5 provides. Simply change into the
util/term directory and build m5term:

```
    % cd gem5/util/term
    % make
    gcc  -o m5term term.c
    % make install
    sudo install -o root -m 555 m5term /usr/local/bin
```

The usage of m5term is:

```
    ./m5term <host> <port>

    <host> is the host that is running gem5

    <port> is the console port to connect to. gem5 defaults to
    using port 3456, but if the port is used, it will try the next
    higher port until it finds one available.

    If there are multiple systems running within one simulation,
    there will be a console for each one.  (The first system's
    console will be on 3456 and the second on 3457 for example)

    m5term uses '~' as an escape character.  If you enter
    the escape character followed by a '.', the m5term program
    will exit.
```

m5term can be used to interactively work with the simulator, though
users must often set various terminal settings to get things to work

A slightly shortened example of m5term in action:

```
    % m5term localhost 3456
    ==== m5 slave console: Console 0 ====
    M5 console
    Got Configuration 127
    memsize 8000000 pages 4000
    First free page after ROM 0xFFFFFC0000018000
    HWRPB 0xFFFFFC0000018000 l1pt 0xFFFFFC0000040000 l2pt 0xFFFFFC0000042000 l3pt_rpb 0xFFFFFC0000044000 l3pt_kernel 0xFFFFFC0000048000 l2reserv 0xFFFFFC0000046000
    CPU Clock at 2000 MHz IntrClockFrequency=1024
    Booting with 1 processor(s)
    ...
    ...
    VFS: Mounted root (ext2 filesystem) readonly.
    Freeing unused kernel memory: 480k freed
    init started:  BusyBox v1.00-rc2 (2004.11.18-16:22+0000) multi-call binary

    PTXdist-0.7.0 (2004-11-18T11:23:40-0500)

    mounting filesystems...
    EXT2-fs warning: checktime reached, running e2fsck is recommended
    loading script...
    Script from M5 readfile is empty, starting bash shell...
    # ls
    benchmarks  etc         lib         mnt         sbin        usr
    bin         floppy      lost+found  modules     sys         var
    dev         home        man         proc        tmp         z
    #
```

#### Full System Benchmarks

We have several full-system benchmarks already up and running. The
binaries are available in the disk images you can obtain/download from
us, and the .rcS files are in the gem5/configs/boot/ directory. To run
any of them, you merely need to set the benchmark option to the name of
the test you want to run. For example:

    %./build/ALPHA/gem5.opt  configs/example/fs.py -b NetperfMaerts

To see a comprehensive list of all benchmarks available:

    %./build/ALPHA/gem5.opt configs/examples/fs.py -h

### Experimenting with DVFS

This is a quick hands-on tutorial to start a DVFS-enabled system where
the Linux DVFS governors can change voltage and frequencies of the
ongoing simulation. Right now, the driver and interface components live
in ARM-specific parts of the Linux kernel / gem5, but there is no
fundamental reason why this could not be ported to work on other
architectures, too.

#### Quick Instructions

These instructions apply for a Ubuntu-based machine, but can be easily
adapted / extended etc. for other use cases and systems.

  - Get gem5 with the proper changesets added
  - Anything after [1](http://repo.gem5.org/gem5/rev/d65768b9ffc2)

<!-- end list -->

    hg clone http://repo.gem5.org/gem5

<li>

Build gem5

</li>

    scons build/ARM/gem5.opt -j 8

<li>

Get a DVFS-enabled Linux kernel

</li>

  - From here:
    [2](http://www.linux-arm.org/git?p=linux-linaro-tracking-gem5.git;a=summary)
  - Anything after / including
    [3](http://www.linux-arm.org/git?p=linux-linaro-tracking-gem5.git;a=commit;h=a75e551a89819c96bb762c25fa104b32eda7b99b)

<!-- end list -->

    git clone --depth 10 git://www.linux-arm.org/linux-linaro-tracking-gem5.git

<li>

Get a cross-compile tool chain

</li>

    sudo apt-get install gcc-arm-linux-gnueabihf

<li>

Build the kernel

</li>

  - See also [Linux_kernel](Linux_kernel "wikilink")

<!-- end list -->

    make ARCH=arm vexpress_gem5_dvfs_defconfig
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j8

<li>

Check / select the right DTS / DTB file

</li>

    ls arch/arm/boot/dts/vexpress-v2*dvfs*

<li>

Get / prepare a disk image

</li>

    wget http://www.gem5.org/dist/current/arm/arm-system-2013-07.tar.bz2
    tar xvjf arm-system-2013-07.tar.bz2

<li>

Add DVFS points to the configuration

</li>

  - Enable and link the energy controller / DVFS handler
        patch -p1 << EOF
        diff --git a/configs/example/fs.py b/configs/example/fs.py
        --- a/configs/example/fs.py
        +++ b/configs/example/fs.py
        @@ -106,7 +106,11 @@
             # Create a source clock for the CPUs and set the clock period
             test_sys.cpu_clk_domain = SrcClockDomain(clock = options.cpu_clock,
                                                      voltage_domain =
        -                                             test_sys.cpu_voltage_domain)
        +                                             test_sys.cpu_voltage_domain,
        +                                             domain_id = 0)
        +
        +    test_sys.dvfs_handler.domains = test_sys.cpu_clk_domain
        +    test_sys.dvfs_handler.enable = 1

             if options.kernel is not None:
                 test_sys.kernel = binary(options.kernel)
        EOF

<!-- end list -->

  - Can also change the clock frequencies here, or from command line

<li>

Start a simple test
    simulation

</li>

    M5_PATH=$(pwd)/.. ./build/ARM/gem5.opt --debug-flags=DVFS,EnergyCtrl \
      --debug-file=dfvs_debug.log configs/example/fs.py --cpu-type=AtomicSimpleCPU \
      -n 2 --machine-type=VExpress_EMM --kernel=../linux-linaro-tracking-gem5/vmlinux \
      --dtb-filename=../linux-linaro-tracking-gem5/arch/arm/boot/dts/\
    vexpress-v2p-ca15-tc1-gem5_dvfs_2cpus.dtb \
      --disk-image=../disks/arm-ubuntu-natty-headless.img \
      --cpu-clock=\['1 GHz','750 MHz','500 MHz'\]

<li>

Test DVFS functionality

</li>

    util/term/m5term 3456
    <login>
    cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_frequencies
    echo 750187 > /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq

</ul>

#### Futher Experiments

  - Set up different voltages for the operating points
        patch -p1 < EOF
        diff --git a/configs/example/fs.py b/configs/example/fs.py
        --- a/configs/example/fs.py
        +++ b/configs/example/fs.py
        @@ -101,12 +101,16 @@
                     voltage_domain = test_sys.voltage_domain)

             # Create a CPU voltage domain
        -    test_sys.cpu_voltage_domain = VoltageDomain()
        +    test_sys.cpu_voltage_domain = VoltageDomain(voltage = ['1V','0.9V','0.8V'])

             # Create a source clock for the CPUs and set the clock period
             test_sys.cpu_clk_domain = SrcClockDomain(clock = options.cpu_clock,
                                                      voltage_domain =
        -                                             test_sys.cpu_voltage_domain)
        +                                             test_sys.cpu_voltage_domain,
        +                                             domain_id = 0)
        +
        +    test_sys.dvfs_handler.domains = test_sys.cpu_clk_domain
        +    test_sys.dvfs_handler.enable = 1

             if options.kernel is not None:
                 test_sys.kernel = binary(options.kernel)
        EOF
  - Per-core DVFS

<!-- end list -->

  - Set up separate clock (and voltage) domains per core
  - Separate clock domains need separate clusters in the device tree
        diff -u linux-linaro-tracking-gem5/arch/arm/boot/dts/\
        vexpress-v2p-ca15-tc1-gem5_dvfs_{,per_core_}4cpus.dts</code>
  - Change the `socket_id` to have a separate socket per CPU core

