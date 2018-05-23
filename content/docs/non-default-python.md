---
title: "Using a non-default Python"
date: 2018-05-13T18:51:37-04:00
draft: false
---


The particular installation of the Python interpreter that executes the
SCons build scripts is also linked with M5 to execute runtime Python
simulation scripts. (This correspondence is necessary since some Python
code is shared between the build process and runtime execution.) As a
result, you can get M5 to link with a non-default installation of the
Python interpreter by using that instance of the interpreter to execute
the SCons scripts. Typically this means either:

1.  rearranging your PATH so that scons finds the non-default 'python'
    first (assuming that the version you want to use is called
    'python'), or
2.  explicitly invoking an alternative interpreter on the scons script,
    e.g.:

``% python2.4 `which scons` [scons args]``

If you're using a standard Python installation that's just not the
default version (e.g., Python 2.4 is installed as `/usr/bin/python2.4`,
but you have to invoke it as `python2.4` because `/usr/bin/python`
points to Python 2.3) then that's typically all there is to it.

However, if you built Python from source in some user directory, then it
gets trickier. You can still use this custom Python to run scons using
either of the techniques above, but you must also be sure that the same
Python interpreter and Python library modules can be located at runtime
too. Details of how to make this work depend on which of the (at least)
three different ways of building M5 & Python you select: dynamic Python
library with dynamic modules, static Python library with dynamic
modules, or static Python library with static modules.

### Dynamic (shared) Python library with dynamic modules

This approach works well if you'll be running M5 on the same system that
you build it on, or on a set of systems that have matching directory
structures (e.g., because your home directory is NFS-mounted in the same
place on all of them). The steps are:

1.  When compiling Python from source, make sure you add the
    `--enable-shared` argument to `configure` so that the shared-library
    version of the interpreter is built.
2.  When running M5, you need to make sure the loader can find your
    Python shared library, typically by adding the directory to your
    `LD_LIBRARY_PATH` environment variable.

Note that the paths to the Python library modules are hard-coded into
the interpreter when it is built, so once the shared library is found,
nothing additional needs to be done to tell Python where those modules
are.

### Static Python library with dynamic modules

This approach results in a larger M5 binary (since the Python
interpreter is statically linked), but avoids the need to set
`LD_LIBRARY_PATH`. However, Python library modules will still be loaded
dynamically, so those files must be present at the same locations on the
system where you run M5 as they were on the system where you built it.

There are two ways to force the linker to use the static library:

1.  Don't add the `--enable-shared` argument to `configure` when you
    build Python. The static Python library gets bulit no matter what.
    If the linker can't find the shared library, it will be forced to
    use the static version.
2.  Use the compiler/linker flag (e.g., `-static` for gcc/g++) to force
    all libraries to be linked statically.

You will probably also have to:

  - tweak the path to the Python library in the `m5/SConstruct` file
  - add some linker options so that Python symbols not needed at link
    time but needed later by dynamically loaded Python library modules
    don't get stripped out; see
    [1](http://www.python.org/doc/ext/link-reqs.html).

### Static Python library with static modules

In theory, this approach avoids the need to have any additional Python
files present when running M5. It could be useful if the system
installation on which you'll be running M5 is drastically different from
that of the system where you're building M5.

This approach builds on the static Python library approach above, but
further includes all of the Python modules needed to run your scripts in
the static Python library. This is accompished by editing the
Modules/Setup file in the Python source distribution before you build
Python. See the comments in that file for further instructions.

In practice, we haven't tried this. Please let us know if you try it,
whether you succeed or fail. I believe the static linking step will work
for compiled-code modules (.so's), but not for library modules written
in Python, so it's probably still not a complete solution for getting
everything into a single executable file.
