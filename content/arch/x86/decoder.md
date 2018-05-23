---
title: "Decoder"
date: 2018-05-13T18:15:42-04:00
draft: false
weight: 20
---

## Requirements

The decoder must handle:

  - Variable length instructions
  - Misaligned instructions
  - Instructions that span fetch buffer/cache line/page boundaries
  - Microcoded instructions
  - Instructions longer than native data types
  - Self modifying code
  - Externally modified memory and page mappings
  - Instructions beginning "inside" other instructions

## Current Design

The decoder process will happen in several stages.

First, the native variable length instructions will be translated into
"fixed width" instructions. These instructions will be too large to
represent with native data types, so they will be stored as structures.
This translator will be set up as a state machine which operates on one
byte at a time. This takes advantage of multiple bytes being ready at
once in a buffer for filling in multiple byte immediates, for instance.

The fields in the structure will contain:

  - A bitfield specifying what prefixes were present
  - What the opcode(s) was/were
  - The ModRM and SIB bytes
  - Any immediate values

Second, this structure will be passed into a decoder generated using the
isa_parser. This will require modifying the current decoding structure
and isa_parser so that they accept a structure with fields. It may also
require being able to swap out parts of the decoder to simulate
different decoding "modes". This step will produce a static_inst object
which has now been fully decoded.

At this point, the decoded instruction is passed back to the cpu. A flag
in the instruction indicates whether it's implemented by microcode. If
it isn't, the instruction is processed by the cpu as is. If it is, the
instruction is really a wrapper which contains other static_insts.
These internal microops actually implement their containing macroop, and
they are what is passed down the pipe. Once the sequence of microops are
finished, the macroop is discarded and decoding continues with the next
instruction.
