---
title: "Micro Code"
date: 2018-05-13T18:16:25-04:00
draft: false
weight: 40
---

## Microop Parameter Specialization

Microops are not, in general, written in an absolute sense. They are
templates which provide an implementation for a macroop, but need to be
specialized using the arguments of the original instruction. In
hardware, this appears to be handled in two different ways depending on
the origin of the microop. In the case of combinationally generated
microops, the instructions are actually generated with the correct
parameters in place. This is how the macroops in SPARC work. If the
microops come from the ROM, they are specilized afterwards by a special
hardware unit which operates on them before they pass on to the rest of
the pipeline. The "emulation environment", aka the particulars of the
instruction being emulated by the microop sequence, is stored and is
used by this specializer to get the values to substitute in. In
hardware, this means recognizing a bit pattern and replacing the value
with a different one on a match. In C++, this becomes more complicated.
There seem to be three options.

  - The instructions could be allocated with the correct parameters,
    similar to the combinational case. This has the benefit that the
    instruction is totally specialized for it's environment from the
    very start.
  - The instructions could be specialized after the fact by adding a new
    function. This lets you reuse instructions without having to fully
    reallocate them.
  - The instructions could be specialized by adding in an external blob
    of data which holds all the relevant parameters.

The simplest approach would be the first, but allocating a new microop
for every instruction from the ROM would likely have a fairly high
performance overhead. The later two approaches help with that somewhat
by letting you skip actually allocating memory, but they prevent you
from using more than one version of a microop at a time.

Immediate values are handled by making ROM based microops generate a few
combinational microops before actually going to the ROM. These push the
immediate parameters into certain microcode registers which the ROM code
is written to use.

A potential solution is to create an "EmulEnv" (or similar) class which
contains the emulation environment. That's passed into the macroop, and
then the pieces of it are used to construct the contained microops using
the standard "operand" mechanism. In the case where there is no
containing macroop (which we might not want anyway), the "emulation
environment" will exist only in python and its values will be used to
fill in the microops parameters directly.

## Microop Allocation

Microops are allocated in two different ways. When they're generated
combinationally, they can be set up by their containing macroop when
*it* gets allocated. Existing mechanisms which cache regular
instructions also cache the macroop, and the microops end up being saved
as well. With microops that come from the microcode ROM, microops aren't
stored with the macroop, and the caching mechanism doesn't automatically
work.

Also, the microop that should be generated from a particular microopc
will depend, probably significantly, on the macroop it was generated
for. This ties in with the above topic about parameter specialization,
and means that the ROM can't directly hold instatiations of it's
microops which would be a handy caching mechanism.

One potential solution would be for the ROM to have it's own cache of
microop instantiations. This wouldn't be as simple as in the regular
decode case because there aren't handy machine code representations to
work with, but the micropc might be used instead. Unlike regular
instruction memory, the same micropc should always refer to the same
instruction. Note that while this might seem like a reasonable
assumption, there are situations I can imagine someone at some point
might want to have dynamic microcode.

Another option would be to actually store the microops in the macroop
which requested them. The draw back here is that it's possible that
you're macroop might need a non constant and large set of microops to
implement itself, and it might be difficult to keep track of what's been
instantiated and what hasn't.

Yet another solution would be to try to work the microops into the main
cache somehow. This doesn't sound like a great idea since they really
aren't the same thing.

## Macroop Argument Specialization

This sounds like but is very different from microop parameter
specialization. This refers to generating groups of macroops which all
perform the same operation, but use different arguments from different
places including memory.

In order to simplify the system which specifies the microcode which
makes up instructions, a preprocessor will need to be implemented. This
will probably use syntax mimicking NASM's.

To allow the decoder to specify what variant of arguments to use, the
system of passing tags into the instruction format will be revamped. It
will work essentially the same, but instead of doing replacement on the
register arguments, it will need to take advantage of whatever the
microop parameter specialization mechanism is to set things up.
