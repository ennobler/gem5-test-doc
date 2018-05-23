---
title: "Adding Files to Build"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

The `EXTRAS` SCons option is a way to add functionality in gem5 without
adding your files to the gem5 source tree. Specifically, it allows you
to identify one or more directories that will get compiled in with gem5
as if they appeared under the 'src' part of the gem5 tree, without
requiring the code to be actually located under 'src'. It's present to
allow user to compile in additional functionality (typically additional
SimObject classes) that isn't or can't be distributed with gem5. This is
useful for maintaining local code that isn't suitable for incorporating
into the gem5 source tree, or third-party code that can't be
incorporated due to an incompatible license. Because the EXTRAS location
is completely independent of the gem5 repository, you can keep the code
under a different revision control system as well.

The main drawback of the EXTRAS feature is that, by itself, it only
supports adding code to gem5, not modifying any of the base gem5 code.
If you are changing the base gem5 code (instead of or in addition to
adding code with EXTRAS), see [Managing Local Changes with Mercurial
Queues](Managing_Local_Changes_with_Mercurial_Queues "wikilink").

One use of the EXTRAS feature is to support EIO traces. The trace reader
for EIO is licensed under the SimpleScalar license, and due to the
incompatibility of that license with gem5's BSD license, the code to
read these traces is not included in the gem5 distribution. Instead, the
EIO code is distributed via a separate "encumbered"
[repository](repository "wikilink").

The following examples show how to compile the EIO code. By adding to or
modifying the extras path, any other suitable extra could be compiled
in. To compile in code using EXTRAS simply execute the following

`scons EXTRAS=/path/to/encumbered build/ALPHA/gem5.opt`

In the root of this directory you should have a SConscript that uses the
`Source()` and `SimObject()` scons functions that are used in the rest
of M5 to compile the appropriate sources and add any SimObjects of
interest. If you want to add more than one directory, you can set EXTRAS
to a colon-separated list of paths.

Note that EXTRAS is a "sticky" parameter, so after a value is provided
to scons once, the value will be reused for future scons invocations
targeting the same build directory (`build/ALPHA_SE` in this case) as
long as it is not overridden. Thus you only need to specify EXTRAS the
first time you build a particular configuration or if you want to
override a previously specified value. For more information on sticky
scons options, see the [SCons build
system](SCons_build_system "wikilink") page.

To run a regression with EXTRAS use a command line similar to the
following:

`./util/regress --scons-opts="EXTRAS=/path/to/encumbered" -j 2 quick`
