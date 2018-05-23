---
title: "Architectural State"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

## Registers

### Register types - float, int, misc

### Indexing - register spaces stuff

See [Register Indexing](Register_Indexing "wikilink") for a more
thorough treatment.

A "nickle tour" of flattening and register indexing in the CPU models.

First, an instruction has identified that it needs register such and
such as determined by its encoding (or the fact that it always uses a
certain register, or ...). For the sake of argument, lets say we're
talking about SPARC, the register is %g1, and the second bank of globals
is active. From the instructions point of view, the unflattened register
is %g1, which, likely, is just represented by the index 1.

Next, we need to map from the instruction's view of the register file(s)
down to actual storage locations. Think of this like virtual memory. The
instruction is working within an index space which is like a virtual
address space, and it needs to be mapped down to the flattened space
which is like physical memory. Here, the index 1 is likely mapped to,
say, 9, where 0-7 is the first bank of globals and 8-15 is the second.

This is the point where the CPU gets involved. The index 9 refers to an
actual register the instruction expects to access, and it's the CPU's
job to make that happen. Before this point, all the work was done by the
ISA with no insight available to the CPU, and beyond this point all the
work is done by the CPU with no insight available to the ISA.

The CPU is free to provide a register directly like the simple CPU by
having an array and just reading and writing the 9th element on behalf
of the instruction. The CPU could, alternatively, do something
complicated like renaming and mapping the flattened index further into a
physical register like O3.

One important property of all this, which makes sense if you think about
the virtual memory analogy, is that the size of the index space before
flattening has nothing to do with the size after. The virtual memory
space could be very large (presumably with gaps) and map to a smaller
physical space, or it could be small and map to a larger physical space
where the extra is for, say, other virtual spaces used at other times.
You need to make sure you're using the right size (post flattening) to
size your tables because that's the space of possible options.

One other tricky part comes from the fact that we add offsets into the
indices to distinguish ints from floats from miscs. Those offsets might
be one thing in the preflattening world, but then need to be something
else in the post flattening world to keep things from landing on top of
each other without leaving gaps. It's easy to make a mistake here, and
it's one of the reasons I don't like this offset idea as a way to keep
the different types separate. I'd rather see a two dimensional index
where the second coordinate was a register type. But in the world as it
exists today, this is something you have to keep track of.

## PCs

