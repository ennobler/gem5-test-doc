---
title: "Introduction"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---


## What is gem5?

gem5 is a modular discrete event driven computer system simulator
platform. That means that:

1.  gem5's components can be rearranged, parameterized, extended or
    replaced easily to suit your needs.
2.  It simulates the passing of time as a series of discrete events.
3.  Its intended use is to simulate one or more computer systems in
    various ways.
4.  It's more than just a simulator; it's a simulator platform that lets
    you use as many of its premade components as you want to build up
    your own simulation system.

gem5 is written primarily in C++ and python and most components are
provided under a BSD style license. It can simulate a complete system
with devices and an operating system in **full system mode (FS mode)**,
or user space only programs where system services are provided directly
by the simulator in **syscall emulation mode (SE mode)**. There are
varying levels of support for executing Alpha, ARM, MIPS, Power, SPARC,
and 64 bit x86 binaries on CPU models including two simple single CPI
models, an out of order model, and an in order pipelined model. A memory
system can be flexibly built out of caches and crossbars. Recently the
Ruby simulator has been integrated with gem5 to provide even more
flexible memory system modeling.

There are many components and features not mentioned here, but from just
this partial list it should be obvious that gem5 is a sophisticated and
capable simulation platform. Even with all gem5 can do today, active
development continues through the support of individuals and some
companies, and new features are added and existing features improved on
a regular basis.

## Capabilities out of the box

gem5 is designed for use in computer architecture research, but if
you're trying to research something new and novel it probably won't be
able to evaluate your idea out of the box. If it could, that probably
means someone has already evaluated a similar idea and published about
it.

To get the most out of gem5, you'll most likely need to add new
capabilities specific to your project's goals. gem5's modular design
should help you make modifications without having to understand every
part of the simulator.

As you add the new features you need, please consider contributing your
changes back to gem5. That way others can take advantage of your hard
work, and gem5 can become an even better simulator.

## Quick Start

If you are just getting started with gem5, you can follow the steps
below or watch the following video to download, build, and run the code.
Refer back to the [documentation](documentation "wikilink") page for
many more details.

{{\#ev:youtube|SW63HJ0nW90|400|center|A youtube video of the
installation process on Ubuntu 12.04 64bit. Video resolution can be set
to 1080}}

### Getting a copy

gem5's source code is managed using the Mercurial revision control
system. More details on the repository structure and on Mercurial are on
the [repository](repository "wikilink") page. Assuming you have
Mercurial installed on your system, you can get your own copy of the
source repository by typing:

`hg clone `<http://repo.gem5.org/gem5>

### Getting Additional Tools and Files

The additional tools (and other platform dependencies) required to build
gem5 are discussed [here](Dependencies "wikilink").

