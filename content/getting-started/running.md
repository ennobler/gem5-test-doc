---
title: "Running"
date: 2018-05-12T22:42:18-04:00
draft: false
weight: 35
---


### Modes
\= System-call Emulation (SE) mode is one of the two modes of gem5 and
the most basic. gem5 simulates your program and traps any system calls
made to the host then emulates them. In the older versions of gem5, only
statically linked binaries were allowed to be simulated in SE mode
however, gem5 2.0 now supports dynamically linked binaries.
{{% notice info %}}
Not all architecutres support dynamically linked binaries
{{% /notice %}}


