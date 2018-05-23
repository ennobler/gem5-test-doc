---
title: "Looking to help? "
date: 2018-05-13T18:51:37-04:00
draft: false
---

This page contains ideas for projects to work on during the gem5 coding
sprint held in conjunction with HPCA 2017. For more information on the
sprint, see: <http://learning.gem5.org/tutorial/index.html>.

Below are three categories of projects: small, medium, and large. I
expect that only the "small" can be easily completed in an afternoon.
For medium projects, you should be able to get a good start on them in
an afternoon and finish up with a few extra hours of work after the
sprint. Large project ideas are unlikely to be finished within a few
days. These ideas are mainly for the community to get together and
discuss at the sprint, not to start coding.

### Small ideas
-------

#### Fix TLB warmup for x86 \[Jason\]

See <http://reviews.gem5.org/r/3474/>. This patch was a first stab at
fixing this problem, but it should be re-written and re-tested based on
Steve's feedback.

The problem this patch tries to solve is that when switching from the
atomic CPU to a timing CPU, the information in the TLB is lost. It is
very important to not lose this TLB information when using the atomic
CPU to warm up the system. By losing the TLB data, all experiments run
with warmup will experience many more TLB misses than expected.

Note: This requires using FS mode to test.

#### Modify EventWrapper to understand C++11 lambdas \[Jason\]

The current EventWrapper can only wrap a very simple function that takes
no parameters. However, I believe that C++11 supports passing pointers
to partially bound function. See the documentation for std::bind:
<http://en.cppreference.com/w/cpp/utility/functional/bind>.

With this feature, we should be able to delete most of the Event
subclasses throughout the codebase, too. The idea is that most of the
time the Event subclasses are only used to pass a single parameter from
where the event is created to the event callback function.

This project should be quick for someone familiar with C++11 lambdas and
early binding of parameters.

#### Develop some ISA instruction tests \[Many people\]

The goal of this project is to find out what is implemented correctly
and possibly find some bugs in our ISAs. For instance, a recent GCC
change exposed a bug in the x86 ISA.

The RISC-V insttest is an example of instruction tests that are already
in the codebase. It should be pretty quick to put together a simple test
which goes through some of the x86 and/or ARM instructions and
explicitly tests each instruction like the RISC-V example above.

A longer-term goal would be to automate this process so we have some
confidence that our implementation of the ISA is correct. Someone with
experience validating hardware may have ideas on how to do this.

#### Clean up serialization code \[Andreas S\]

Increase the code reuse (particularly container helpers).

#### CI smoke tests (faster than quick) \[Andreas S\]

Create a separate test classification to quickly check to see if a
change breaks anything major. Right now, contributors often just compile
gem5 and say "close enough". This project would create a new class of
tests (probably just move some tests around) so that developers can
quickly (less than 5 minutes) test to see if they have made any breaking
changes. This is also important for our upcoming CI changes.

#### Push in fixes to x86 KVM

See <http://m5-dev.m5sim.narkive.com/SUFVlG5b/gem5-dev-x86-kvm-status>.

There are a couple of patches on reviewboard that fix some problems with
the x86 KVM code. See <http://reviews.gem5.org/r/2557/> and
<http://reviews.gem5.org/r/2613>. It would be great for someone with
access to both AMD and Intel hardware (and maybe multiple versions of
Linux?) to test these and/or fix them up so they can be merged.

#### Unify PS/2 handling \[Andreas S\]

PS2 is implemented by both the i8042 and PL050 models, but almost no
code is shared. This should be an easy way to get your hands a little
dirty with gem5's current code.

#### Unifty PCI config interfaces

Many supported architectures (at least x86, ARM, and Alpha) implement
PCI. Some of the complexity has been hidden by the new PCI Host
abstraction, but we still need to do platform-specific port wiring. We
should unify this to make it possible for configuration scripts to reuse
more code between architectures.

#### Other ideas \[Tony\]

  - Adding checkpointing support to the GPU model
  - Properly supporting atomics
  - Add support for event-based scheduling in the GPU model, and
    FUPool-style functional units

### Medium sized projects
-----
#### Config cleanups \[Andreas S/Jason\]

Most of the configs are not very object-oriented. I think these can be
restructured in a much better designed system. Jason thinks a good
example is his SimpleFS scripts:
<https://github.com/powerjg/gem5/tree/devel/simplefs/configs/myconfigs>
Andreas likes this script:
<https://github.com/gem5/gem5/blob/master/configs/example/arm/fs_bigLITTLE.py>

Some ideas on how to do this:

  - move some of config/common/ to a m5.config name space.
  - move the DRAM controller specifics to common/dram
  - create some common CPU configs in common/cpu
  - make some better example scripts

#### Embed the generated system SVG in a web page \[Andreas H\]

It would be cool if you could interactively navigate through the system
and see the simulation results.

This should be fairly easy for anyone skilled in client-side scripting.
It may even be used to view incremental results while the simulation is
running.

#### Other ideas

  - New test binaries based on the LLVM test suite \[Andreas S\]
  - Mini-DSL for param overrides from the command line \[Andreas S\]
  - Proper support for pthreads in SE mode \[Andreas S\]
  - Implement a fast mode in the HDLCD controller to support graphical
    workloads (e.g., Android) in KVM \[Andreas S\]
  - Fixing the structure and design of the GPU coalescer \[Tony\]

### Long-term ideas to discuss
----

#### Syscall emulation on different platforms \[Andreas H\]

See <https://www.mail-archive.com/gem5-dev@gem5.org/msg21463.html>. When
compiling Linux syscall emulation mode on other operating systems (e.g.,
MacOS, FreeBSD) many of the syscall interfaces are different. This
causes compilation errors since the headers are not available on these
other systems. It is a good idea to take some time to talk about a
long-term solution to this problem.

#### Re-write the test infrastructure \[Jason\]

Jason really doesn't like how the current test infrastructure uses
SCons. I think it makes in incredibly hard to use. Plus, it makes it
difficult to integrate into CI systems (e.g., Jenkins).

A couple of "requirements". This is not an exhaustive list.

  - Incremental testing (don't test things for parts of the code that
    didn't change)
  - Functional-only tests
  - Easy to compare a subset of stats (e.g., all that matters is that
    the number of cache misses is the same)

#### Re-write the build system \[Jason/Andreas S\]

Andreas S. has taken some steps to replace SCons with cmake. Another
interesting option is bazel: <https://bazel.build/>, a new build system
from Google. This would be a rather huge undertaking, though.

A few requirements. Again, this list isn't exhaustive.

  - Split gem5's components into separate dynamically-linked objects
    (reduce the size of the gem5 binary)
  - Faster\!
  - Play nice with test and CI systems.

#### A binary-translation CPU \[Andreas H\]

A large-sized project for some crafty person out there. This would be
useful for fast-forwarding, much like the KVMCpu, but more portable. It
could, for example, be built on top of the Tiny Code Generator (TCG), as
it is BSD licensed. Another possible place to look for inspiration is
pydgin (https://github.com/cornell-brg/pydgin) which leverages Python
(and PyPy/RPython) to build a jit for binary translation.

Quite a big task, but also a very big contribution to gem5.
