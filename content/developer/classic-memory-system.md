---
title: "Classic Memory System"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

## MemObjects

## Caches

The default cache is a non-blocking cache with MSHR (miss status holding
register) and WB (Write Buffer) for read and write misses. The Cache can
also be enabled with prefetch (typically in the last level of cache).
The default [replacement policy](replacement_policy "wikilink") for the
cache lines is LRU (least recently used).

### Hooking them up

test

### Parameters

## Interconnects

### Crossbars

The two types of traffic in the crossbar are memory-mapped packets and
snooping packets. The memory-mapped requests go down the memory
hierarchy, and responses go up the memory hierarchy (same route back).
The snooping requests go horizontally and up the cache hierarchy,
snooping responses go horizontally and down the hierarchy (same route
back). Normal snoops go horizontally and express snoops go up the cache
hierarchy.

![Bus.png](Bus.png "Bus.png")

### Bridges

### Anything else?

## Coherence

M5 2.0b4 introduced a substantially rewritten and streamlined cache
model, including a new coherence protocol. (The old pre-2.0 cache model
had been patched up to work with the new [Memory
System](Memory_System "wikilink") introduced in 2.0beta, but not
rewritten to take advantage of the new memory system's features.)

The key feature of the new coherence protocol is that it is designed to
work with more-or-less arbitrary cache hierarchies (multiple caches each
on multiple levels). In contrast, the old protocol restricted sharing to
a single bus.

In the real world, a system architecture will have limits on the number
or configuration of caches that the protocol can be designed to
accommodate. It's not practical to design a protocol that's fully
realistic and yet efficient for arbitrary configurations. In order to
enable our protocol to work on (nearly) arbitrary configurations, we
currently sacrifice a little bit of realism and a little bit of
configurability. Our intent is that this protocol is adequate for
researchers studying aspects of system behavior other than coherence
mechanisms. Researchers studying coherence specifically will probably
want to replace the default coherence mechanism with implementations of
the specific protocols under investigation.

The protocol is a MOESI snooping protocol. Inclusion is **not**
enforced; in a CMP configuration where you have several L1s whose total
capacity is a significant fraction of the capacity of the common L2 they
share, inclusion can be very inefficient.

Requests from upper-level caches (those closer to the CPUs) propagate
toward memory in the expected fashion: an L1 miss is broadcast on the
local L1/L2 bus, where it is snooped by the other L1s on that bus and
(if none respond) serviced by the L2. If the request misses in the L2,
then after some delay (currently set equal to the L2 hit latency), the
L2 will issue the request on its memory-side bus, where it will possibly
be snooped by other L2s and then be issued to an L3 or memory.

Unfortunately, propagating snoop requests incrementally back up the
hierarchy in a similar fashion is a source of myriad nearly intractable
race conditions. Real systems don't typically do this anyway; in general
you want a single snoop operation at the L2 bus to tell you the state of
the block in the whole L1/L2 hierarchy. There are a handful of methods
for this:

1.  just snoop the L2, but enforce inclusion so that the L2 has all the
    info you need about the L1s as well---an idea we've already rejected
    above
2.  keep an extra set of tags for all the L1s at the L2 so those can be
    snooped at the same time (see the Compaq Piranha)---reasonable, if
    you're hierarchy's not too deep, but now you've got to size the tags
    in the lower-level caches based on the number, size, and
    configuration of the upper-level caches, which is a configuration
    pain
3.  snoop the L1s in parallel with the L2, something that's not hard if
    they're all on the same die (I believe Intel started doing this with
    the Pentium Pro; not sure if they still do with the Core2 chips or
    not, or if AMD does this as well, but I suspect so)---also
    reasonable, but adding explicit paths for these snoops would also
    make for a very cumbersome configuration process

We solve this dilemma by introducing "express snoops", which are special
snoop requests that get propagated up the hierarchy instantaneously and
atomically (much like the atomic-mode accesses described on the [Memory
System](Memory_System "wikilink") page), even when the system is running
in timing mode. Functionally this behaves very much like options 2 or 3
above, but because the snoops propagate along the regular bus
interconnects, there's no additional configuration overhead. There is
some timing inaccuracy introduced, but if we assume that there are
dedicated paths in the real hardware for these snoops (or for
maintaining the additional copies of the upper-level tags at the
lower-level caches) then the differences are probably minor.

(More to come: how does a cache know when its request is completed? and
other fascinating questions...)

Note: there are still some bugs in this protocol as of 2.0b4,
particularly if you have multiple L2s each with multiple L1s behind it,
but I believe it works for any configuration that worked in 2.0b3.

## Debugging

There is a feature in the classic memory system for displaying the
coherence state of a particular block from within the debugger (e.g.,
gdb). This feature is built on the classic memory system's support for
functional accesses. (Note that this feature is currently rarely used
and may have bugs.)

If you inject a functional request with the command set to PrintReq, the
packet traverses the memory system (like a regular functional request)
but on any object that matches (other queued packet, cache block, etc.)
it simply prints out some information about that object.

There's a helper method on Port called printAddr() that takes an address
and builds an appropriate PrintReq packet and injects it. Since it
propagates using the same mechanism as a normal functional request, it
needs to be injected from a port where it will propagate through the
whole memory system, such as at a CPU. There are helper printAddr()
methods on MemTest, AtomicSimpleCPU, and TimingSimpleCPU objects that
simply call printAddr() on their respective cache ports. (Caveat: the
latter two are untested.)

Putting it all together, you can do this:

    (gdb) set print object
    (gdb) call SimObject::find(" system.physmem.cache0.cache0.cpu")
    $4 = (MemTest *) 0xf1ac60
    (gdb) p (MemTest*)$4
    $5 = (MemTest *) 0xf1ac60
    (gdb) call $5->printAddr(0x107f40)

    system.physmem.cache0.cache0
      MSHRs
        [107f40:107f7f] Fill   state:
          Targets:
            cpu: [107f40:107f40] ReadReq
    system.physmem.cache1.cache1
      blk VEM
    system.physmem
      0xd0

... which says that cache0.cache0 has an MSHR allocated for that address
to serve a target ReadReq from the CPU, but it's not in service yet
(else it would be marked as such); the block is valid, exclusive, and
modified in cache1.cache1, and the byte has a value of 0xd0 in physical
memory.

Obviously it's not necessarily all the info you'd want, but it's pretty
useful. Feel free to extend. There's also a verbosity parameter that's
currently not used that could be exploited to have different levels of
output.

Note that the extra "p (MemTest\*)$4" is needed since although "set
print object" displays the derived type, internally gdb still considers
the pointer to be of the base type, so if you try and call printAddr
directly on the $4 pointer you get this:

    (gdb) call $4->printAddr(0x400000)
    Couldn't find method SimObject::printAddr

