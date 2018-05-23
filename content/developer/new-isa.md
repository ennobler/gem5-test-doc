---
title: "How to create an ISA"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

## arguments.hh

Move out of isa?

## faults.hh

ISA specific fault classes to be used in the ISA.

Functions I'd like to get rid of

  - genMachineCheckFault
  - genAlignmentFault

## interrupts.hh

### Interrupts class

#### Interrupt Level

*Shouldn't be capitalized.* Gets a request to set the interrupt level
and returns what the interrupt level was actually set to.

#### post

Request that an interrupt happen. Like asserting the interrupt line.

#### clear

Request that an interrupt not happen. Like deasserting a level triggered
interrupt line.

#### clear_all

*Breaks naming convention.* Clear all interrupts

#### check_interrupts

*Breaks naming convention.* *Not sure if this is right.* Check to see if
there are any interrupts pending

#### getInterrupt

Get an interrupt fault object to invoke. The Interrupts object should
internally prioritize interrupts so that they are handled in the right
order.

#### updateIntrInfo

*Breaks naming convention.* *Don't remember what this does. Need to find
the email from Ali.*

#### get_vec

*Breaks naming convention.* Returns the vector which describes the
pending(?) interrupts in SPARC which is available as a (memory mapped?)
register. This function panics in anything currently implemented other
than SPARC, and it breaks the abstraction of the Interrupts object. Its
existence allows necessary communication between two ISA specific
components but goes through the ISA agnostic cpu. An argument for this
function is that the cpu may need to modify the value returned by this
function, so it needs to be able to intercept values and change them.
This -really- breaks the abstraction of the Interrupts object by forcing
the cpu to know what the vector means and what values are appropriate.
This function needs to go away.

#### serialize/unserialize

Functions to serialize and unserialize the state in the Interrupts
object.

## isa_traits.hh

### MachineBytes

*Not sure if this is right* The number of bytes in a machine
instruction? If that's the case, this doesn't make sense for x86 and may
need to be handled differently.

### Set endianness

A "using" directive which selects which endianness conversion functions
are available.

### delay slot define

A preprocessor define which sets whether that architecture has delay
slots. This needs to go away.

### NoopMachInst

A machine instruction which encodes a noop

### DependenceTags

Offsets which separate the different register types from one another in
a flattened indexing space. It would be nice to phase these out.

### ZeroReg

The integer register which is defined by the architecture to always
contain the value zero. Not all ISAs actually have such a register, so
this needs to be done differently. This is simultaneously used as the
offset of the integer zero register and the floating point zero
register. This is correct in Alpha, but SPARC doesn't have a zero
floating point register, and not all architectures will necessarily have
the same index for both registers.

### ABI significant register indices

It's not clear how these should be defined for architectures like x86
which keep some of these values exclusively on the stack.

  - StackPointerReg
  - ReturnAddressReg
  - ReturnValueReg
  - FramePointerReg
  - ArgumentReg0
  - ArgumentReg1
  - ArgumentReg2
  - ArgumentReg3
  - ArgumentReg4
  - ArgumentReg5

### SyscallPseudoReturnReg

Second syscall return register

### MaxInstSrcRegs

*Maybe have the isa parser generate this value?* Maximum number of
source registers used by any instruction.

### MaxInstDestRegs

*Maybe have the isa parser generate this value?* Maximum number of
destination registers used by any instruction.

### LogVMPageSize, VMPageSize

Describes the size of a virtual memory page. Where is this used outside
of translation? Where is translation not ISA specific? If there is no
such place, maybe this should be defined internally to the ISA.

### SegKPMEnd SegKPBase

Not sure what these are for. If they only make sense for one ISA or are
used only in ISA specific places, they might not need to be part of the
ISA interface

### PageShift PageBytes

Not sure how this is different from VMPageSize and LogVMPageSize
respectively.

### BranchPredAddrShiftAmt

The number of bits a PC value should be shifted in order to provide a
compacted instruction addressing space. This makes no sense for x86 or
any other variable instruction width ISA.

### decodeInst

Prototype for the decodeInst function. Maybe this should be generated
automatically? Not a big deal.

### Full system stuff

#### LoadAddrMask

Yes, a mask for the address used by loads, but to produce an address
with what properties and why?

#### TLB stuff

Tlbs are ISA specific, so it's probably a good idea to put these in a
new ISA internal header file, or in the existing tlb.hh.

