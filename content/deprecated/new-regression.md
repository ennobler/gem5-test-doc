---
title: "New regression framework"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

We'd like to revamp the regression tests by moving to a new framework.
This page is intended to host a discussion of features and design for
the new framework.

## Ali's plan for a new implementation

  - Use [pytest](http://pytest.org/latest/)
  - It has, by far, the best documentation of any of the python testing
    frameworks and seems to be the most active
  - Seems to be completely extensible via python plugins and
    [hooks](http://pytest.org/latest/plugins.html#well-specified-hooks)
  - Supports outputting JUnit XML incase we want to use a continuous
    integration solution such as [Jenkins](http://jenkins-ci.org/) or
    [Hudson](http://hudson-ci.org/)
  - The [pytest xdist](http://pytest.org/latest/xdist.html#xdist) and
    [plugin](http://pypi.python.org/pypi/pytest-xdist) support running
    tests on multiple-cpus or multiple machines
  - Good collection of [tasks and
    tutorials](http://pytest.org/latest/talks.html#tutorial-examples-and-blog-postings)

### How things would work

  - [Marks](http://pytest.org/latest/mark.html) may be assigned to tests
    either with [python
    decorators](http://www.python.org/dev/peps/pep-0318/) or a class
    attribute if we want to stay python 2.5 compatible
      - The decorators would probably include cpu model, memory system,
        ISA, mode, and run length.
      - We might want to use pytest_addoption to be able to pass lists
        specifically for each of the decorators and generate tests that
        match appropriately with
        [this](http://pytest.org/latest/example/parametrize.html)
      - Alternatively we could use
        [pytest-markfiltration](http://pypi.python.org/pypi/pytest-markfiltration/0.4)
        although the syntax can be rather contrived

### Outstanding Questions

  - How would we do test discovery?
      - pytest will search py files looking for tests
      - Files can match a pattern, classes in files can match a pattern
        or functions can match a pattern
      - or it can only match things that inherit from Python UnitTest
  - Should we use [xunit](http://pytest.org/latest/xunit_setup.html)
    style or [func args](http://pytest.org/latest/funcargs.html) style
    setups?
  - Should we have a class that inherits from Python.UnitTest and does
    the heavy lifting or should we have a completely separate class that
    does the heavy lifting and use a factory class to create a bunch of
    instances of the seperate class?
  - Should gem5 be called as a library or on the command line?
  - How should we store output files? Same way we do now? should each
    directory just have a __init__.py and then the tests can be
    referred to as long.linux_boot.arm.linux.o3?

## Desirable features

  - Ability to add regressions via EXTRAS
      - For example, move eio tests into eio module so we don't try to
        run them when it's not compiled in
      - If we used py.test on the build directory that would find the
        eio code or whatever else is in there and pick them up. This
        could be through any of the methods described above
        --[Saidi](User:Saidi "wikilink") 18:09, 22 August 2011 (PDT)
  - Ability to not run regressions for which binaries or other inputs
    aren't available
      - This could be done easily based on the BaseClass that we use and
        tell about the file dependencies. e.g. `if not dependencies:
        pytest.skip("test binaries unavailable")`
        --[Saidi](User:Saidi "wikilink") 18:09, 22 August 2011 (PDT)
      - With maybe some nice semi-automated way of downloading binaries
        when they're publicly available
      - Perhaps we should make use of the large file extensions to
        mercurial or have a test binaries repository??
  - Graph run time, performance results over time

## Headline text

  - Better categorization of tests, and ability to run tests by
    category, e.g.:
      - by CPU model
      - by ISA
      - by Ruby protocol
      - by length
      - I believe these can all be satisfied via decorators and command
        line options we define --[Saidi](User:Saidi "wikilink") 18:09,
        22 August 2011 (PDT)
  - More directed tests that cover specific functionality and complete
    faster. Running spec benchmarks is important but spends a lot of
    time doing the same thing over and over. Those should only be a
    component of our testing, not almost all of it like it is now. This
    is a desirable feature of our testing strategy, not necessarily
    something that impacts the regression framework.
      - This is future work. needed for sure, but a bit orthoginal
        --[Saidi](User:Saidi "wikilink") 18:09, 22 August 2011 (PDT)
  - Better checkpoint testing
      - I believe we can create dependent tests, although it's an open
        question exactly how this interacts with the xdist stuff.
        --[Saidi](User:Saidi "wikilink") 18:09, 22 August 2011 (PDT)
      - some of this doesn't really depend on the regression framework,
        just needs new tests
      - e.g., integrating util/checkpoint-tester.py
          - I would rather duplicate/steal the functionality into a Base
            class for regression testing
            --[Saidi](User:Saidi "wikilink") 18:09, 22 August 2011 (PDT)
  - Support for random testing (e.g., for background testing processes)
      - Random latencies?
      - Random testing a la memory testers but with different seeds,
        longer intervals
          - I don't think there is anything preventing us from calling
            random.randint() in a test, and it's all python
            --[Saidi](User:Saidi "wikilink") 18:09, 22 August 2011 (PDT)
  - Decouple from SCons
      - Avoid having scons dependency bugs force unnecessary re-running
        of tests, particularly for update-refs
          - not quite sure how we would do updaterefs with py.test. I've
            been using a 4 line bash script anyway, because I don't want
            to update every test, only the failed ones
            --[Saidi](User:Saidi "wikilink") 18:09, 22 August 2011 (PDT)
      - Don't rely on scons to run jobs... running scons -j8 with a
        bunch of tests and a batch queing system means that 8 cpus are
        consumed, even if there is only one job running.
      - Either make scons be able to submit the jobs or have something
        else that manages the jobs and their completion status
          - pytest should be able to do this, again, open question it if
            does it via multi-threads and bsub -K on each job or if we
            should have pytest directly connect to running machines.
  - Easy support for running separate tests where only the input
    parameters differ
      - As long as we do a good job creating the base class, this should
        be trivial --[Saidi](User:Saidi "wikilink") 18:09, 22 August
        2011 (PDT)
      - For example, several protocols utilize different state
        transitions depending on configuration flags. It would be great
        if we could test these without having to create new directories
        and tests.
          - Big question is how do we store all of the file data,
            especially when the stats are python objects (hint hint
            nate). Perhaps putting everything in a SQLite DB might not
            be a bad approach so we don't need complex directory
            structures --[Saidi](User:Saidi "wikilink") 18:09, 22 August
            2011 (PDT)
      - Similarly, we could/should test topologies this way as well.
  - Automated way to use nightly regressions as a basis for updating
    "m5-stable"
      - I would vote to defer this until we have a continuous
        integration solution --[Saidi](User:Saidi "wikilink") 18:09, 22
        August 2011 (PDT)
      - How do you identify the last working revision? (from Ali)
      - Maybe need a bug-tracking system so we could record facts like
        "changeset Y fixes a bug introduced in changeset X" then we
        could automatically exclude changesets between X and Y, but we
        don't have that. (from stever)
  - Better definitions of success criteria.
      - E.g. Stats were changed, but output is all still correct vs
        simply passed and failed. (Passed, stats diffs, failed)
      - For example you could say that the terminal output changing is
        fail, or the stdout and spec binary outputs changing are failed,
        but a 1% difference in stats is a stats difference, which needs
        to be addresses
      - I envision this as providing reasonable certainty that if you
        create a change you know will modify the stats, you have a quick
        verification that nothing broke horribly before updating the
        stats.
          - I think all of these can be wrapped up into what pass/fail
            means and what is printed. Depending on the test we could
            say that stats don't matter but simout must be the same,
            etc. --[Saidi](User:Saidi "wikilink") 18:09,

22 August 2011 (PDT)

  -   - It would be nice to also track macroop/regular instruction and
        microop counts separately. In SE mode at least, if the number of
        ISA level instructions changes it's more important than if the
        number of microops change. That way when you change microcode,
        you can tell if the same instructions are executing more or less
        and just their implementation has changed.
        --[Gblack](User:Gblack "wikilink") 21:35, 22 August 2011 (PDT)
      - For directed tests like the ones that test specific
        instructions, it might be worthwhile to add pseudo ops that
        signal success or completion through some sort of backdoor.
        There could be similar instructions which pause the simulation
        and give the simulation script a chance to look at state (I
        don't think we can now, but it would be handy) and decide if,
        say, the add put the right thing in the right registers, or
        memory was updated correctly. One major drawback to this sort of
        approach is that we couldn't easily run these on real machines.
        On the positive side, we could avoid problems where
        complementary bugs make a test pass when it shouldn't. For
        instance, maybe adds and subtracts have the same bug, so both
        the output and checking the output against reference values
        might be wrong.--[Gblack](User:Gblack "wikilink") 21:35, 22
        August 2011 (PDT)

## Implementation ideas

Just ideas... no definitive decisions have been made yet.

  - Use Python's [unittest
    module](http://docs.python.org/library/unittest.html), or something
    that extends it such as
    [nose](http://somethingaboutorange.com/mrl/projects/nose/1.0.0)
  - Use SCons to manage dependencies between binaries/test inputs and
    test results, but in a different SCons invocation (i.e., in its own
    SConstruct/SConscript)

## Tests wish list

  - Checkpointing creates valid checkpoint
  - Restore of checkpoint (maybe run something like the auto-checkpoint
    tester)
  - CPU switching
  - SMARTS sampling
  - tests in between spec and hello world. Maybe the hpc games
    benchmarks
    <http://www.google.com/codesearch#iw4AHL8Ip0w/~kohl/HPC.GAMES/hpcg1.0.tar.gz>
    would be a good choice and could be distributed next to gem5.
  - Getting rid of tests that run for more that 1 hour
  - Using a memory tester to create interesting coherence actions,
    specifically around prefetching, "bonus" block allocation, etc.
