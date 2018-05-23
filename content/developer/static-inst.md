---
title: "StaticInst"
date: 2018-05-13T18:51:37-04:00
draft: false
---

This section describes the static instruction objects. These objects are
what a decoder function instantiates for each machine instruction
fetched during simulation. It has information such as opcode, source and
destination register, immediate value, of the machine instruction it was
generated from.

The definitive documentation for these objects is associated with the
source code (particularly in static_inst.hh). The purpose of this
section is to provide an overview of the static instruction class
hierarchy and how it is used to factor the common portions of
instruction descriptions.

The instruction objects returned by the decoder during simulation are
instances of classes derived from the C++ class StaticInst, defined in
static_inst.hh. Because some aspects of StaticInst depend on the ISA,
such as the maximum number of source registers, StaticInst is defined as
a template class. The ISA-specific characteristics are determined by a
class template parameter.

The following code snippet is from Alpha's decoder function defined in
build/ALPHA_{FS|SE}/arch/alpha/decoder.cc. As the instruction decoded
is a load address instruction, the decoder initiates an object of class
Lda and returns it. Lda is a class derived from class StaticInst, and
its class definition is in in
build/ALPHA_{FS|SE}/arch/alpha/decoder.hh. This object is used
throughout the simulation to represent the particular machine
instruction at specific PC. Both of these files (decoder.cc and
decoder.hh) are generated from
[The_M5_ISA_description_language](The_M5_ISA_description_language "wikilink")
by [Code_parsing](Code_parsing "wikilink"), so they are under
subdirectory *build*, not *src*.

    StaticInstPtr
    AlphaISA::decodeInst(AlphaISA::ExtMachInst machInst)
    {
        using namespace AlphaISAInst;
        switch (OPCODE) {
        case 0x8:
          // LoadAddress::lda([' Ra = Rb + disp; '],{})
          return new Lda(machInst);
    ...

### StaticInst base class

The information contained in a StaticInst object includes:

  - a vector of flags that indicate the instruction's basic properties,
    such as whether it's a memory reference;
  - the instruction's operation class, used to assign the instruction to
    an execution unit in a detailed CPU mode;
  - the number of source and destination registers
  - the number of destination registers that are integer and floating
    point, respectively
  - the register indices of sources and destinations
  - a function to generate the text disassembly of the instruction
  - functions to emulate the execution of the instruction under various
    CPU models
  - additional information specific to particular classes of
    instructions, such as target addresses for branches or effective
    addresses for memory references.

Some of these items, such as the number of source and destination
registers, are simply data fields in the base class. When decoding a
machine instruction, the decoder simply initializes these fields with
appropriate values. Other items, such as the functions that emulate
execution, are best defined as virtual functions, with a different
implementation for each opcode. As a result, the decoder typically
returns an instance of a different derived class for each opcode, where
the execution virtual functions are defined appropriately. Still other
features, such as disassembly, are best implemented in a hybrid fashion.
Virtual functions are used to provide a small set of different
disassembly formats. Instructions that share similar disassembly formats
(e.g., integer immediate operations) share a single disassembly function
via inheritance.

### StaticInst class hierarchy

The class hierarchy is not visible outside the decoder; only the base
StaticInst class definition is exported to the rest of the simulator.
Thus the structure of the class hierarchy is entirely up to the designer
of the decoder module. At one extreme, a single class could be used,
containing a superset of the required data fields and code. The
execution and disassembly functions of this class would then examine the
opcode stored in the class instance to perform the appropriate action,
effectively re-decoding the instruction. However, this design would run
counter to the goal of paying decode overhead only once.

In the end, the structure of the class hierarchy is driven by practical
considerations, and has no set relationship to opcodes or to the
instruction categories defined by the ISA. For example, in the Alpha
ISA, an "addq" instruction could generate an instance of one of three
different classes:

1.  A typical register-plus-register add generates an instance of the
    Addq class, which is derived directly from the base AlphaStaticInst
    class.
2.  A register-plus-immediate add generates an instance of AddqImm,
    which derives from the intermediate IntegerImm class. IntegerImm
    derives directly from AlphaStaticInst, and is the base class of all
    integer immediate instructions. It adds an 'imm' data field to store
    the immediate value extracted from the machine instruction, and
    overrides the disassembly function to format the immediate value
    appropriately.
3.  An "addq" instruction that specifies r31 (the Alpha zero register)
    as its destination is a no-op, and returns an instance of the Nop
    class, regardless of whether it uses a reg-reg or reg-imm format.
