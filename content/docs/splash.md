---
title: "Splash"
date: 2018-05-13T18:51:37-04:00
draft: false
---

It is possible to run the SPLASH-2 benchmarks on M5 in two different
ways, each with their own caveats.

## Using full-system mode

The most robust approach is to compile the benchmarks using a Pthreads
implementation of the PARMACS macros, then link with the standard Linux
Pthreads library and run this binary on M5 under full-system mode.

Advantages:

  - Realistic: you're getting the actual Linux thread scheduler to
    schedule your threads
  - Robust (in contrast to current SE-mode approaches... see below)
  - You can build a cross-compiler to compile the binaries on non-Alpha
    platforms (see [Using linux-dist to Create Disk Images and Kernels
    for
    M5](Using_linux-dist_to_Create_Disk_Images_and_Kernels_for_M5 "wikilink")...
    note that you don't need to build a kernel, just the
    cross-compiler).

Disadvantages:

  - CPU limits: the Tsunami platform we model only supports 4 CPUs,
    though we have patches to make that scale to 64 (see [Frequently
    Asked Questions\#How many CPUs can M5
    run?](Frequently_Asked_Questions#How_many_CPUs_can_M5_run? "wikilink")).
  - Overhead: you've got to download a disk image, get the binaries onto
    the disk image, boot Linux under M5, etc. This isn't nearly as bad
    as it sounds, but it's still extra work.

For a step by step guide on running the benchmarks in full-system mode
see [this
document](https://docs.google.com/View?id=dfkk59gg_1079rhf4bd5) or
follow the next steps:

### Download

Download [splash2](http://kbarr.net/splash2). Download patches for
SPLASH2 from [UDEL](http://www.capsl.udel.edu/splash/Download.html).

### Compiling

Follow the directions from the UDEL site. Most of the benchmarks will
compile fine. Some need additional massaging.

radiosity: descend into glibdumb and glibps and ‘make’.

volrend: unarchive and build the libtiff; I had to modify its makefile
by adding -DBSDTYPES in the CFLAGS.

### Copying

Mount the disk image used in simulation, for example

    sudo mount -o loop,offset=32256 linux-arm-ael.img /mnt/tmp

### Executing

The following are the basic "default" ways to run benchmarks. If you
compile for your host architecture you should be able to test these
directly.

barnes: ./BARNES \< input

fmm: uncompress the files in the inputs directory, then ./FMM \<
inputs/input.16384

ocean: ./OCEAN

radiosity: ./RADIOSITY -batch

raytrace: uncompess the files in the inputs directory. I had to run with
additional memory (probably a data sizing issue with 64-bit machinery):
./RAYTRACE -m64 inputs/car.env

volrend: Uncompress the files in the inputs directory, then ./VOLREND 1
inputs/head (omit the .den extension, it is automatically appended in
the code).

water-nsquared: ./WATER-NSQUARED \< input

water-spatial: ./WATER-SPATIAL \< input

## Using syscall-emulation mode
{{% notice warning %}}
Note this section is really out of date; who wants to use Tru64
{{% /notice %}}



There are two approaches to support SPLASH benchmarks in SE mode:

1.  Use the same Pthreads implementation of PARMACS that you can run in
    FS mode, and enhance SE mode to handle the necessary syscalls.
2.  Use a custom M5-specific PARMACS library, possibly coupled with
    M5-specific syscalls, to support only the thread management features
    needed by SPLASH.

Unfortunately for option 1, supporting general Pthreads applications in
SE mode is extremely difficult. Under both Tru64 and Linux, Pthreads
uses an extra "management thread" to perform some tasks, which means an
N-thread SPLASH application (which is what you'd typically want to run
on an N-CPU machine) really has N+1 threads, and suddenly you need a
thread scheduler in M5 to figure out which threads are runnable, assign
them to CPUs, maybe preempt one of them if all N+1 are runnable, etc.
Worse yet, the Linux Pthreads library uses a pipe to communicate from
the application threads to the management threads, requiring you to
implement poll() and add signal support (so you can deliver SIGIO to
threads), and lots of other nasty stuff. Frankly it's just not worth the
effort, given that the Linux kernel already has excellent
implementations of pipes and poll() and signals, and you can just run
that under FS mode (see above).

Option 2 is arguably the "right" way to support SPLASH applications
under SE mode. Your custom PARMACS macro implementation can assume
you'll never allocate more threads than CPUs, so you don't need any
thread scheduling in M5. This implementation could call existing
syscalls where appropriate, or call new M5-specific syscalls that are
added specifically for this purpose. This is what most existing
simulators that support SPLASH do.

Unfortunately neither of these environments currently exist in a clean
form. What does exist is a historical artifact that resulted from me
(Steve) trying to support option 1 for Tru64, i.e. SPLASH applications
compiled to the Tru64 Pthreads library. (This code actually predates
M5's support for Alpha Linux.) There is a lot of complex code in
`src/kern/tru64` that attempts to do this. Partway through I came to the
realization I mentioned in the paragraph above, that doing a complete
job would end up with me writing a full thread scheduler inside M5. At
that point I gave up and finished the job by switching to option 2,
adding M5-specific implementations of the remaining PARMACS macros that
were giving me trouble, using special M5 syscalls I added for that
purpose.

This code (including binaries) is available
[here](http://www.m5sim.org/dist/m5_benchmarks/v1-splash-alpha.tgz).
Feel free to use it, but be aware of the following caveats:

  - Because this code is based on Tru64, and it's not possible (to our
    knowledge) to build a gcc cross-compiler that targets Tru64, you
    can't compile new binaries without a native Alpha Tru64 system.
  - Because the code partly tries to support the N+1 thread Tru64
    Pthreads model, there may be situations where it doesn't work (e.g.,
    if you use large numbers of processors, or unusual inputs).
  - Some of the synchronization primitives use "magic" M5 system calls,
    so the synchronization overheads may not be realistic.

If you have trouble, see [this email
message](http://article.gmane.org/gmane.comp.emulators.m5.users/1365)
for more information.

In summary, your best bet is to use FS mode. If someone would like to do
an Linux-based "option 2" implementation for SE mode, that would be
terrific, and we would be happy to redistribute that with M5. However,
to date, that has not happened. Meanwhile, you're welcome to use the
Tru64 code, but be aware that it's got some issues.

