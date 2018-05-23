---
title: "Derivative projects"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---
Below is a list of projects that are based on gem5, are extensions of
gem5, or use gem5.

# MV5

  - MV5 is a reconfigurable simulator for heterogeneous multicore
    architectures. It is based on M5v2.0 beta 4.
  - Typical usage: simulating data-parallel applications on SIMT cores
    that operate over directory-based cache hierarchies. You can also
    add out-of-order cores to have a heterogeneous system, and all
    different types of cores can operate under the same address space
    through the same cache hierarchy.
  - Research projects based on MV5 have been published in ISCA'10,
    ICCD'09, and IPDPS'10.

### Features

  - Single-Instruction, Multiple-Threads (SIMT) cores
  - Directory-based Coherence Cache: MESI/MSI. (Not based on gems/ruby)
  - Interconnect: Fully connected and 2D Mesh. (Not based on gems/ruby)
  - Threading API/library in system emulation mode (No support for
    full-system simulation. A benchmark suite using the thread API is
    provided)

### Resources

  - Home Page: [1](https://sites.google.com/site/mv5sim/home)
  - Tutorial at ISPASS '11:
    [2](https://sites.google.com/site/mv5sim/tutorial)
  - Google group: [3](http://groups.google.com/group/mv5sim)

# gem5-gpu

  - Merges 2 popular simulators: gem5 and gpgpu-sim
  - Simulates CPUs, GPUs, and the interactions between them
  - Models a flexible memory system with support for heterogeneous
    processors and coherence
  - Supports full-system simulation through GPU driver emulation

### Resources

  - Home Page: [4](https://gem5-gpu.cs.wisc.edu)
  - Overview slides:
    [5](http://gem5.org/wiki/images/7/7d/2012_12_gem5_gpu.pdf)
  - Mailing list: [6](http://groups.google.com/group/gem5-gpu-dev)
