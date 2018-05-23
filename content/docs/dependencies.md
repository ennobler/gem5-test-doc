---
title: "Dependencies"
date: 2018-05-12T21:14:19-04:00
draft: false
weight: 20
---
gem5 runs on Linux and Mac OS X, but should be easily portable to other
Unix-like OSes. At times in the past gem5 has worked on OpenBSD and Microsoft
Windows (under Cygwin), but these platforms are not regularly tested. Cygwin in
particular is no longer actively supported; if you must run on a Windows host,
we recommend installing Linux (e.g., Ubuntu Server) under a VM and running gem5
there. Free virtualization solutions such as VirtualBox and VMware Player work
well for this usage.  Cross-endian support has been mostly added, however this
has not been extensively tested. Running a program with syscall emulation is
supported regardless of host/target endianess, however full-system simulation
may have cross-endian issues (ALPHA full-system is known not to work on big
endian machines).

#### Hardware
gem5 is largely agnostic about the hardware it runs on. However, there
are several considerations to keep in mind when running gem5:

  - A 64-bit platform is strongly preferred over a 32-bit platform.
    Simulating a platform with a significant amount of physical memory
    will require the ability to address that much memory from within the
    gem5 process. Specifically, a 32-bit platform will typically be
    limited to simulating platforms with roughly 1GB of physical memory.
    Also, many of the ISAs simulated by gem5 are 64 bit (e.g., x86-64,
    ARM aarch64 and Alpha), so simulating their operation on a 32-bit
    machine will incur additional slowdowns.

  - gem5's ISA support involves some very large auto-generated C++
    files, which can require up to 1GB for g++ to compile. If you intend
    to do parallel builds (using the scons "-j" flag), you may
    occasionally see significant slowdowns from paging if your system
    has less than 1GB per core. This constraint is particularly worth
    noting if you're configuring a VM to run gem5 under Windows, as
    suggested above.

  - Ideally you should choose a host with the same endianness as the ISA
    you will be simulating. gem5 does support cross-endian simulation,
    but this feature is not extensively tested. Cross-endian simulation
    works best in syscall emulation (SE) mode.

#### External tools and required versions

To build gem5, you will need the following software:

  - [g++](http://gcc.gnu.org/) version 4.8 or newer or clang version 3.1
    or newer.
  - [Python](http://www.python.org), version 2.6 - 2.7 (we don't support
    Python 3.X). gem5 links in the Python interpreter, so you need the
    Python header files and shared library (e.g.,
    /usr/lib/libpython2.6.so) in addition to the interpreter executable.
    These may or may not be installed by default. For example, on
    Debian/Ubuntu, you need the "python-dev" package in addition to the
    "python" package. If you need a newer or different Python
    installation but can't or don't want to upgrade the default Python
    on your system, see our page on [using a non-default Python
    installation](using_a_non-default_Python_installation "wikilink").
  - [SCons](http://www.scons.org), version 0.98.1 or newer. SCons is a
    powerful replacement for make. See
    [here](http://sourceforge.net/project/showfiles.php?group_id=30337)
    to download SCons. If you don't have administrator privileges on
    your machine, you can use the "scons-local" package to install scons
    in your m5 directory, or install SCons in your home directory using
    the '--prefix=' option. Some scripts require argparse, which is
    available by default in Python 2.7 and can be installed from PyPi
    for older versions.
  - [zlib](http://www.zlib.net), any recent version. For Debian/Ubuntu,
    you will need the "zlib-dev" or "zlib1g-dev" package to get the
    zlib.h header file as well as the library itself.
  - [m4](http://www.gnu.org/software/m4/), the macro processor.

The following is optional, but highly recommended:

  - [protobuf](https://code.google.com/p/protobuf/), version 2.1 or
    newer for trace capture and playback support.
  - [pydot](https://pypi.python.org/pypi/pydot), Pythons interface to
    graphviz, is needed for generation of graphical representations of
    the simulated system topology.

There a few utility scripts written in Perl, but Perl is not necessary
to build or run the simulator.

#### Included dependencies

Some packages which might be difficult to find or which were modified
for us in gem5 are included in the ext directory.

  - [libfdt](http://www.denx.de/wiki/U-Boot/UBootFdtInfo) -- provides
    support for flattened device tree "blob" files
  - [dnet](http://libdnet.sourceforge.net/) -- dnet provides a
    simplified, portable interface to several low-level networking
    routines.
  - [iostream3](https://github.com/madler/zlib/tree/master/contrib/iostream3)
    -- A C++ stream interface to the zlib library.
  - [libelf](http://www.mr511.de/software/english.html) -- ELF object
    file access library.
  - [PLY](http://www.dabeaz.com/ply/) -- PLY is an implementation of lex
    and yacc parsing tools for Python.
  - x11ksyms -- Keycodes from X11 for VNC support.
  - fputils -- Compiler-independent library for 80-bit floating point
    arithmetic
