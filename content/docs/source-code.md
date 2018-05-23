---
title: "Tour of sourcecode"
date: 2018-05-13T18:51:37-04:00
draft: false
---

## Source Browsing Tools

The gem5 source code is browsable online via several methods. You can
browse the latest version or the developmen history in [our Mercurial
repository](http://repo.gem5.org/gem5), search the code using [our
OpenGrok installation](http://grok.gem5.org/), or look at the
[Doxygen-generated
documentation](http://www.gem5.org/docs/html/index.html) (note that the
[class list](http://www.gem5.org/docs/html/annotated.html) is perhaps
the most useful starting point).

Once you have your own local copy of the tree, you use other tools to
index and search that copy. Many gem5 developers use
[cscope](http://cscope.sourceforge.net), and have included a script in
`util/cscope-index.py` to generate a cscope index.

## Tour of the tree

  - <b>AUTHORS</b> - A list of people who have historically contributed
    to gem5.
  - <b>LICENSE</b> - The license terms that apply to gem5 as a whole,
    unless overridden by a more specific license.
  - <b>README</b> - Some very basic information introducing gem5 and
    explaining how to get started.
  - <b>SConstruct</b> - A part of the build system, as is the
    build_opts directory.
  - <b>build_opts</b> - holds files that define default settings for
    build different build configurations. These include X86_FS and
    MIPS_SE, for instance.
  - <b>configs</b> - Simulation configuration scripts which are written
    in python, described in more detail later. The files in this
    directory help make writing configurations easier by providing some
    basic prepackaged functionality, and include a few examples which
    can be used directly or as a starting point for your own scripts.
  - <b>ext</b> - Things gem5 depends on but which aren’t actually part
    of gem5. Specifically, dependencies that are harder to find, not
    likely to be available, or where a particular version is needed.
  - <b>src</b> - gem5 source code.
      - <b>arch</b> - ISA implementations.
          - <b>generic</b> - Common files for use in other ISAs.
          - <b>isa_parser.py</b> - Parser that interprets ISA
            descriptions.
          - <b>ISA directories</b> - The files associated with the given
            ISA.
              - <b>OS directories</b> - Code for supporting an ISA/OS
                combination, generally in SE mode.
              - <b>isa</b> - ISA description files.
      - <b>base</b> - General data structures and facilities that could
        be useful for another project.
          - <b>loader</b> - Code for loading binaries and reading symbol
            tables.
          - <b>stats</b> - Code for keeping statistics and writing the
            data to a file or a database.
          - <b>vnc</b> - VNC support.
      - <b>cpu</b> - CPU models.
      - <b>dev</b> - Device models.
          - <b>ISA directories</b> - Device models specific to the given
            ISA
      - <b>doxygen</b> - Doxygen templates & output
      - <b>kern</b> - Operating system specific but architecture
        independent code (e.g. types of data structures).
          - <b>OS directories</b> - Code specific to the given simulated
            operating system.
      - <b>mem</b> - Memory system models and infrastructure.
          - <b>cache</b> - Code that implements a cache model in the
            classic memory system.
          - <b>ruby</b> - Code that implements the ruby memory model.
          - <b>protocol</b> - Ruby protocol definitions.
          - <b>slicc</b> - The slicc compiler.
      - <b>python</b> - Python code for configuration and higher level
        functions.
      - <b>sim</b> - Code that implements basic, fundamental simulator
        functionality.
  - <b>system</b> - Low level software like firmware or bootloaders for
    use in simulated systems.
      - <b>alpha</b> - Alpha console and palcode.
      - <b>arm</b> - A simple ARM bootloader.
  - <b>tests</b> - Files related to gem5’s regression tests.
      - <b>configs</b> - General configurations used for the tests.
      - <b>test-progs</b> - "Hello world" binaries for each ISA, other
        binaries are downloaded separately.
      - <b>quick, long</b> - Quick and long regression inputs, reference
        outputs, and test specific configuration files, arranged per
        test.

<!-- end list -->

  - <b>util</b> - Utility scripts, programs and useful files which are
    not part of the gem5 binary but are generally useful when working on
    gem5.

## Style rules

All of the code in gem5 is expected to follow a set rules described in
our [style guide](Coding_Style "wikilink"). These rules make the code
more consistent which make it easier to read, maintain, and extend the
code. Specific coding style has been defined for things such as
[Indentation and Line
Breaks](Coding_Style#Indentation_and_Line_Breaks "wikilink"),
[Spacing](Coding_Style#Spacing "wikilink"), [Naming of Variables &
Classes](Coding_Style#Naming "wikilink"), and [M5 Status
Messages](Coding_Style#M5_Status_Messages "wikilink"). There are also
[Documentation Guidelines](Documentation_Guidelines "wikilink") which
you should follow which allow documentation to be generated
automatically using the Doxygen system.

## Generated files - where do they end up

## .m5 config files

If you would like to set some gem5 parameters to a value by default you
can create a `.m5` directory within your home directory and inside place
a file called `options.py`. Within this file you may set any gem5
command line option to a new default. For example placing
`options.stdout_file=simout` in `options.py` will result in the
simulators stdout always being re-directed to a file named simout.

