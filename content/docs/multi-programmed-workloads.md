---
title: "Multi-programmed workloads"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

## Running

In SE mode, simply create a system with multiple CPUs and assign a
different workload object to each CPU's workload parameter. If you're
using the O3 model, you can also assign a vector of workload objects to
one CPU, in which case the CPU will run all of the workloads
concurrently in SMT mode. Note that SE mode has no thread scheduling; if
you need a scheduler, run in FS mode and use the fine scheduler built
into the Linux kernel.

## Terminating

There are several options when deciding how to stop your workloads:

1.  Terminate as soon as any thread reaches a particular maximum number
    of instructions. This is equivalent to max_insts_any_thread. The
    problem here is that multithreaded programs are non-determinstic,
    and there is no way to determine how many instructions the other
    threads will have executed. The amount of work done per instruction
    could change as well if more or less time is spent waiting for other
    threads. The benefit of this approach is that all threads are
    running fully until the simulation terminates.
2.  Terminate once all threads have reached a maximum number of
    instructions. This is equivalent to max_insts_all_threads. With
    this option you can be sure all threads do at least a certain amount
    of work, but threads that reach the maximum continue executing and
    there's no way to know how much extra work these other threads will
    do.
3.  Terminate each thread individually after it executes a particular
    number of instructions. This option is not currently supported. All
    threads will do the same amount of work which avoids some of the
    problems mentioned above, but now the threads may not all be running
    for the entire simulation. After some threads have finished, the
    remaining threads will have less competition for resources, and the
    compute resources will have less work to do.
4.  Terminate each thread individually after it reached a particular,
    per thread number of instructions. This is basically the same as the
    previous option but allows tuning the number of instructions
    executed per thread. It is also not currently supported.

## Merging Single Workload Checkpoints
