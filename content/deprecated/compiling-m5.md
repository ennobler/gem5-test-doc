---
title: "Compiling M5"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

gem5 runs on Linux and Mac OS X, but should be easily portable to other
Unix-like OSes. At times in the past gem5 has worked on OpenBSD and
Microsoft Windows (under Cygwin), but these platforms are not regularly
tested. Cygwin in particular is no longer actively supported; if you
must run on a Windows host, we recommend installing Linux (e.g., Ubuntu
Server) under a VM and running gem5 there. Free virtualization solutions
such as VirtualBox and VMware Player work well for this usage.

Cross-endian support has been mostly added, however this has not been
extensively tested. Running a program with syscall emulation is
supported regardless of host/target endianess, however full-system
simulation may have cross-endian issues (ALPHA full-system is known not
to work on big endian machines).

To build gem5, you need to ensure you have all the
[dependencies](dependencies "wikilink") installed.

## Possible Targets

gem5 can build many binaries each for a different guest architecture.
The currently available architectures are **ALPHA**, **ARM**, **MIPS**,
**POWER**, **SPARC**, and **X86**. In addition there is a **NULL**
architecture.

For each possible architecture and mode, several different executables
can be built:

  - **gem5.debug** - A binary used for debugging without any
    optimizations. Since no optimizations are done this binary is
    compiled the fastest, however since no optimizations are done it
    executes very slowly.
  - **gem5.opt** - A binary with debugging and optimization. This binary
    executes much faster than the debug binary and still provides all
    the debugging facility of the debug version. However when debugging
    source code it can be more difficult to use that the debug target.
  - **gem5.prof** - This binary is like the opt target, however it also
    includes profiling support suitable for use with gprof.
  - **gem5.perf** - Similar to prof, this target is aimed for CPU and
    heap profiling using the google perftools.
  - **gem5.fast** - This binary is the fastest binary and all debugging
    support is removed from the binary (including trace support). By
    default it also uses Link Time Optimization

## Compiling

Starting in the root of the source tree, you can build gem5 using a
command of the form:

`% scons build/`<arch>`/gem5.`<binary>

where the items between \<\> are the architectures, modes and binaries
listed above. For example:

    % cd gem5
    % scons build/ALPHA/gem5.debug

    scons: Reading SConscript files ...
    Checking for C header file fenv.h... yes
    Building in /tmp/gem5/build/ALPHA
    Options file /tmp/gem5/build/options/ALPHA not found,
      using defaults in build_opts/ALPHA
    Compiling in ALPHA with MySQL support.
    scons: done reading SConscript files.
    scons: Building targets ...
    g++ -o build/ALPHA/base/circlebuf.do -c -pipe -fno-strict-aliasing
        -Wall -Wno-sign-compare -Werror -Wundef -g3 -gdwarf-2 -O0
        -DTHE_ISA=ALPHA_ISA -DDEBUG -Iext/dnet -I/usr/include/python2.4
        -Ibuild/libelf/include -I/usr/include/mysql -Ibuild/ALPHA
        build/ALPHA/base/circlebuf.cc
    ...

If your output looked like the above, then congratulations, you've
compiled gem5\! The final binary is located at the path you specified in
the argument to scons, e.g., build/ALPHA/gem5.debug. For more build
options and further details about the build system, see the [SCons build
system](SCons_build_system "wikilink") page.

## Installing full system files

If you want to run the full-system version (including the full-system
regression tests), you will also need to download the full-system files
(disk images and binaries) from the [Download](Download "wikilink")
page.

The path to these files is determined in configs/common/SysPaths.py.
There are a couple of default paths hard-coded into this script; you can
place the system files at one of those paths, edit SysPaths.py to change
those paths, or override the paths in that file by setting your M5_PATH
environment variable. If this is not done correctly you will see an
error like `ImportError: Can't find a path to system files.` when you
first attempt to run the simulator in full-system mode.

Note that the default path, `/dist/m5/system`, is designed for
environments where you have root (sudo) access (to create `/dist`) and
want the files in a place where they can be shared by multiple users. If
both of these are true, you can follow this example to put the system
files at the default location:

    % sudo mkdir -p /dist/m5/system
    % cd /dist/m5/system
    % sudo tar vxfj <path>/m5_system_2.0b3.tar.bz2
    % sudo mv m5_system_2.0b3/* . ; sudo rmdir m5_system_2.0b3/
    % sudo chgrp -R <grp> /dist  # where <grp> is a group that contains all the m5 users

In most cases, it's simplest to put the files wherever is convenient and
then set M5_PATH to point to them.

## Testing your build

Once you've compiled gem5, you can verify that the build worked by
running regression tests. Regression tests are also run via scons. The
command to run all tests for a particular is constructed as follows:

`% scons build/`<target>`/tests/`<binary>

For example, to run the regression tests on ALPHA/gem5.opt, type:

`% scons build/ALPHA/tests/opt`

The regression framework is integrated into the scons build process, so
the command above will (re)build ALPHA/gem5.opt if necessary before
running the tests. Also thanks to scons's dependence tracking, tests
will be re-run only if the binary has been rebuilt since the last time
the test was run. If the previous test run is still valid (as far as
scons can tell), only a brief pass/fail message will be printed out
based on the result of that previous test, rather than the full output
and statistics diff that is printed when the test is actually executed.

Regression tests are further subdivided into three categories ("quick",
"medium", and "long") based on runtime. You can run only the tests in a
particular category by adding that category name to the target path,
e.g.:

`% scons build/ALPHA/tests/opt/quick`

(Note that currently the "medium" category is empty; all of the tests
are "quick" or "long".)

Specific tests can be run by appending the test name:

`% scons build/ALPHA/tests/opt/quick/fs/10.linux-boot`

For more details, see [Regression Tests](Regression_Tests "wikilink").

