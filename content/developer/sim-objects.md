---
title: "SimObjects"
date: 2018-05-13T18:51:37-04:00
draft: false
---

## SimObjects

### The python side

#### [Param types](Python_Parameter_Types "wikilink")

#### Inheritance

#### Special attribute names

#### Rules for importing - how to get what

#### Pro tips - avoiding cycles, always descend from root, etc.

### The C++ side

#### create functions

#### Stages of initialization

The basic order of C++ SimObject construction and initialization is
controlled by the `instantiate()` and `simulate()` Python functions in
`src/python/m5/simulate.py`. This process was revised in
[changeset 3f6413fc37a2](http://repo.m5sim.org/gem5?cmd=changeset;node=3f6413fc37a2).
This page documents the process as of that changeset.

Once the Python SimObject hierarchy is created by the user's simulation
script, that script calls `instantiate()` to create the C++ equivalent
of that hierarchy. The primary steps in this process are:

1.  Resolve proxy parameters (those specified using `Self` or `Parent`).
2.  Dump the `config.ini` file to record the final resolved set of
    SimObjects and parameter values.
3.  Call the C++ constructors for all of the SimObjects. These
    constructors are called in an order that satisfies parameter
    dependencies, i.e., if object A is passed as a parameter to object
    B, then object A's constructor will be called before object B's
    constructor so that the C++ pointer to A can be passed to B's
    constructor. Note that this policy means that cyclic parameter
    dependencies cannot be supported.
4.  Instantiate the memory-system port connections. In the case of
    restoring from a checkpoint, not all ports are connected at this
    stage, i.e., the switch CPU(s) are not connected.
5.  Call `init()` on each SimObject. This provides the first opportunity
    to perform initializations that depend on *all* SimObjects being
    fully constructed.
6.  Call `regStats()` on each SimObject.
7.  Complete initialization of SimObject state from a checkpoint or from
    scratch.
      - If restoring from a checkpoint, call `loadState(ckpt)` on each
        SimObject. The default implementation simply calls
        `unserialize()` if there is a corresponding checkpoint section,
        so we get backward compatibility with earlier versions of the
        startup code for existing objects. However, objects can override
        `loadState()` to get other behaviors, e.g., doing other
        programmed initializations after `unserialize()`, or complaining
        if no checkpoint section is found.
      - If not restoring from a checkpoint, call `initState()` on each
        SimObject instead. This provides a hook for state
        initializations that are only required when *not* restoring from
        a checkpoint.
8.  If restoring from a checkpoint, the switch CPU ports are connected
    in `BaseCPU::takeOverFrom` at this stage.
9.  Later, the first time that the user script calls `simulate()`, call
    `startup()` on each SimObject. This is the point where SimObjects
    that do self-initiated processing should schedule internal events,
    since the value of `curTick` could change during unserialization.

### Initialization Function Call Sequence

The figure below shows the function call sequence for the initialization
of gem5 when running the "Hello, World" example given in the
[Introduction](Introduction#Running "wikilink") section. Each node in
the graph gives the path to the source code file in which the function
is located. This representation should aid in the first steps of
familiarization with the basic initialization steps of gem5, including
how the configuration files are used and how the objects are
constructed.

![gem5_initialization_call_sequence.png](gem5_initialization_call_sequence.png
"gem5_initialization_call_sequence.png")

#### Header files to include
