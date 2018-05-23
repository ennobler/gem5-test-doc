---
title: "Todo list"
date: 2018-05-13T17:08:16-04:00
draft: false
weight: 70
---

{{% notice warning %}}
This todo list is likely out-of-date
{{% /notice %}}

### Highest priority

  - [ ] Flesh out and debug 64-bit modern ISA (what's needed by users)
  - [ ] FS-mode core timing issues
      - [ ] Debug TimingSimpleCPU issues?
      - [ ] In-order pipeline core model support
      - [ ] Out-of-order core model (O3) support
  - [ ] Multiprocessor timing support: need to enforce atomicity of locked
    load/op/store sequences in timing cache models
      - [ ] Ruby and M5 classic?
  - [ ] Performance correlation
      - [ ] With real hardware and/or existing correlated simulator
      - [ ] Micro-op counts for functional implementation
      - [ ] Timing for out-of-order core (requires O3 support)
  - [ ] Complete x87 support
  - [ ] AVX support

### Useful but not strictly necessary

  - [ ] Split up ISA output for faster compiling
  - [ ] Improve ISA description language support for x86

### To do eventually but not right away

  - [ ] ACPI support
  - [ ] KVM-based fast functional CPU model
  - [ ] Virtualization extensions support (AMD SVM, etc.)

### Could be done but might never happen

  - [ ] other OS support (OSX, Windows, ??)
  - [ ] complete real mode support
