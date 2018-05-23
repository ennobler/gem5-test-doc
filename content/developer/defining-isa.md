---
title: "Defining an ISa"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

## Overview

First, make sure you have basic understanding of how an ISA description
generates instructions within the M5 framework. A good start is the [The
M5 ISA description language](The_M5_ISA_description_language "wikilink")
page.

For this example, we will be constructing an ISA called MyISA which will
just be a renamed version of the MIPS ISA. We will go through the steps
of creating the files and configuration opions for an M5 ISA
description.

Your new ISA description, MyISA, will need to generate correct
instructions for the different CPU models. More specifically, your MyISA
description will allow your MyISA architecture (analagous to
ALPHA,MIPS,SPARC,etc.) to be plugged into System-Call Emulation (SE) and
Full-System (FS) simulations of any M5 CPU Model.

## Syscall Emulation (SE) MyISA

### Creating the Files for MyISA

The correct place to insert your ISA files in M5 is in the src/arch
directory. In this directory, create another directory called 'myisa' to
keep your code. For this example, we will copy the relevant files needed
from the M5 MIPS ISA description.

    cd src/arch
    mkdir myisa
    cd myisa
    cp -r ../mips/*.hh ./
    cp -r ../mips/*.cc ./

The relevant files are as follows:

  - isa_traits.hh/cc - <explanation here>
  - regfile.hh/cc, regfile/\* - <explanation here>
  - process.hh/cc - <explanation here>
  - linux/\* - <explanation here>
  - isa/\* - <explanation here>
  - faults.hh/cc - <explanation here>

You need to make sure that all instances of MipsISA needs to be replaced
with MyISA in the files.

    perl -pe s/Mips/My/g ????

### Making M5 Recognize MyISA

  - m5/build_opts/MYISA_SE - Create this file that allows M5 to
    recognize MyISA as a Syscall Emulation build option. It's contents
    should contain:

<!-- end list -->

    TARGET_ISA = 'myisa'
    FULL_SYSTEM = 0

  - m5/src/arch/isa_specific.hh - Edit this file by adding a constant
    for MyISA and then adding MyISA to the \#define if/else structure.

<!-- end list -->

    ...
    #define ALPHA_ISA 21064
    ...
    #define MY_ISA 6400

    ...

    #if THE_ISA == ALPHA_ISA
        #define TheISA AlphaISA
    #elif THE_ISA == SPARC_ISA
        #define TheISA SparcISA
    ...
    #elif THE_ISA == MY_ISA
        #define TheISA MyISA
    #else
        #error "THE_ISA not set"
    #endif

  - m5/src/arch/myisa/SConsopts - Edit the file so that the SCons build
    system will recognize your ISA

<!-- end list -->

    Import('*')

    all_isa_list.append('myisa')

### MyISA Decoding & Instruction Object Creation - src/arch/MyISA/isa/\*

At this point, the next major component for defining your own ISA is
setting up the MyISA decoder. Please refer to the [ISA description
page](ISA_description_system "wikilink") for detailed specifics of the
instruction object decoding and construction process.

We'll go over the files you will be using for your MyISA description
here:

  - isa/\*
      - operands.isa
      - base.isa
      - decoder.isa
      - main.isa
      - includes.isa
  - isa/formats/\*
      - formats.isa
      - basic.isa
      - int.isa
      - branch.isa
      - control.isa
      - fp.isa
      - mem.isa
      - noop.isa
      - tlbop.isa
      - trap.isa
      - unimp.isa
      - unknown.isa
      - util.isa

### Testing MyISA

**Test That your decoder
    builds:**

    scons build/MYISA_SE/arch/MyISA/atomic_simple_cpu_exec.cc CPU_MODELS=AtomicSimpleCPU
