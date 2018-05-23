---
title: "Register Indexing"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

CPU register indexing in gem5 is a complicated by the need to support
multiple ISAs with sometimes very different register semantics (register
windows, condition codes, mode-based alternate register sets, etc.). In
addition, this support has evolved gradually as new ISAs have been
added, so older code may not take advantage of newer features or
terminology.

## Types of Register Indices

There are three types of register indices used internally in the CPU
models: relative, unified, and flattened.

### Relative

A **relative register index** is the index that is encoded in a machine
instruction. There is a separate index space for each class of register
(integer, floating point, etc.), starting at 0. The register class is
implied by the opcode. Thus a value of "1" in a source register field
may mean integer register 1 (e.g., "%r1") or floating point register 1
(e.g., "%f1") depending on the type of the instruction.

### Unified

While relative register indices are good for keeping instruction
encodings compact, they are ambiguous, and thus not convenient for
things like managing dependencies. To avoid this ambiguity, the decoder
maps the relative register indices into a **unified register space** by
adding class-specific offsets to relocate each relative index range into
a unique position. Integer registers are unmodified, and continue to
start at zero. Floating-point register indices are offset by (at least)
the number of integer registers, so that the first FP register (e.g.,
"%f0") gets a unified index that is greater than that of the last
integer register. Similarly, miscellaneous (a.k.a. control) registers
are mapped past the end of the FP register index space.

### Flattened

Unified register indices provide an unambiguous description of all the
registers that are accessible as instruction operands at a given point
in the execution. Unfortunately, due to the complex features of some
ISAs, they do not always unambiguously identify the actual state that
the instruction is referencing. For example, in ISAs with register
windows (notably SPARC), a particular register identifier such as "%o0"
will refer to a different register after a "save" or "restore" operation
than it did previously. Several ISAs have registers that are hidden in
normal operation, but get mapped on top of ordinary registers when an
interrupt occurs (e.g., ARM's mode-specific registers), or under
explicit supervisor control (e.g., SPARC's "alternate globals").

We solve this problem by maintaining a **flattened register space**
which provides a distinct index for every unique register storage
location. For example, the integer portion of the SPARC flattened
register space has distinct indices for the globals and the alternate
globals, as well as for each of the available register windows. The
"flattening" process of translating from a unified or relative register
index to a flattened register index varies by ISA. On some ISAs, the
mapping is trivial, while others use table lookups to do the
translation.

A key distinction between the generation of unified and flattened
register indices is that the former can always be done statically while
the latter often depends on dynamic processor state. That is, the
translation from relative to unified indices depends only on the context
provided by the instruction itself (which is convenient as the
translation is done in the decoder). In contrast, the mapping to a
flattened register index may depend on processor state such as the
interrupt level or the current window pointer on SPARC.

### Combining Register Index Types

Although the typical progression for modifying register indices is
relative -\> unified -\> flattened, it turns out that *relative vs.
unified* and *flattened vs. unflattened* are orthogonal attributes.
Relative vs. unified indicates whether the index is relative to the base
register for its register class (integer, FP, or misc) or has the base
offset for its class added in. Flattened vs. unflattened indicates
whether the index has been adjusted to account for runtime context such
as register window adjustments or alternate register file modes. Thus a
relative flattened register index is one in which the runtime context
has been accounted for, but is still expressed relative to the base
offset for its class.

A single set of class-specific offsets is used to generate unified
indices from relative indices regardless of whether the indices are
flattened or unflattened. Thus the offsets must be large enough to
separate the register classes even when flattened addresses are being
used. As a result, the unflattened unified register space is often
discontiguous.

## Illustration

![Gem5-regs.png](Gem5-regs.png "Gem5-regs.png")

As an illustration, consider a hypothetical architecture with four
integer registers (%r0-%r4), three FP registers (%f0-%f2), and two
misc/control registers (%msr0-%msr1). In addition, the architecture
supports a complete set of alternate integer and FP registers for fast
context switching.

The resulting register file layout, along with the unified flattened
register file indices, is shown at right. Although the indices in the
picture range from 0 to 15, the actual set of valid indices depends on
the type of index and (for relative indices) the register class as well:

|                          |                              |
| ------------------------ | ---------------------------- |
| **Relative unflattened** | Int: 0-3; FP: 0-2; Misc: 0-1 |
| **Unified unflattened**  | 0-3, 8-10, 14-15             |
| **Relative flattened**   | Int: 0-7; FP: 0-5; Misc: 0-1 |
| **Unified flattened**    | 0-15                         |

In this example, register %f1 in the alternate FP register file could be
referred to via the relative flattened index 4 as well as the relative
unflattened index 1, the unified unflattened index 9, or the unified
flattened index 12. Note that the difference between the relative and
unified indices is always 8 (regardless of flattening), and the
difference between the unflattened and flattened indices is 3
(regardless of relative vs. unified status).

## Caveats

  - Although the gem5 code is unfortunately not always clear about which
    type of register index is expected by a particular function,
    functions whose name incorporates a register class (e.g.,
    readIntReg()) expect a relative register index, and functions that
    expect a flattened index often have "flat" in the function name.
  - Although the general case is complicated, the common case can be
    deceptively simple. For example, because integer registers start at
    the beginning of the unified register space, relative and unified
    register indices are identical for integer registers. Furthermore,
    in an architecture with no (or rarely-used) alternate integer
    registers, the unflattened and flattened indices are (almost always)
    the same as well, meaning that all four types of register indices
    are interchangeable in this case. While this situation seems to be a
    simplification, it also tends to hide bugs where the wrong register
    index type is used.
  - The description above is intended to illustrate the typical usage of
    these index types. There may be exceptions that don't precisely
    follow this description, but I got tired of writing "typically" in
    every sentence.
  - The terms 'relative' and 'unified' were invented for use in this
    documentation, so you are unlikely see them in the code until the
    code starts catching up with this page.
  - This discussion pertains only to the \*architectural\* registers. An
    out-of-order CPU model such as O3 adds another layer of complexity
    by renaming these architectural registers (using the flattened
    register indices) to an underlying physical register file.

## Outstanding Questions

  - The SPARC code looks broken, in that the FP_Base_DepTag value is
    much smaller than NumIntRegs (see src/arch/sparc/registers.hh). Is
    it really broken, or is there some subtlety not captured here that
    allows it to work?

__NOTOC__

