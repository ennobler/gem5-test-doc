---
title: "Execution Tracing"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

## Tracing Interface

Each cpu/thread has a "tracer" sim object. This defaults to an object
which will write to the the same file descriptor as DPRINTFs. If you
want, you can nullify the tracer to disable tracing for that cpu/thread.
For output you want to be interleaved, use the same trace object (or
maybe file?) You could also set up several different files to catch
information from each cpu/thread.

There would be other tracer objects you could use instead if you wanted
to do something else at each instruction boundary. You could, for
instance, create a tracer object that synced with legion. You could also
create one that works with statetrace.

## Configuration

If there are tracer objects in the python script, it becomes harder for
the main m5 executable to set tracing parameters on the command line in
a uniform way. Maybe these options would manipulate the default tracer
object? That way, you would get the same behavior you're used to
(mostly) but still have the ability to change things up if you wanted
to.

## Microcode

It would be very nice to be able to trace both macroops and microops
simultaneously and separately. Right now, the microops are the only
thing that's traced because they're the only instructions that are
executed or commit. It would be ideal to be able to trace macroops only
to see the macroop disassembly, microops only like now, or an
interleaving of the two. The later option would be useful to see what
was going on, but still have everything in the context of the original
program. Currently, this isn't possible because there is no link from a
microop back to the macroop that generated it, and the macroops get
thrown out before they could be traced. It would be nice to print the
macroop before any block of microops that come from it, even if the
instruction doesn't commit. That way if it doesn't commit, you can still
tell where the instructions came from. To see that the macroop actually
finished, the final microop could be marked in the disassembly somehow.

For instance:

    ADD [$RAX+4], $RCX
    ---- ld t1, DS:[0*t0+rax+4]
    ---- add t1, t1, rcx
    ==== st t1, DS:[0*t0+rax+4]

or

    ADD [$RAX+4], $RCX {
        ld t1, DS:[0*t0+rax+4]
        add t1, t1, rcx
        st t1, DS:[0*t0+rax+4]
    }

I prefer the first because all the "}" lines could take considerable
space, and because non-microcode instructions still fit in easily.