If you want to run the full-system version (including the full-system
regression tests), you will also need to download the full-system files
(disk images and binaries). Kernels, disk images, and boot loaders for
Alpha, ARM, and x86 are available on the [Download](Download "wikilink")
page. SPARC disk images are available on the [with the OpenSPARC
Architecture
tools](http://www.opensparc.net/offers/OpenSPARCT1_Arch.1.5.tar.bz2).

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

### Building

gem5 uses the scons build system which is based on python. To build the
simulator binary, run scons from the top of the source directory with a
target of the form **build/<config>/<binary>** where **<config>** is
replaced with one of the predefined set of build parameters and
**<binary>** is replaced with one of the possible m5 binary names. The
predefined set of parameters determine build-wide configuration settings
that affect the behavior, composition, and capabilities of the binary
being built. These parameters include the ISA to be supported, the CPU
models to be compiled, and the coherence protocol Ruby should use.
Several example config files are available in the directory build_opts
and it is fairly easy to see what each parameter represents. We'll talk
about the build system in more detail later. Valid binary names are
gem5.debug, gem5.opt, gem5.fast, and gem5.prof. These binaries all have
different properties suggested by their extension. gem5.debug has
optimization turned off to make debugging easier in tools like gdb,
gem5.opt has optimizations turned on but debug output and asserts left
in, gem5.fast removes those debugging tools, and gem5.prof is built to
use with gprof. Normally you'll want to use gem5.opt. To build the
simulator in syscall emulation mode with ARM support, optimizations
turned on, and debugging left in, you would run:

`scons build/ARM/gem5.opt`

In your source tree, you'd then find a new **build/ARM/** directory with
the requested **gem5.opt** in it. For the rest of this chapter we'll
assume this is the binary you're using.

See the [Build System](Build_System "wikilink") page for more details on
building gem5 binaries.

### Running

Now that you've built gem5, it's time to try running it. An gem5 command
line is composed of four parts, the binary itself, options for gem5, a
configuration script to run, and then finally options for the
configuration script. Several example configuration scripts are provided
in the “configs/example” directory and are generally pretty powerful.
You are encouraged to make your own scripts, but these are a good
starting point. The example script we'll use is called se.py and sets up
a basic SE mode simulation for us. We'll tell it to run the hello world
binary provided in the gem5 source
tree.

`build/ARM/gem5.opt configs/example/se.py -c tests/test-progs/hello/bin/arm/linux/hello`

This builds up a simulated system, tells it to run the binary found at
the location specified, and kicks off the simulation. As the binary
runs, its output is sent to the console by default and looks like
this:

`gem5 Simulator System.  `<http://gem5.org>` `
`gem5 is copyrighted software; use the --copyright option for details.`

`gem5 compiled Jul 17 2011 19:16:28`
`gem5 started Jul 17 2011 19:18:16`
`gem5 executing on zizzer`
`command line: ./build/ARM/m5.opt configs/example/se.py`
`Global frequency set at 1000000000000 ticks per second`
`0: system.remote_gdb.listener: listening for remote gdb #0 on port 7000`
`**** REAL SIMULATION ****`
`info: Entering event queue @ 0.  Starting simulation...`
`Hello world!`
`hack: be nice to actually delete the event here`
`Exiting @ tick 3188500 because target called exit()`

You can see a lot of output from the simulator itself, but the line
“Hello world\!” came from the simulated program. Output files
generated from the simulation are put in the m5out directory, including
statistics in stats.txt.

In this example we didn't provide any options to gem5 itself. If we had,
they would have gone on the command line between gem5.opt and se.py. If
you'd like to see what command line options are supported, you can pass
the --help option to either gem5 or the configuration script. Note that
the two groups of options are different, so make sure you keep track of
whether they go before or after the configuration script.

See the [Running gem5](Running_gem5 "wikilink") page for more details on
running gem5 simulations.

## Asking for help

gem5 has two main mailing lists where you can ask for help or advice.
gem5-dev is for developers who are working on the main version of gem5.
This is the version that's distributed from the website and most likely
what you'll base your own work off of. gem5-users is a larger mailing
list and is for people working on their own projects which are not, at
least initially, going to be distributed as part of the official version
of gem5. Most of the time gem5-users is the right mailing list to use.
Most of the people on gem5-dev are also on gem5-users including all the
main developers, and in addition many other members of the gem5
community will see your post. That helps you because they might be able
to answer your question, and it also helps them because they'll be able
to see the answers people send you. To find more information about the
mailing lists, to sign up, or to look through archived posts visit
[Mailing Lists](Mailing_Lists "wikilink").

Before reporting a problem on the mailing list, please read [Reporting
Problems](Reporting_Problems "wikilink")

## What works

gem5 combines several different ISAs, system modes (SE or FS), CPU
models and memory models which all need to work together. Every
combination of these may not be fully tested or completely work. The
[Status Matrix](Status_Matrix "wikilink") describes the current status
of these combinations. Please send an e-mail to the mailing list if a
supported combination no longer works, or if you find that a combination
works that we're unsure of.
