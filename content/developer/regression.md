---
title: "Regression Tests"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

## Running Regressions

Running the M5 regression tests is the recommended way to make sure that
any changes to the simulator still comply with M5. Regression Testing is
performed using SCons to help guide which tests are run. All current
tests are located in the: tests/ directory. The command for running
regression tests has the following
    format:

    % scons build/<arch>/tests/<binary_type>/<test_directory>/<mode>/<test directory>/<ISA>/linux/<config script>

Here is an example of running all of the 'quick' regression tests for
the ALPHA architecture in syscall-emulation (SE) mode. You can leave out
any of the trailing details in the directory string to execute all of
the sub-tests.

    % scons build/ALPHA/tests/debug/quick/se
    ...
    **** build/ALPHA/tests/debug/quick/se/00.hello/alpha/linux/o3-timing PASSED!
    ...
    **** build/ALPHA/tests/debug/quick/se/00.hello/alpha/linux/simple-atomic PASSED!
    ...
    **** build/ALPHA/tests/debug/quick/se/00.hello/alpha/linux/simple-timing PASSED!
    ...
    **** build/ALPHA/tests/debug/quick/se/00.hello/alpha/tru64/o3-timing PASSED!
    ...

The script **util/regress** can be used for running regression tests.
Run the script with the option '-h' or '--help' for the options can be
passed to the script. This scripts executes both compiling (scons
build/\<build_opts (ISA)\>/gem5.\<compile-variants (opt,debug,etc.)\>)
and running regressions (scons
build/<ISA or build_opts>/tests/\<test-variants
(opt,debug,etc.)\>/<quick/long>/\<modes (se/fs)\>

The options contained in the test directory passed to scons are detailed
below.

  - <arch>: The build to run. This can be any file in gem5/build_opts
  - <binary_type>: The binary to run (e.g., opt, debug)
  - <test_directory>: "quick" or "long" see tests/
  - <mode>: se or fs
  - <test directory>: The directory in tests/<quick or long>/<se or fs>
    of the test to run. This usually specifies the workload to run
    (e.g., 00.hello)
  - <ISA>: Lowercase version of ISA to use
  - <config script>: Script from tests/configs to run

## Regression scripts execution description

The regression scripts are complicated. Below, we describe the
interaction and execution order of all of the scripts.

1.  util/regress executes scons with varying options depending on its
    parameters.
2.  scons is executed. For compile tests, this is the same way you
    compile gem5. For regression tests scons executes
    builds/<build to use>/gem5.<binary name> tests/run.py
    <crazy path that encodes everything>
      - Control is now transferred to tests/run.py
      - run.py execute the primary configuration script (the last
        directory on the scons command line)
      - The way scons knows which tests to run is by looking in
        tests/<quick or long>/<se or fs>/<secondary script>/ref/<ISA>/linux/
          -
            The name of this directory should match the primary script
            (in tests/configs) that you want to execute.
3.  tests/run.py executes the python file tests/configs/<name of dir>.py
4.  After the file in tests/configs executes to completion, run.py
    executes the secondary config file, or the file specific to this
    test
      -
        The file is found in
        tests/<quick or long>/<se or fs>/<secondary script>/test.py
        This file is usually used to specify the binary to simulate
5.  tests/run.py sets up some other parameters, like initializing CPUs.
6.  tests/run.py executes the function run_test(root)
      -
        Note: At some point in the files that tests/run.py executes, you
        must specify a variable "root"
        You can overload run_test(root) by specifying it in one of the
        executed files. These files are executed in python as if their
        code were contained in run.py.
7.  Usually, in run_test(root), gem5 simulation is started.
8.  The output is written to
    build/<build to use>/tests/<binary name>/<quick or long>/<se or fs>/<secondary script>/<ISA>/linux/<primary script>
9.  The output is compared to the reference in
    tests/<quick or long>/<se or fs>/<secondary script>/ref/<ISA>/linux/<primary script>

## Creating Your Own Regressions

Adding a regression test is done by adding a new directory to the
'm5/tests' directory and filling that directory up with the appropriate
reference files. Every regression test needs these files in order to
compare the current changes of M5 with the 'known working state:

  - simout - Standard Output Stream
  - simerr - Standard Error Stream
  - config.ini - M5 Configuration File
  - stats.txt - Statistics From M5 Simulation

One needs to run the M5 with the appropriate binary in order to get the
aforementioned simulation files. For example, a command line to collect
simulation data for a hello world regression test might
    be:

    % build/ALPHA/gem5.fast -re configs/example/se.py --cmd=tests/test-progs/hello/bin/alpha/hello

Creating the 'hello world' regression test directory and copying over
the reference output might then follow this command line sequence:

    % mkdir -p tests/quick/00.hello/ref/alpha/linux/simple-timing
    % cp config.ini stats.txt simout simerr tests/quick/00.hello/ref/alpha/linux/simple-timing

You then need to create the test configuration file (e.g.,
simple-timing.py in this case) in tests/configs, if it does not already
exist. This file executes during tests/run.py. In this file you must
create a variable named "root", and you can optionally create function
name run_test(root) which takes a single parameter (the root of the
system). This function overloads a simple version in tests/run.py

Next, you need to add your primary script name to the SConscript file in
tests (tests/SConscript). In this file there is a variable named
"configs". You need to add the name of you primary script to that list.
Without this scons will not recognize your new test.

    configs += ['simple-timing']

The last thing one needs to do is add a 'test.py' configuration file
that specifies the workload for the regression test. For this 'hello
world' example, the 'test.py' file would just
    contain:

    root.system.cpu.workload = LiveProcess(cmd='hello', executable = binpath('hello'))

This file gets executed during tests/run.py after the primary script is
executed.

Once the test.py is copied to the 'hello world' regression test
directory, the individual regression test can be run:

    % cp test.py tests/quick/00.hello
    % scons build/ALPHA/tests/debug/quick/00.hello/alpha/linux/simple-timing

Or, you can execute all of the tests in 00.hello by running:

    % scons build/ALPHA/tests/debug/quick/00.hello/

## Updating Regression Tests

Once you've made and verified changes to the simulator, you can update
any regression tests with the "update" option:

    % scons --update-ref build/ALPHA/tests/debug/quick/00.hello

