---
title: "O3"
date: 2018-05-13T18:33:29-04:00
draft: false
---


The O3CPU is our new detailed model for the v2.0 release. It is an out
of order CPU model loosely based on the Alpha 21264. This page will give
you a general overview of the O3CPU model, the pipeline stages and the
pipeline resources. We have made efforts to keep the code well
documented, so please browse the code for exact details on how each part
of the O3CPU works.

## Pipeline stages

The O3CPU has the following pipeline stages:

  - Fetch
      - Fetches instructions each cycle, selecting which thread to fetch
        from based on the policy selected. This stage is where the
        DynInst is first created. Also handles branch prediction.
  - Decode
      - Decodes instructions each cycle. Also handles early resolution
        of PC-relative unconditional branches.
  - Rename
      - Renames instructions using a physical register file with a free
        list. Will stall if there are not enough registers to rename to,
        or if back-end resources have filled up. Also handles any
        serializing instructions at this point by stalling them in
        rename until the back-end drains.
  - Issue/Execute/Writeback
      - Our simulator model handles both execute and writeback when the
        execute() function is called on an instruction, so we have
        combined these three stages into one stage. This stage (IEW)
        handles dispatching instructions to the instruction queue,
        telling the instruction queue to issue instruction, and
        executing and writing back instructions.
  - Commit
      - Commits instructions each cycle, handling any faults that the
        instructions may have caused. Also handles redirecting the
        front-end in the case of a branch misprediction.

## Pipeline resources

Additionally it has the following structures:

  - Branch predictor
      - Allows for selection between several branch predictors,
        including a local predictor, a global predictor, and a
        tournament predictor. Also has a branch target buffer and a
        return address stack.
  - Reorder buffer
      - Holds instructions that have reached the back-end. Handles
        squashing instructions and keeping instructions in program
        order.
  - Instruction queue
      - Handles dependencies between instructions and scheduling ready
        instructions. Uses the memory dependence predictor to tell when
        memory operations are ready.
  - Load-store queue
      - Holds loads and stores that have reached the back-end. It hooks
        up to the d-cache and initiates accesses to the memory system
        once memory operations have been issued and executed. Also
        handles forwarding from stores to loads, replaying memory
        operations if the memory system is blocked, and detecting memory
        ordering violations.
  - Functional units
      - Provides timing for instruction execution. Used to determine the
        latency of an instruction executing, as well as what
        instructions can issue each cycle.
  - Memory dependence prediction using [store
    sets](http://citeseer.ist.psu.edu/chrysos98memory.html)
      - Informs the IQ which memory instructions it predicts as ready to
        issue (in terms of memory ordering). In the Alpha models, memory
        operations have been atomic operations where the address
        calculation and memory access are bundled as one instruction.
        Because the effective addresses are not calculated separately,
        memory dependence prediction is necessary in order to give some
        idea of the order in which memory operations can execute.

## Execute-in-execute model

For the O3CPU, we've made efforts to make it highly timing accurate. In
order to do this, we use a model that actually executes instructions at
the execute stage of the pipeline. Most simulator models will execute
instructions either at the beginning or end of the pipeline;
SimpleScalar and our old detailed CPU model both execute instructions at
the beginning of the pipeline and then pass it to a timing backend. This
presents two potential problems: first, there is the potential for error
in the timing backend that would not show up in program results. Second,
by executing at the beginning of the pipeline, the instructions are all
executed in order and out-of-order load interaction is lost. Our model
is able to avoid these deficiencies and provide an accurate timing
model.

## Template Policies

The O3CPU makes heavy use of template policies to obtain a level of
polymorphism without having to use virtual functions. It uses template
policies to pass in an "Impl" to almost all of the classes used within
the O3CPU. This Impl has defined within it all of the important classes
for the pipeline, such as the specific Fetch class, Decode class,
specific DynInst types, the CPU class, etc. It allows any class that
uses it as a template parameter to be able to obtain full type
information of any of the classes defined within the Impl. By obtaining
full type information, there is no need for the traditional virtual
functions/base classes which are normally used to provide polymorphism.
The main drawback is that the CPU must be entirely defined at compile
time, and that the templated classes require manual instantiation. See
src/cpu/o3/impl.hh and src/cpu/o3/cpu_policy.hh for example Impl
classes.

## ISA independence

The O3CPU has been designed to try to separate code that is ISA
dependent and code that is ISA independent. The pipeline stages and
resources are all mainly ISA independent, as well as the lower level CPU
code. The ISA dependent code implements ISA-specific functions. For
example, the AlphaO3CPU implements Alpha-specific functions, such as
hardware return from error interrupt (hwrei()) or reading the interrupt
flags. The lower level CPU, the FullO3CPU, handles orchestrating all of
the pipeline stages and handling other ISA-independent actions. We hope
this separation makes it easier to implement future ISAs, as hopefully
only the high level classes will have to be redefined.

## Interaction with ThreadContext

The [ThreadContext](ThreadContext "wikilink") provides interface for
external objects to access thread state within the CPU. However, this is
slightly complicated by the fact that the O3CPU is an out-of-order CPU.
While it is well defined what the architectural state is at any given
cycle, it is not well defined what happens if that architectural state
is changed. Thus it is feasible to do reads to the ThreadContext without
much effort, but doing writes to the ThreadContext and altering register
state requires the CPU to flush the entire pipeline. This is because
there may be in flight instructions that depend on the register that has
been changed, and it is unclear if they should or should not view the
register update. Thus accesses to the ThreadContext have the potential
to cause slowdown in the CPU simulation.

## Anatomy of the pipeline

## Fetch Handling

![Fetch.png](Fetch.png "Fetch.png")

## DynInsts

Pipeline, PC/predicted target, dynamic information

## Wires/delay

## Squashing

## Load/store handling

![LDSTR.png](LDSTR.png "LDSTR.png")

## Renaming

## etc.