#### InterruptTypes

This might belong in sparc_traits.hh (SPARC specific ISA parameters),
or in interrupt.hh. There should be an alpha_traits.hh for analogous
Alpha specific ISA parameters

## kernel_stats.hh

Not sure

## locked_mem.hh

### handleLockedRead handleLockedWrite

Not sure what these do, although I'm sure it has to do with LL/SC.

## mmaped_ipr.hh

### handleIprRead handleIprWrite

These are for handling accesses to memory mapped registers. I believe
IPR is an Alpha-ism and a different name might be more appropriate. Are
these called from anywhere outside of ISA specific code? If not, they
should be moved out of the ISA interface.

## process.hh

'' Need to document exactly what this does and how.'' Defines an isa
specific process object.

## predecoder.hh

### getTC setTC

These should both be eliminated. A better mechanism should be used to
attach a predecoder to a thread context, perhaps just passing one in to
relevant functions.

### moreBytes

Feed more bytes to the predecoder.

### needMoreBytes

Returns whether or not the predecoder needs more bytes, or in other
words, whether you -don't- have left over data to start working with
immediately.

### extMachInstReady

Returns whether a whole ExtMachInst has been generated and is ready to
be used.

### getExtMachInst

Returns the ExtMachInst the predecoder has generated.

## regfile.hh

### readPC/setPC

Provide access to the current pc.

### readNextPC/setNextPC

Provide access to the next pc.

### readNextNPC/setNextNPC

Provide access to the next NPC. Most relevant for architectures with a
delay slot.

### clear

Blank register file contents.

### FlattenIntIndex

*Shouldn't be capitalized.* *Should link to an explanation of this whole
system.* Turn an architected register index into a canonical one.

### readMiscRegNoEffect/setMiscRegNoEffect

Provides access to a misc register without triggering ISA defined side
effects.

### readMiscReg/setMiscReg

Provides access to a misc register and triggers ISA defined side
effects.

### instAsid dataAsid

Alpha specificish but hasn't yet been removed.

### readFloatReg setFloatReg

Provides access to the floating point registers.

### readFloatRegBits setFloatRegBits

Provide access to the floating point registers as an integer. This
allows writing/reading floating point values as text without introducing
rounding error.

## remote_gdb.hh

### RemoteGDB object

*This might not be complete.* *Need to document these functions.*

#### acc

#### getregs

#### setregs

#### clearSingleStep

#### setSingleStep

## stacktrace.hh

### ProcessInfo

Class to describe a process? Is this for Alpha's process control blocks?
(or whatever those are called)

### StackTrace

## syscallreturn.hh

*Should there be an underscore in this file name?* Takes the generic
SyscallReturn value and a thread context and sets up state as if a
syscall was returning the value described by the SyscallReturn.

## tlb.hh

### lookup

Looks up a tlb entry. Is this used only internally by the tlb?

### ITLB DTLB

#### translate

Translates a request object

#### doMmuRegRead doMmuRegWrite

*For consistency, maybe these should be called readMmuReg and
setMmuReg?* Access an mmu control register

#### GetTsbPtr

*Shouldn't be capitalized* Is this part of the interface? Should look
again.

## types.hh

Typedefs for the following types.

  - MachInst
  - ExtMachInst
  - IntReg
  - LargestRead
  - MiscReg
  - FloatReg
  - FloatRegBits
  - AnyReg

<!-- end list -->

  - RegContextParam
  - RegContextVal

<!-- end list -->

  - RegIndex

## utility.hh

### inUserMode

Returns whether a thread context is operating in user mode. "User mode"
is a little vague, but not too vague to be useful.

### is(Callee|Caller)Save(Integer|Float)Register

Can these be purged? They don't seem to be used by anything.

### realPCToFetchPC fetchPCToRealPC

What are these used for?

### fetchInstSize

What is this used for? It doesn't make sense for x86 regardless.

### zeroRegisters

Writes zero to the "zero" registers. This really needs to go away, for
efficiency's sake if nothing else.

### initCPU

Initialize a cpu to the state it would have immediate after being
powered on.

### startupCPU

Actually activate a cpu to start "doing something".

## vtophys.hh

### kernel_pte_lookup

*Breaks naming convention.* What does this do, precisely?

### vtophys

*In what context is this supposed to be used? SE? FS?* Translates a
virtual address to a physical one.

