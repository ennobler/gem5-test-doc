---
title: "SimpleThread"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

The SimpleThread class derives from the
[ThreadState](ThreadState "wikilink") class, and is used to provide all
architectural state for models that don't need anything more complex. It
is basically the context of a hardware thread, and provides all
necessary functions needed to access it as defined by the
[ThreadContext](ThreadContext "wikilink") class. However, SimpleThread
does not actually derive from ThreadContext, but rather uses a proxy
class to forward all ThreadContext function calls to its own functions.
This allows the CPU models to use SimpleThread without paying for
virtual function overhead.

The SimpleThread class has:

  - An architected register file, including the PC, next PC, integer,
    FP, and miscellaneous registers
  - A pointer to the thread's CPU
  - A pointer to the ITB, the DTB
  - A pointer to the system
  - A pointer to the thread's ThreadContext proxy
  - All state from ThreadState, such as kernel statistics and memory
    ports

