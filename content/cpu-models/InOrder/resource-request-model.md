---
title: "Resource request model"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

## Overview

Resources consists of any CPU object that an instruction wants to
access. This could be a branch predictor, a cache, a execution unit,
etc. In the InOrder CPU model we abstract what a resource is into a
generic "Resource" class that all specific resources must derive from.
In any given pipeline stage, an instruction will request that a resource
perform a specific operation on it's behalf. If an instruction can
complete all it's resource requests for a given stage, then it may pass
to the next stage.

**Relevant source files:**

  - resource.\[hh,cc\]
  - resources/\*.\[hh,cc\]
  - pipeline_traits.\[hh,cc\]
  - cpu.\[hh,cc\]

## Resource-Request Model

Ideally, resources can used by instructions as a black box that will
perform specific operation.The basic steps between an instruction and
resource are as follows:

  - An instruction accesses a resource by generating a
    **resource-request** that contains:

<!-- end list -->

1.  The ID of the resource to be accessed
2.  A command for the resource to perform

<!-- end list -->

  - After generating a resource-request, an instruction sends the
    request to the Resource Pool which will forward the request to the
    appropriate resource.

<!-- end list -->

  - Once a resource receives a request, it will:

<!-- end list -->

1.  Check to see if there is room (i.e. bandwidth) to process that
    resource request. If there is room, a **slot** is allocated to that
    instruction
2.  Attempt to process the command that the instruction requests of it.
    1.  If that command is successful, the resource mark the request as
        completed and return it back to the instruction.
    2.  If that command is unsuccessful, the resource will mark the
        request as not completed before returning it back to the
        instruction.

## Resource Internals

### Commands

Each Resource contains a enumerated list of commands that it recognizes.
The execute() function expects an instruction to request a resource to
perform one of the commands on this list for processing.

An example from resources/branch_predictor.hh:

```
   enum Command {
        PredictBranch,
        UpdatePredictor
    };
```

### Slots

Slots represent the bandwidth of the resource (or how many concurrent
instructions can access the resource). Each time an instruction makes a
request to a resource, the resource allocates a slot to that instruction
to represent the resources' bandwidth being used.

### Execute

Once an instruction is allocated a slot, it is eligible to be executed
within that resource. The execute function read the command from the
resource-request and then tries to perform the appropriate operation. If
the operation isnt completed successfully, execute() will mark the
completed variable "false" and if the the operation is successful, the
completed variable for that resource-request will marked as true.

An example from resources/branch_predictor.cc:

    void
    BranchPredictor::execute(int slot_num)
    {
        ...
        ResourceRequest* bpred_req = reqMap[slot_num];

        ...

        switch (bpred_req->cmd)
        {
          case PredictBranch:
            {

                ...

                bpred_req->done();
            }
            break;

          case UpdatePredictor:
            {
                ...

                bpred_req->done();
            }
            break;

          default:
            fatal("Unrecognized command to %s", resName);
        }
    }

## Resource Pool

The ResourcePool is the interface that the CPU must go through in order
to access a resource. The "pool" instantiates all resources, allocates
IDs to resources, and can schedule events to be processed on one or all
of the resources available to the CPU.

**Relevant source files:**

  - resource_pool.\[hh,cc\]
  - resource.\[hh,cc\]
  - resources/\*.\[hh,cc\]
  - pipeline_traits.\[hh,cc\]
  - cpu.\[hh,cc\]

### Interaction w/CPU

As stated before, the resource pool encapsulates all of the resources
that a CPU accesses. Consequently, the CPU must first access the
resource pool before receiving access to any specific resource.
Conversely, a resource can talk to the CPU by grabbing a pointer to the
CPU through the resource pool.

Typically, the CPU will access the resource pool in just a few cases:

1.  An instruction request in a pipeline stage
2.  Broadcasting a CPU Event
3.  Non-cycle-accurate instances such as initialization, system calls,
    and in the future checkpointing.

### Resource Pool Events

The resource pool allows the CPU to broadcast events that all of the
resources will see and process. This is beneficial to provide the CPU
ability to transfer information to resources and provide for resources
the ability to generically pass information to other resources.

By default, any CPU event that is scheduled (excluding the Tick event)
is copied and scheduled as a ResourcePool event. A list of CPU events is
available in cpu.hh:

```
    enum CPUEventType {
        ActivateThread,
        ActivateNextReadyThread,
        DeactivateThread,
        HaltThread,
        SuspendThread,
        Trap,
        InstGraduated,
        SquashFromMemStall,
        UpdatePCs,
        NumCPUEvents
    };
```

Each of these events are implemented as virtual functions in the
"Resource" base class, such that any resource that needs to react to any
of the aforementioned events simply needs to override the virtual
function in it's implementation.

The ResourcePool also has a few specific events that resources can
schedule themselves. The current list is found in resource_pool.hh:

```
    enum ResPoolEventType {
        InstGraduated = InOrderCPU::NumCPUEvents,
        SquashAll,
        UpdateAfterContextSwitch,
        Default
    };
```

## Predefined Resources

The following pipeline resources are defined for InOrderCPU:

  - Fetch Unit
  - Instruction Cache (I-Cache)
  - Branch Prediction Unit (BPred Unit)
  - Register File Manager (RF Manager)
  - Address Generation Unit (AGen Unit)
  - Execution Unit (EXU)
  - Integer Multiply and Divide Unit (Int MDU)
  - Data Cache (D-Cache)
  - Graduation Unit (Grad Unit)

## Defining Your Own Resources

The easiest way to define your own resource is to find a resource that
is similar to what you are trying to create, and then use that as a
template for your design.

More specifically, you'll need to derive from the "Resource" class and
then define your own resource-specific "execute" function. In the
simplest case, where your resource is of zero (same-cycle) latency, then
this should do the trick. If you resource processes requests on multiple
cycles, then a good example of that is the instruction buffer, the
caches, or the multiply/divide units.

Also note, for an instruction to use your resource, you need to :

1.  Add the resource to the resource pool
2.  Add the resource to that instruction's instruction schedule
