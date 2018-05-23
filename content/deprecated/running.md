---
title: "Running gem5"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---
{{% notice warning %}}
**THIS IS AN OBSOLETE PAGE. DO NOT EDIT.**
for the most recent version of this
documentation.
{{% /notice %}}

### Quick Start

We'll assume that you've already [built](Compiling_M5 "wikilink") an
ALPHA_FS version of the M5 simulator, and [downloaded and
installed](Compiling_M5#Installing_full_system_files "wikilink") the
full-system binary and disk image files. Then you can just run the fs.py
configuration file in the m5/configs/examples directory. For example:

    % build/ALPHA_FS/m5.debug -d /tmp/output configs/example/fs.py
    M5 Simulator System

    Copyright (c) 2001-2006
    The Regents of The University of Michigan
    All Rights Reserved


    M5 compiled Aug 16 2006 18:51:57
    M5 started Wed Aug 16 21:53:38 2006
    M5 executing on zeep
    command line: ./build/ALPHA_FS/m5.debug configs/example/fs.py
          0: system.tsunami.io.rtc: Real-time clock set to Sun Jan  1 00:00:00 2006
    Listening for console connection on port 3456
    0: system.remote_gdb.listener: listening for remote gdb #0 on port 7000
    warn: Entering event queue @ 0.  Starting simulation...
    <...simulation continues...>

### Basic Operation

By default, the fs.py script boots Linux and starts a shell on the
system console. To keep console traffic separate from simulator input
and output, this simulated console is associated with a TCP port. To
interact with the console, you must connect to the port using a program
such as `telnet`, for example:

` % telnet localhost 3456`

Telnet's echo behavior doesn't work well with m5, so if you are using
the console regularly, you probably want to use
[M5term](M5term "wikilink") instead of telnet. By default m5 will try to
use port 3456, as in the example above. However, if that port is already
in use, it will increment the port number until it finds a free one. The
actual port number used is printed in the m5 output.

In addition to loading a Linux kernel, M5 mounts one or more disk images
for its filesystems. At least one disk image must be mounted as the root
filesystem. Any application binaries that you want to run must be
present on these disk images. To begin running benchmarks without
requiring an interactive shell session, M5 can load .rcS files that
replace the normal Linux boot scripts to directly execute from after
booting the OS. These .rcS files can be used to configure ethernet
interfaces, execute special m5 instructions, or begin executing a binary
on the disk image. The pointers for the linux binary, disk images, and
.rcS files are all set in the simulation script. (To see how these files
work, see [Simulation Scripts
Explained](Simulation_Scripts_Explained "wikilink").) Examples: Going
into / of root filesystem and typing ls will show:

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

### Full System Benchmarks

We have several full-system benchmarks already up and running. The
binaries are available in the disk images you can obtain/download from
us, and the .rcS files are in the m5/configs/boot/ directory. To run any
of them, you merely need to set the benchmark option to the name of the
test you want to run. For example:

    %./build/ALPHA_FS/m5.opt  configs/example/fs.py -b NetperfMaerts

To see a comprehensive list of all benchmarks available:

    %./build/ALPHA_FS/m5.opt configs/examples/fs.py -h

Not every benchmark is commonly used though, and not all are guaranteed
to be useful or in working condition. However, we do often run:

  - NetperfMaerts
  - NetperfStreamNT
  - SurgeSpecweb

These should run without a problem, since we have flushed out most bugs.

Currently under development:

  - NFS
  - iSCSI
  - video streaming

