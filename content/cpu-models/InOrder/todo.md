---
title: "TODO"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

## Python Configurability

  - Resource Configuration - How can we specify what resources are
    instantiated via the Python config files?
      - ResourceType - Type of resource (Enum type)
          - ResourceParams - Parameters for this type of resource
          - Request - List of requests for this type of resource (Enum
            type)
              - Latency - operation latency and issue latency
                (intra/inter thread)
          - Count - Number of such resource type

<!-- end list -->

  - Pipeline Description
      - InstSchedule - Instruction schedule specified as a vector of
        InstClassSchedule
          - InstClassSchedule - Vector of schedules per instruction
            class - load/store, Int execute, FP execute, specialized
            inst, etc. (do we still want a distinction between front end
            and back end schedules?)
      - ResourceRequestList - Vector of ResourceRequest (per stage?)
          - ResourceRequest - Vector of requests for resources
      - ResourceType/Request options

## Resources

  - Execution Unit
      - Fold Address Generation Unit (AGEN) into the EXE
      - Pass a Function Unit Pool object to the EXE and specify what
        functions can operate through a specific unit (similar to O3)

## Simulation Speed

  - Instruction Sleeping
      - Sleep instructions waiting for an long-delay event (instead of
        constantly ask a resource if it's complete

<!-- end list -->

  - Event Sleeping
      - Sleep CPU w/no activity - Implemented on a coarse-grain level,
        but Activity object can be tuned to be exact.

<!-- end list -->

  - Resource Pool
  - Instead of broadcasting all events to the resource pool, have
    resources declare what functions they have virtual functions for (or
    auto-detect this) and then only broadcast to the right set of
    resources every time (e.g. the BranchPredictor may not care about a
    Trap Event)

## ISA Support

  - ALPHA - **completed**
  - MIPS - **completed**
  - SPARC - *partially completed* (not currently being developed)
  - ARM - not completed - *Support for Micro-Ops Needed (Template code
    from Simple or O3 CPU?)*
  - X86 - not completed - *Support for Micro-Ops Needed (Template code
    from Simple or O3 CPU?)*
  - POWER - not completed

## Full System Support

  - InOrder can boot Linux, but testing for benchmark suites
      - PARSEC
      - SPLASH2
      - SPEC2K6

## Checkpointing

  - The serialize/unserialize functions are currently unimplemented in
    InOrder

## SwitchOut

  - Implement the drain function for InOrder
