---
title: "Nate's wishlest"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

### Framework

  - Parallelize M5
      - Converting everything to support the new Params mechanism is a
        prerequisite
      - Make various global structures thread safe (e.g. Instruction
        cache, Statistics, etc.)
  - Unify full system and syscall emulation modes into a single build
  - Unify all ISAs into a single build
      - This requires fixes to the way endianness is handled in the
        memory system.
      - Add a NOISA option that doesn't have CPUs till this happens.
  - Modularize the build, allowing people to pick and choose which
    SimObjects get compiled in.
  - Make Python/C++ I/O systems work much better together. i.e. C++ can
    cprintf to python file objects, or Python can print to C++
    iostreams.
  - Handle reading and writing of gzipped files
  - Make switching between CPU models way easier
  - Get simulations with \> 100 cores working

### CPU Models

  - Support something resembling direct execution using KVM.
  - Have a very fast model that dynamically compiles like PIN (or even
    use PIN).
  - Make a configurable in-order model
  - Make a very fast functional CPU model that doesn't have any stats or
    anything like that and is as fast as possible.
  - Clean up wakeup/sleeping/interrupts so the CPU models are more
    consistent and the interface isn't so horrendous.

### Monitoring

  - Listen on a socket with cherrypy and allow people to browse and
    query the status of a simulation.

### Configuration

  - Clean up SimObject.py significantly
  - Clean up configs/\* and make it much more modular and library like
  - Make a top level DictParam that is like a VectorParam

### Memory System

  - Clean up interrupt handling and route interrupts through the memory
    system as packets.
  - Implement something like SLICC for M5.
  - Implement the ARM instruction.
  - Add support to flush caches.

### Disk Models

  - Support something like qemu's qcow2 disk image
      - This is actually bsd code, so we can steal it
      - Figure out how to mount it natively on linux so we don't have to
        convert back and forth to add stuff
  - Add host Linux file system support so the guest can easily access
    files on the host.
      - Maybe support the host side of vmhgfs since the guest side
        already exists and is GPL.

### Disk Images

  - Just clean this stuff up
  - Figure out a replacement for ptxdist.
      - Currently think Gentoo is probably the best option.

### Statistics

  - Make all stats types actually work.
  - Move lots of stats processing to python. Keep collection in C++, but
    all of the printing stuff should go to python.
  - Add a sqlite backend.
  - Clean up and better integrate the graphing stuff.

### Ethernet

  - Add Ethernet overheads (preamble, trailer, inter frame gap) to
    EtherLink model.
  - Build and Ethernet switch model.
  - Get EtherTap working.
      - Keep the stuff that must run as root separate.
