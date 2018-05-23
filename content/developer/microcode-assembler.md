---
title: "Micro-code assembler"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

## Syntax

The assembler is responsible for taking listings similar to traditional
assembly and processing it into a set of python objects. The listing is
broken into sections which represent macroops or sections of a microcode
ROM. In each section, there can be assembler directives and microops.
Microops can be preceeded with labels which may be local, or in the case
of the ROM, global. The distinction is to allow local labels to be
duplicated between sections, but global labels to globally mark a
position to, for instance, branch into the ROM or amongst sections of
the
ROM.

`   local labels aren't really local to ROM sections at the moment --`[`Gblack`](User:Gblack "wikilink")` 19:52, 5 June 2007 (EDT)`

Micro assembly syntax allows directives which provide control over the
assembler (indirectly), and actual microop instantiations. The
directives are names preceeded by a "." and followed by their arguments.
The arguments should follow legal python syntax and start immediatelly
after the white space trailing the directive name until an unescaped
newline or semicolon, the two line terminators. The following are some
examples:

    .example "some text %s" % "other text", 5*4

    .harder_example python_variable \
            arguments on the next line

    .example_with_semicolon argument;

    .example_with_semicolon2 argument\; ;

Microops are defined similarly, but without the preceeding "." and with
optional labels in front of them. Labels are names followed by a colon.
There may be more than one label for a particular microop. Lables which
are intended to be global should be preceeded by the keyword "extern".
The following is an
    example:

    extern my_macroop: top_of_loop: add destReg, sourceReg, 4*bytes_in_word
                                    branch less_than, top_of_loop

To define a macroop, there are two options. These options correspond to
whether the macroop will be generated combinationally by the decoder, or
if the macroop will refer to code in the ROM. In the combinational case,
The syntax is:

    def macroop macroop_name
    {
       [microops and directives]
    };

A macroop object will be created and given the name macroop_name, and
then populated using the microops and directives in it's body.

In the ROM based case, the syntax is simpler and looks like the
following:

    def macroop macroop_name (target);

A macroop object will be created and given the name macroop_name, but
instead of being filled with microops, it will only be told what it's
target is which should be a label.

To define a section of the microcode ROM, use syntax like:

    def rom
    {
        [microops and directives]
    };

A new rom object will **not** be created. Instead, the microops and
directives defined in the body will be used to extend the contents of an
existing ROM object.

Comments are ignored by the assembler and have two forms of syntax.
Single line comments begin with a \# and continue to the end of the
line. Multiline comments begin with a /\* and extend to the following
\*/.

    #This is a single line comment

    label: some_microop #Comments can be here too

    /*
     * A
     * multiline
     * comment
     */

## ISA Parser Interface

In order to allow customizing the assembler to work with different ISAs,
a lot of the work of the assembler is actually performed by python
objects and classes passed in with the listing. There is both a
combinational and ROM based macroop class which are instantiated by the
assembler to hold the macroops it recognizes in the listing, and in the
combinational case, to support any assembler directives. Instances of
this class must provide a dict which maps from directive names to their
implementations. ROM macroops are instantiated and set to their target.
A ROM object is passed into the assembler and is filled with microops
which are defined in ROM sections.

Each of the microops the assembler should recognize need to have a
corresponding python class. The arguments of the microop are passed to
the constructor of the python class, and the object created is what is
stored in the macroops. A dict of the mappings between mnemonics and
these classes is also passed into the assembler. After each microop is
created, it's added into its container using the container's
add_microop method.

## Preprocessor

The microcode assembler could be outfitted with a preprocessor, if one
is needed. Because the assembler is instantiated from python and just
passed a string containing the listing, it does not need to be aware of
the preprocessor. Simple process the input string in whatever way fits
the need, and then send the resulting output to the assembler. A common
preprocessor may be developed which could follow nasm style syntax. gas,
the gnu assembler, doesn't use a preprocessor and instead relies on C's.
