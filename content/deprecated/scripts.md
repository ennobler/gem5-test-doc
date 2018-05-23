---
title: "Config scripts ideas"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

### Background

The configurations we have work, but are really hard to learn and the
code is getting harder and harder to effectively maintain

### Issues

A extensible configuration system is one of the road blocks to a better
regression system Asymmetric configurations are problematic with the
current system as we need to come up with a way to set CPUID information
for each cluster and size them independently Having clocks to emulate
some sort of DVFS (just frequency scaling) is hard. Ultimately you'd
like to be able to change the clock and associated objects (caches etc)
would all slow down or speed up.

### Possible Solutions

There are a couple of things that I propose here, but generally it
involves creating a CPU container object (or maybe more generally a
container object). The container has a set of ports so that functions
can always know how to connect to them, but it's not a specific object.
In this way we could wrap a core + l1s in a container and hand it to
another function to build a cluster out of and similar we could hand
multiple containers of objects to a system to build a multi-cluster
system. The containers could have clocks and in that way we could use
the'Nc' notation for latencies in gem5 where instead of specifying
'10ns' you can specify '5c' (5 of the parent clocks), so that should
deal with the first level of configuration issues. Th

The second part of the process is to make the various building function
we have much more generic in what they do and move them to be part of
gem5. This is somewhat like what addL1caches() or connectAllPorts() does
on CPU models, but we should have more them and they should take objects
to instantiate, not instantiate fixed object. Another part of it is
better use of Base classes. If we name all L1 caches L1Cache (maybe
BL1Cache/LL1Cache for an asymmetric system), then a script to change all
the big or the little cache sizes is as stipple as BL1Cache = '32kB', no
crazy tree of if statements is required.

Finally, it might let use use decorators to build a system, for example
if we start with a working configuration we could then have decorators
(http://stackoverflow.com/questions/739654/understanding-python-decorators)
permute it:

`@detailedCPU(A15Like)`
`@L1caches('64kb')`
`@L2caches('2MB')`
`@makeMultiCluster(2)`
`makeArmSystem(cores=4)`

With the underlying containers the decorators should be able to pull
apart and recreate the system each time adding the functionality.

### Rethinking configuration from the ground up

1.  fix the paths stuff. Right now we've hard coded some paths that use
    and use a env variable to find the rest. We then hard code specific
    directories after those paths (e.g. binaries, disks, etc). This
    seems rather pointless and only allows a single monothlic location
    to store files in.We should have configuration file like
    ~/.gem5_paths that contains paths to search. The functions that
    find files should just search everywhere, and if we're worried about
    name collisions we can record the md5 sum of files in (3) and warn
    if a file with the right md5 sum can't be found or error if there
    are two or more possible files neither of which has a matching md5
    sum.
2.  encapsulate the right things in SysConfig()
3.  re-do the notion of benchmarks where a benchmark is a directory with
    a configuration file in it; search for necessary files in this
    directory or the search paths; don't need to edit a file to add the
    benchmarks just go looking for directories with benchmark.cfg or
    whatever in it
4.  Create generic classes within gem5 for as many objects as possible
    (e.g. L1ICache, etc)
5.  Create helper methods that live in gem5 such as connectAllPorts() to
    connect objects
6.  When objects are being instantiated, pass those objects as kv
    parameters (with defaults) rather then hard coding their names so
    that someone can utilize the function. For example rather than
    having a createCaches(options, system) we really should have
    createCaches(l1i=L1ICache, l1d=L1DCache, L2=L2Cache, L3=None) and
    the user can override these and still get a working system out as
    opposed to having a tree of ifs or hard coding specific names within
    the function. The creation of the objects and the configuration of
    them should be separate. Later in the config you can say
    L1Cache.size = '128kB" and not have to worry.
7.  Remove as much duplication as possible with the above in FSConfig.py
8.  Encapsulate units of the configuration in containers described above
9.  Provide a mechanism to supply "default" configs that can execute a
    request of L1ICache.size = ... commands and make the topology look
    like a particular configuration
10. Re-write simulation.py to be comprehendible

