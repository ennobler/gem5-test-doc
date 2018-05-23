---
title: "Build System"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 10
---

## Build System

gem5's build system is based on SCons, an open source build system
implemented in Python. You can find more information about scons at
<http://www.scons.org>. The main scons file is called SConstruct and is
found in the root of the source tree. Additional scons files are named
SConscript and are found throughout the tree, usually near the files
they're associated with.

### Build targets

In gem5, scons build targets are of the form
<build dir>/<configuration>/<target>. For example:

`scons build/ARM/gem5.opt`

The <build dir> part of the target is a directory path that ends in
"build". Typically this is simply "build" by itself (as in the example),
but you can specify a directory called "build" located somewhere else
instead. All targets under the same build dir are assumed to be compiled
for the same host platform, and share the same [global
variable](#Global "wikilink") settings. These settings are stored in the
build/variables.global file.

The <configuration> part ("ARM" in the example above) selects a set of
compile-time build options that control simulator functionality, such as
the ISA used, which CPU models are included, which Ruby coherence
protocol to use, etc. These [per configuration
variables](#Per_Configuration "wikilink") are stored in separate files
in the build/variables directory. This location makes it possible to
clean out a build by deleting the configuration directory completely
(e.g., using 'rm -rf') without losing the variable settings.

The first time a particular configuration is built, the configuration
name is also used to determine the default per-configuration variable
settings by looking for a matching name in the build_opts directory. If
you would like to create a new configuration with a name not found
there, use the --default flag to tell scons which file in build_opts or
build/variables to use for the default.

The build target ("gem5.opt" in the example above) is typically the name
of the gem5 binary to build, which specifies the set of compiler flags
used. Currently supported versions are gem5.debug, gem5.opt, gem5.fast,
gem5.prof and gem5.perf.

  - **gem5.debug** has optimizations turned off. This ensures that
    variables won't be optimized out, functions won't be unexpectedly
    inlined, and control flow will not behave in surprising ways. That
    makes this version easier to work with in tools like gdb, but
    without optimizations this version is significantly slower than the
    others. You should choose it when using tools like gdb and valgrind
    and don't want any details obscured, but other wise more optimized
    versions are recommended.

<!-- end list -->

  - **gem5.opt** has optimizations turned on and debugging functionality
    like asserts and DPRINTFs left in. This gives a good balance between
    the speed of the simulation and insight into what's happening in
    case something goes wrong. This version is best in most
    circumstances.

<!-- end list -->

  - **gem5.fast** has optimizations turned on and debugging
    functionality compiled out. This pulls out all the stops performance
    wise, but does so at the expense of run time error checking and the
    ability to turn on debug output. This version is recommended if
    you're very confident everything is working correctly and want to
    get peak performance from the simulator.

<!-- end list -->

  - **gem5.prof** is similar to gem5.fast but also includes
    instrumentation that allows it to be used with the gprof profiling
    tool. This version is not needed very often, but can be used to
    identify the areas of gem5 that should be focused on to improve
    performance.

<!-- end list -->

  - **gem5.perf** also includes instrumentation, but does so using
    google perftools, allowing it to be profiled with google-pprof. This
    profiling version is complementary to gem5.prof, and can probably
    replace it for all Linux-based systems.

These versions are summarized in the following
table.

| Binary name | Optimizations | Run time debugging support | Profiling support |
| ----------- | ------------- | -------------------------- | ----------------- |
| gem5.debug  |               | X                          |                   |
| gem5.opt    | X             | X                          |                   |
| gem5.fast   | X             |                            |                   |
| gem5.prof   | X             |                            | X                 |
| gem5.perf   | X             |                            | X                 |

### Command line options

Scons will recognize the following command line options specific to
gem5.

| Option          | Effect                                                                              |
| --------------- | ----------------------------------------------------------------------------------- |
| \--colors       | Turn on colorized output                                                            |
| \--no-colors    | Turn off colorized output                                                           |
| \--default      | Override which existing build configuration or build_opts file to use for defaults |
| \--ignore-style | Disable style checking hooks                                                        |
| \--update-ref   | Update test reference outputs                                                       |
| \--verbose      | Print full tool command lines                                                       |

### Environment variables

The following environment variables are imported from the host
environment for use in
scons:

| Variable            | Use                                                                                 |
| ------------------- | ----------------------------------------------------------------------------------- |
| AS                  | Assembler command                                                                   |
| AR                  | Archive tool command                                                                |
| CC                  | C compiler command                                                                  |
| CXX                 | C++ compiler command                                                                |
| HOME                | User's home directory                                                               |
| LD_LIBRARY_PATH   | Path to search for library files at loading time                                    |
| LIBRARY_PATH       | Path to search for library files at linking time                                    |
| PATH                | Path to search for programs                                                         |
| PROTOC              | protobuf compiler command                                                           |
| PYTHONPATH          | Path to search for python files                                                     |
| RANLIB              | Ranlib command                                                                      |
| M5_CONFIG          | Where to look for the special ".m5" directory                                       |
| M5_DEFAULT_BINARY | The default build target which overrides the default default build/ALPHA/gem5.debug |

### Configuration variables

These configuration variables are used to control the way gem5 is built.
Some are global, affecting all configurations in a build directory, and
some only affect the configuration being built. Unlike the command line
options, these variables retain their value between invocations of
scons.

#### Global

| Variable         | Description                              | Default                                               |
| ---------------- | ---------------------------------------- | ----------------------------------------------------- |
| CC               | C Compiler                               | CC environment variable or value determined by scons  |
| CXX              | C++ Compiler                             | CXX environment variable or value determined by scons |
| BATCH            | Use batch pool for build and tests       | False                                                 |
| BATCH_CMD       | Batch pool submission command            | qdo                                                   |
| M5_BUILD_CACHE | Cache built objects in this directory    | False                                                 |
| EXTRAS           | Add extra directories to the compilation |                                                       |

#### Per Configuration

| Variable           | Description                                              | Default                                          | Exported as config/\*.hh |
| ------------------ | -------------------------------------------------------- | ------------------------------------------------ | ------------------------ |
| CP_ANNOTATE       | Enable critical path annotation capability               | False                                            | X                        |
| CPU_MODELS        | CPU Models                                               | AtomicSimpleCPU,InOrderCPU,O3CPU,TimingSimpleCPU |                          |
| PROTOCOL           | Coherence protocol for Ruby                              | MI_example                                      | X                        |
| SS_COMPATIBLE_FP | Make floating-point results compatible with SimpleScalar | False                                            | X                        |
| TARGET_ISA        | Target ISA: ALPHA, ARM, MIPS, POWER, X86                 | alpha                                            | X                        |
| USE_CHECKER       | Use checker for detailed CPU models                      | False                                            | X                        |
| USE_FENV          | Use \<fenv.h\> IEEE mode control                         | whether fenv.h was found on this host            | X                        |
| USE_POSIX_CLOCK  | Use POSIX Clocks                                         | whether posix clocks are available on this host  | X                        |

#### Setting configuration variable values

The first way you set configuration variable values is through the
configuration name you choose as part of the build target. This file is
loaded from build_opts and contains preset values for some of these
variables which configures the build as the file name suggests.

It is important to note that the values in the file corresponding to the
configuration you picked are -default- values and are only used if no
directory already exists with its own values already in place. Those
files are for defining reasonable starting points to configure gem5 to
behave the way you want it to, and are not intended to actively
configure a particular build.

If you want to change a value after your build and configuration
directory is already created, or if you want to override a value as it's
created, you can specify the new values on the command line. The syntax
is similar to setting environment variables at a shell prompt, but these
go after the scons command. For example, to build MESI_CMP_directory
protocol for an existing ALPHA build, you could use the following
command.

scons PROTOCOL=MESI_CMP_directory build/ALPHA/gem5.opt

It's often a good idea to add --help to the scons command line which
will print out all of the configuration variables and what their values
are. This way you can make sure everything is set up like you want, and
that you don't have any typos in any variable names. If everything is as
you expect, you can remove --help to actually start the build.

### Running regressions / Testing Your Build

gem5's regression system is built into SCons. That ensures that the gem5
binary is automatically rebuilt if necessary, and that tests are only
rerun if they might have different results.

The regression targets are all found under "tests" in the build
directory. The components of the path to a tests output files determine
which test to run and how to run it. These components are as
follows:

`tests/`<gem5 binary extension>`/`<test category>`/`<test name>`/`<architecture>`/`<operating system>`/`<configuration>`/`

You can leave out components of the path farther down, and scons will
automatically build all available tests that match the components that
were specified. To run all "quick" tests on the "opt" version of ALPHA
configuration of gem5, you can run the following command:

`scons build/ALPHA/tests/opt/quick`

The regression framework is integrated into the scons build process, so
the command above will (re)build ALPHA/gem5.opt if necessary before
running the tests. Also thanks to scons's dependence tracking, tests
will be re-run only if the binary has been rebuilt since the last time
the test was run. If the previous test run is still valid (as far as
scons can tell), only a brief pass/fail message will be printed out
based on the result of that previous test, rather than the full output
and statistics diff that is printed when the test is actually executed.

Regression tests are subdivided into two categories, "quick" and "long",
based on runtime. You can run only the tests in a particular category by
adding that category name to the target path, e.g.:

`% scons build/ALPHA/tests/opt/long`

Specific tests can be run by appending the test name:

`% scons build/ALPHA/tests/opt/quick/fs/10.linux-boot`

For more details, see [Regression Tests](Regression_Tests "wikilink").

A few of the "quick" category of tests should run without any additional
setup. Some tests depend on EIO support which is provided in the
"encumbered" repository and which has to be built into gem5 using the
EXTRAS mechanism. Other tests depend on system files like particular
disk images and kernels to be found in particular places on your system.
These files are frequently available, and with a little effort and those
files you should be able to get those tests running. Other tests,
typically based on SPEC benchmarks and not part of "quick", require
input files which have a restrictive license and can't be distributed by
us. You won't be able to run these tests unless you have a license and
can get these files somehow.

### Adding source files and debug flags

Files are added to the build by declaring them inside SConscript files
as instances of certain python classes. The build system knows how to
handle those files based on what particular class was used. For
instance, to add a C++ source file foo.cc to the build, you could add
the following line to the SConscript in the same directory as foo.cc:

`Source('foo.cc')`

The build system finds and processes SConscript files automatically, so
you can create one near the files your adding or extend one that's
already there. The following table shows what types of source files
there are and what they're
for.

| Source file type | Description                                                                                      | Parameters |
| ---------------- | ------------------------------------------------------------------------------------------------ | ---------- |
| Name             | Description                                                                                      |
| PySource         | Add a python source file to the named package                                                    | package    |
| source           | The name of the python source file                                                               |
| SimObject        | Add a SimObject python file as a python source object and add it to a list of sim object modules | source     |
| Source           | Add a c/c++ source file to the build                                                             | source     |
| Werror           | Whether to compile with -Werror. Defaults to True.                                               |
| main             | This file is part of main() for the gem5 binary. Defaults to False.                              |
| skip_lib        | Whether this file should be excluded from the library version of gem5. Defaults to False.        |
| UnitTest         | Add a unit test to the build                                                                     | sources    |

A similar mechanism is used to define debug/trace flags. These look very
similar but don't actually refer to a file. A compound trace flag is a
flag which controls a group of normal trace
flags.

| Trace flag type | Description                                                                                  | Parameters |
| --------------- | -------------------------------------------------------------------------------------------- | ---------- |
| Name            | Description                                                                                  |
| DebugFlag       | Add an individual trace flag to the build.                                                   | name       |
| desc            | A description for this flag. This is optional but recommended.                               |
| CompoundFlag    | Add a compound trace flag to the build.                                                      | name       |
| flags           | A list or tuple of the names of trace flags which this new compound trace flag will control. |
| desc            | A description for this flag. This is optional but recommended.                               |

### Using EXTRAS

The [EXTRAS](Extras "wikilink") scons variable can be used to build
additional directories of source files into gem5 by setting it to a
colon delimited list of paths to these additional directories. EXTRAS is
a handy way to build on top of the gem5 code base without mixing your
new source with the upstream source. You can then manage your new body
of code however you need to independently from the main code base.

