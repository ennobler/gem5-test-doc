---
title: "ISA description system"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

The purpose of the M5 ISA description system is to generate a decoder
function that takes a binary machine instruction and returns a C++
object representing that instruction. The returned object encapsulates
all of the information (data and code) needed by the M5 simulator
related to that specific machine instruction. By making the object as
specific as possible to the machine instruction, the decoding overhead
is paid only once; throughout the rest of the simulator, the object
makes the needed instruction characteristics accessible easily and with
low overhead.

Because a typical commercial ISA has hundreds of instructions,
specifying the properties of each individual instruction in detail is
tedious and error prone. The ISA description (as specified by the
programmer) must take advantage of the fact that large classes of
instructions share common characteristics. However, real ISAs invariably
have not only several different instruction classes, but also a number
of oddball instructions that defy categorization. The goal of the M5 ISA
description system is to allow a human-readable ISA description that
captures commonalities across sets of instructions while providing the
flexibility to specify instructions with arbitrary characteristics. For
example, once the structure is in place to define integer ALU
instructions, the specification of simple instructions in that class
(e.g., add or subtract) should constitute a single, readable,
non-redundant line of the description file.

M5 uses a custom, domain-specific language to achieve this goal. The
language's syntax and features are specifically designed to make
instruction definitions compact and readable while minimizing
redundancy. The language's flexibility arises because the final code
generation is performed by user-provided functions written in a
general-purpose programming language (Python).

An ISA description written in this custom language is processed by a
parser to generate several C++ files containing class definitions and
the decode function. This file is in turn compiled into the M5
simulator.

The documentation for the ISA description system is divided into three
pages:

  - [Static instruction objects](Static_instruction_objects "wikilink")
    provides a basic familiarity with the structure of these C++
    objects, which are the final output of the decoding process. The
    goal of the ISA description system is to generate definitions for
    these objects, so you'll need to understand them before continuing.
  - [The M5 ISA description
    language](The_M5_ISA_description_language "wikilink") covers the
    description language proper.
  - [Code parsing](Code_parsing "wikilink") describes a library of
    Python classes and functions that automate the process of deducing
    an instruction's characteristics from a brief description of its
    operation. Although this library is not strictly part of the
    description language, it is used extensively by the Python code that
    generates the final C++ objects.
