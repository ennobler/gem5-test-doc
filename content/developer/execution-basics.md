---
title: "Execution Basics"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

## Predecoding

## StaticInsts

The StaticInst provides all static information and methods for a binary
instruction.

It holds the following information/methods:

  - Flags to tell what kind of instruction it is (integer, floating
    point, branch, memory barrier, etc.)
  - The op class of the instruction
  - The number of source and destination registers
  - The number of integer and FP registers used
  - Method to decode a binary instruction into a StaticInst
  - Virtual function execute(), which defines how the specific
    architectural actions taken for an instruction (e.g. read r1, r2,
    add them and store in r3.)
  - Virtual functions to handle starting and completing memory
    operations
  - Virtual functions to execute the address calculation and memory
    access separately for models that split memory operations into two
    operations
  - Method to disassemble the instruction, printing it out in a human
    readable format. (e.g. `addq r1 r2 r3`)

It does not have dynamic information, such as the PC of the instruction
or the values of the source registers or the result. This allows a 1 to
1 mapping of StaticInst to unique binary machine instructions. We take
advantage of this fact by caching the mapping of a binary instruction to
a StaticInst in a hash_map, allowing us to decode a binary instruction
only once, and directly using the StaticInst the rest of the time.

Each ISA instruction derives from StaticInst and implements its own
constructor, the execute() function, and, if it is a memory instruction,
the memory access functions. See
[ISA_description_system](ISA_description_system "wikilink") for
details about how these ISA instructions are specified.

## DynInsts

The DynInst is used to hold dynamic information about instructions. This
is necessary for more detailed models or out-of-order models, both of
which may need extra information beyond the
[StaticInst](StaticInst "wikilink") in order to correctly execute
instructions.

Some of the dynamic information that it stores includes:

  - The PC of the instruction
  - The renamed register indices of the source and destination registers
  - The predicted next-PC
  - The instruction result
  - The thread number of the instruction
  - The CPU the instruction is executing on
  - Whether or not the instruction is squashed

Additionally the DynInst provides the
[ExecContext](ExecContext "wikilink") interface. When ISA instructions
are executed, the DynInst is passed in as the ExecContext, handling all
accesses of the ISA to CPU state.

Detailed CPU models can derive from DynInst and create their own
specific DynInst subclasses that implement any additional state or
functions that might be needed. See src/cpu/o3/alpha/dyn_inst.hh for an
example of this.

## Microcode support

## ExecContext

The ExecContext describes the interface that the ISA uses to access CPU
state. Although there is a file `src/cpu/exec_context.hh`, it is purely
for documentation purposes and classes do not derive from it. Instead,
ExecContext is an implicit interface that is assumed by the ISA.

The ExecContext interface provides methods to:

  - Read and write PC information
  - Read and write integer, floating point, and control registers
  - Read and write memory
  - Record and return the address of a memory access, prefetching, and
    trigger a system call
  - Trigger some full-system mode functionality

Example implementations of the ExecContext interface include:

  - [SimpleCPU](SimpleCPU "wikilink")
  - [DynInst](DynInst "wikilink")

See the [ISA description](ISA_description_system "wikilink") page for
more details on how an instruction set is implemented.

## ThreadContext

ThreadContext is the interface to all state of a thread for anything
outside of the CPU. It provides methods to read or write state that
might be needed by external objects, such as the PC, next PC, integer
and FP registers, and IPRs. It also provides functions to get pointers
to important thread-related classes, such as the ITB, DTB, System,
kernel statistics, and memory ports. It is an abstract base class; the
CPU must create its own ThreadContext by either deriving from it, or
using the templated ProxyThreadContext class.

### ProxyThreadContext

The ProxyThreadContext class provides a way to implement a ThreadContext
without having to derive from it. ThreadContext is an abstract class, so
anything that derives from it and uses its interface will pay the
overhead of virtual function calls. This class is created to enable a
user-defined Thread object to be used wherever ThreadContexts are used,
without paying the overhead of virtual function calls when it is used by
itself. The user-defined object must simply provide all the same
functions as the normal ThreadContext, and the ProxyThreadContext will
forward all calls to the user-defined object. See the code of
[SimpleThread](SimpleThread "wikilink") for an example of using the
ProxyThreadContext.

### Difference vs. ExecContext

The ThreadContext is slightly different than the
[ExecContext](ExecContext "wikilink"). The ThreadContext provides access
to an individual thread's state; an ExecContext provides ISA access to
the CPU (meaning it is implicitly multithreaded on SMT systems).
Additionally the ThreadState is an abstract class that exactly defines
the interface; the ExecContext is a more implicit interface that must be
implemented so that the ISA can access whatever state it needs. The
function calls to access state are slightly different between the two.
The ThreadContext provides read/write register methods that take in an
architectural register index. The ExecContext provides read/write
register methdos that take in a [StaticInst](StaticInst "wikilink") and
an index, where the index refers to the i'th source or destination
register of that StaticInst. Additionally the ExecContext provides read
and write methods to access memory, while the ThreadContext does not
provide any methods to access memory.

## ThreadState

The ThreadState class is used to hold thread state that is common across
CPU models, such as the thread ID, thread status, kernel statistics,
memory port pointers, and some statistics of number of instructions
completed. Each CPU model can derive from ThreadState and build upon it,
adding in thread state that is deemed appropriate. An example of this is
[SimpleThread](SimpleThread "wikilink"), where all of the thread's
architectural state has been added in. However, it is not necessary (or
even feasible in some cases) for all of the thread's state to be
centrally located in a ThreadState derived class. The DetailedCPU keeps
register values and rename maps in its own classes outside of
ThreadState. ThreadState is only used to provide a more convenient way
to centrally locate some state, and provide sharing across CPU models.

## Faults

