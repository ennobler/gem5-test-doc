---
title: "ThreadContext"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---
ThreadContext is the interface to all state of a thread for anything
outside of the CPU. It provides methods to read or write state that
might be needed by external objects, such as the PC, next PC, integer
and FP registers, and IPRs. It also provides functions to get pointers
to important thread-related classes, such as the ITB, DTB, System,
kernel statistics, and memory ports. It is an abstract base class; the
CPU must create its own ThreadContext by either deriving from it, or
using the templated ProxyThreadContext class.

### ProxyThreadContext

The ProxyThreadContext class provides a way to implement a ThreadContext
without having to derive from it. ThreadContext is an abstract class, so
anything that derives from it and uses its interface will pay the
overhead of virtual function calls. This class is created to enable a
user-defined Thread object to be used wherever ThreadContexts are used,
without paying the overhead of virtual function calls when it is used by
itself. The user-defined object must simply provide all the same
functions as the normal ThreadContext, and the ProxyThreadContext will
forward all calls to the user-defined object. See the code of
[SimpleThread](/developer/simple-thread/) for an example of using the
ProxyThreadContext.

### Difference vs. ExecContext

The ThreadContext is slightly different than the
[ExecContext](/developer/execution-basics/#ExecContext). The
ThreadContext provides access to an individual thread's state; an
ExecContext provides ISA access to the CPU (meaning it is implicitly
multithreaded on SMT systems). Additionally the ThreadState is an
abstract class that exactly defines the interface; the ExecContext is a
more implicit interface that must be implemented so that the ISA can
access whatever state it needs. The function calls to access state are
slightly different between the two. The ThreadContext provides
read/write register methods that take in an architectural register
index. The ExecContext provides read/write register methdos that take in
a [StaticInst](/developer/static-inst/) and an index, where the index
refers to the i'th source or destination register of that StaticInst.
Additionally the ExecContext provides read and write methods to access
memory, while the ThreadContext does not provide any methods to access
memory.
