---
title: "SCons"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

This page provides some tips on using M5's build system.

M5's build system uses [SCons](http://www.scons.org), a powerful
replacement for make (and autoconf, and ccache). SCons uses standard
Python to describe the software build process, enabling some very
sophisticated & complex build procedures.

This page is not an attempt to fully document the build system. Because
many people aren't familiar with SCons, and because gem5's build system
in some areas takes advantage of the complexity that SCons enables, this
page tries to collect useful pointers that might not otherwise be
obvious.

## SCons tips

  - As with make, you can build multiple targets by specifying them all
    on a single command line.
  - As with make, the "-j" option can be used to enable parallel builds
    (useful on multiprocessor hosts).

<!-- end list -->

  -
    For example, the following command will build both the optimized
    Alpha syscall emulation and debug MIPS syscall emulation targets
    using up to 4 concurrent processes:

`% scons -j 4 build/ALPHA/m5.opt build/MIPS/m5.debug`

  - You can print out all the generic SCons options using "`scons -H`".
  - If you specify a directory rather than a file, SCons will build all
    of the targets it knows about under that directory. Thus "`scons
    build/ALPHA`" will build all of the various ALPHA binaries and run
    the regression tests on them (since the regression test targets are
    in the subdirectory build/ALPHA/tests).
  - The `--debug=explain` and `--debug=tree` options are very useful for
    figuring out why SCons does (or doesn't) rebuild a target when you
    change a source file.

## gem5 build system tips

  - There are a number of command-line options you can set of the form
    "option=value", for example:

`% scons CC=gcc44 USE_MYSQL=False build/ALPHA/gem5.opt`

  -
    Almost all of these options are *sticky*, that is, if you specify
    them once, they remain in force for all future builds in that
    particular target (e.g., ALPHA) until they are changed explicitly in
    a future invocation. (The only non-sticky option is `update_ref`,
    which, when set, causes the results of any regression tests to
    overwrite the reference outputs in preparation for being committed
    as the new reference outputs.)

<!-- end list -->

  - You can print out all the gem5-specific build options using "`scons
    -h`".
  - You can define your own configurations beyond ALPHA, ARM, etc. These
    names are just the names of files in the build_opts directory
    containing option settings. You can generate a new configuration by
    specifying an predefined configuration (using `default=`) and
    overriding other options on the command line,
e.g.:

`% scons --default ALPHA USE_MYSQL=False build/ALPHA_NOSQL/gem5.debug`
`% scons --default ALPHA CC=mycc CXX=myc++ build/ALPHA_MYCOMPILERS/gem5.debug`

  -
    The configuration name is actually totally arbitrary, and need not
    include the ISA name or anything else:

`% scons --default MIPS build/FOOBAR/gem5.debug`

  - The build options for a particular configuration are stored in the
    `build/options` directory, so you can blow away your
    configuration-specific build directory ("`rm -rf build/ALPHA`")
    without losing the settings of your sticky options.
  - All the build state, including SCons state as well as any
    non-default build option settings, are stored under the build
    directory. Thus if you delete the entire build directory (""`rm -rf
    build`") then you are back at the state you were in when you first
    unpacked the source tree.
  - You can build binaries in locations other than the "build"
    subdirectory of your source tree. This feature can be useful in some
    circumstances, for example if you have a central NFS-mounted copy of
    your source tree, but you want to build that tree on multiple
    different hosts using the local disk of each host. (Supporting this
    model is another reason why all the bulid state is kept in the build
    directory and not in the root of the source tree.) See the comments
    at the top of the `m5/SConstruct` file.

