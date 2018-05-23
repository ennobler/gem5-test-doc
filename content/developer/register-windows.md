---
title: "Register Windows"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

Register windows prove to be a bit problematic with the current model of
StaticInsts and the read/write register methods. The issue is that based
on the register's index, it is possible that it will be one of the
registers whose actual index is determined by a combination of its index
and the Current Window Pointer (CWP). In SPARC, r0-r7 are non-windowed
registers, while r8-r31 are windowed registers.

## StaticInst

Register windows provide a form of dynamic renaming that is troublesome
with our current StaticInst. Because the CWP's value has nothing to do
with the current instruction, it is impossible to statically decode the
registers in use based on the machine instruction alone. Thus renaming
of registers must happen dynamically, either within the CPU's readReg()
or writeReg() function, or in the instruction's execute() function. The
latter is difficult because the current readReg() and writeReg()
functions refer to a source or destination index, and not the
architectural register index.

## SimpleCPU

Windowing will be built into the SPARC IntRegFile object, which
SimpleCPU will use via CPUExecContext. On writes to the CWP, a callback
from the MiscRegFile object to the CPUExecContext (e.g.,
changeIntRegContext()) will be reflected by the CPUExecContext back down
to the IntRegFile so that it can update its internal notion of the CWP.

changeIntRegContext() will be an ISA-independent function. Presumably it
will take as an argument a pointer to some ISA-specific object (derived
from an ISA-independent base object) that describes the change (sort of
like the new Fault objects)... unless anyone has a better idea.

  -
    I think a better parameter would be an index which specified what
    part of the context you were changing and a new value. Otherwise,
    all of the architectures would have to use a constraining (or
    trivial) base class for their integer register file context, and
    keep around the value of the parts they don't want to change. An
    example would be changing the cwp without changing between sets of
    global registers.

\--[Gblack](User:Gblack "wikilink") 13:04, 6 April 2006 (EDT)

## Detailed CPU

In the detailed CPU case, register window renaming will happen prior to
the normal rename. The initial arch reg index will be renamed by adding
the appropriate register window shift to them. Then normal renaming can
happen given the unique, physical register index. This will allow all
current dependence mechanisms to work normally. The first rename must
happen at decode/rename. A CWP register at decode or rename will get
speculatively updated on SAVE and RESTORE instructions. It will have to
be restored upon squashing.

In this case, the MiscRegFile callback changeIntRegContext() will go to
the proxy ExecContext, which can forward it up to the CPU object to
update its internal CWP. The fact that this update has to be done at
decode/rename (*before* execute) is unresolved... likely we will have to
introduce a new virtual StaticInst method to capture decode/rename-time
execution effects.

Adding this function raises several questions (that probably ought to
have their own wiki page):

  - In the common case where there's nothing to do, is it faster to have
    a null virtual function or a bool that suppresses calling the
    function?
  - Are there other behaviors currently hard-coded into the pipeline (or
    implemented via flags) that could be moved to this function?
  - Should we do the same thing for commit (create a virtual function
    and move some existing code there)? I think probably yes...

