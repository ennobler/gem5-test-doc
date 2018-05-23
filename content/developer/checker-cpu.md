---
title: "Checker CPU"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

The Checker allows for dynamic verification of any CPUs that use
[DynInst](DynInst "wikilink"). Currently it does not support SMT or MP
systems. Extending the Checker for those systems should not be
difficult, but makes it impossible to verify memory value of loads.

## What it verifies

The Checker can verify:

  - Correct instruction path
  - Correct fetched instruction
  - Correct execution results
  - Correct PC redirection due to branches, faults, or PC events

## What it can't verify

The Checker cannot verify:

  - Correct interrupt handling
  - Store values being correct in memory
  - Certain instructions, such as RPCC (reads cycle counter), RC/RS
    (reads interrupt flags)

The Checker does not have access to the main CPU's interrupt signals, so
it can't ensure that the interrupts are detected and handled correctly.

Stores, although they commit and are issued to memory in program order,
may actually complete out of order. This is especially true with store
conditionals, which don't actually complete until the store conditional
result is written back. Because instructions are verified in program
order and only after they complete, the value in memory may not reflect
the store being verified but may instead be due to a younger (but also
committed) store.

Instructions such as RPCC will have different results by the time the
instruction has completed versus when it executed. Thus these
instructions must be marked as `IsUnverifiable` in the decoder.isa file.

## How it works

The Checker works by keeping a separate
[SimpleThread](SimpleThread "wikilink"), which it updates with its own
results of instruction fetching and execution. Once the main CPU
completes an instruction, it sends that instruction to the Checker. The
Checker verifies these instructions in program order. It ensures that
each instruction matches the instruction that the Checker fetches. It
does the same with the execution of the instruction, as well as any
faults that the instruction generates. In the cases of loads it checks
memory to verify the load's value matches what is in memory. This
verification can not be done on anything other than UP systems because
memory's value may have changed between the time the load executes and
the time the load completes.

The Checker defines a [ThreadContext](ThreadContext "wikilink"), called
CheckerThreadContext, which serves as a wrapper for the main CPU's
ThreadContext and its own SimpleThread. It takes any calls made to
CheckerThreadContext and forwards them to either: both the Checker and
the main CPU (in the case of setting registers); or just the main CPU
alone (in the case of reading registers). This allows the Checker to be
able to keep its state updated properly even when the main CPU's state
is updated externally (e.g. through a fault). It also allows
CheckerThreadContext to be used transparently in place of the main CPU's
ThreadContext because it will return the main CPU's state properly.

## How to use

For a CPU that uses DynInst, the following steps must be taken to be
able to use the Checker:

  - Add the Checker to its SimObject params.
  - Setup the Icache and Dcache ports of the Checker when they are
    created at Fetch and in the LSQ.
  - Store the result of any destination register write in the result
    variable inside DynInst.
  - Use the Checker's ThreadContext in place of its normal ThreadContext
    when the checker is enabled.
  - Add calls to tell the Checker when instructions have completed.
    Usually this is when the instruction is committed. However stores do
    not actually complete until their data is written back.
  - If the CPU modifies its own state anywhere inside its code (and not
    through its ThreadContext), be sure to call the right functions to
    keep the Checker up to date.

See the Detailed CPU for an example of using the Checker.

