---
title: "ThreadState"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

The ThreadState class is used to hold thread state that is common across
CPU models, such as the thread ID, thread status, kernel statistics,
memory port pointers, and some statistics of number of instructions
completed. Each CPU model can derive from ThreadState and build upon it,
adding in thread state that is deemed appropriate. An example of this is
[SimpleThread](/internal/simple-thread/), where all of the thread's
architectural state has been added in. However, it is not necessary (or
even feasible in some cases) for all of the thread's state to be
centrally located in a ThreadState derived class. The DetailedCPU keeps
register values and rename maps in its own classes outside of
ThreadState. ThreadState is only used to provide a more convenient way
to centrally locate some state, and provide sharing across CPU models.

