---
title: "DynInst"
date: 2018-05-13T18:51:37-04:00
draft: false
---

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
[ExecContext](Execution_Basics#ExecContext "wikilink") interface. When
ISA instructions are executed, the DynInst is passed in as the
ExecContext, handling all accesses of the ISA to CPU state.

Detailed CPU models can derive from DynInst and create their own
specific DynInst subclasses that implement any additional state or
functions that might be needed. See src/cpu/o3/alpha/dyn_inst.hh for an
example of this.

