---
title: "GPU Models"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

## AMD's Compute-GPU Model

### MICRO-48 Tutoral

A tutorial was held in conjunction with MICRO-48. We have made the
slides available from our 2015 tutorial titled: [The AMD gem5 APU
Simulator: Modeling Heterogeneous Systems in
gem5](Media:AMD_gem5_APU_simulator_micro_2015_final.pptx "wikilink").

### Compute GPU Workloads

### Emualted CL Runtime

  - Download the [emulated OpenCL
    runtime](http://www.gem5.org/dist/current/gpu/cl-runtime.tar.xz).

### OpenCL Compiler

[CLOC](https://github.com/HSAFoundation/CLOC) is used to compile OpenCL
kernels for use with gem5's GPU compute model. The most recent revision
of CLOC that is known to work with gem5 is:

commit cf777856cfce86d11ea97c245992971159b85a4d

### Rondinia Benchmark Suite

## ARM's NoMali GPU Model

The NoMali GPU model models the interface used by ARM Mali GPUs. The
model does not render or compute anything, but can be used to fake a
GPU. This enables Android and ChromeOS experiments without software
rendering which would otherwise make simulation results extremely
misleading. It was
[presented](media:2015_ws_04_ISCA_2015_NoMali.pdf "wikilink") in the
[2015 gem5 User Workshop](User_workshop_2015 "wikilink").

Getting started instructions are currently available for [Android 4.4
(KitKat)](Android_KitKat "wikilink").

