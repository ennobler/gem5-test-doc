---
title: "Instruction schedules & pipeline definition"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

## Instruction Schedules & Pipeline Definitions Overview

At the heart of the InOrderCPU model is the concept of **Instruction
Schedules (IS)**. Instruction schedules create the generic framework
that allow for developer's to make a custom pipeline. A pipeline
definition can be seen as a collection of instruction schedules that
govern what an instruction will do in any given stage and what stage
that instruction will go to next.

In general, each instruction has a stage-by-stage list of tasks that
need to be accomplished before moving on to the next stage. This list we
refer to as the instruction's schedule. Each list is composed of
"ScheduleEntry"s that define a task for the instruction to do for a
given pipeline stage.

Instruction scheduling is then divided into a *front-end schedule* (e.g.
Instruction Fetch and Decode) which is uniform for all the instructions,
and a *back-end schedule*, which varies across the different
instructions (e.g. a 'addu' instruction and a 'mult' instruction need to
access different resources).

The combination of a front-end schedule and a back-end schedule make up
the instruction schedule. Ideally, changing the pipeline can be as
simple as editing how a certain class of instructions operate by editing
the instruction schedule functions.

***Relevant source files:***

  - pipeline_traits.\[hh,cc\]
  - resource.\[hh,cc\]
  - resources/\*.\[hh,cc\]
  - resource_pool.\[hh,cc\]
  - cpu.\[hh,cc\]

## Schedule Entries

Schedule Entries denote a particular task for an instruction to process
in a pipeline stage.

The following code snippet shows the definition of a Schedule Entry:

    struct ScheduleEntry {
            ScheduleEntry(int stage_num, int _priority, int res_num, int _cmd = 0,
                          int _idx = -1) :
                stageNum(stage_num), resNum(res_num), cmd(_cmd),
                idx(_idx), priority(_priority)
            { }

            virtual ~ScheduleEntry(){}

            // Stage number to perform this service.
            int stageNum;

            // Resource ID to access
            int resNum;

            // See specific resource for meaning
            unsigned cmd;

            // See specific resource for meaning
            unsigned idx;

            // Some Resources May Need Priority?
            int priority;
        };

Instruction Schedules consist of lists of "ScheduleEntry"s specific to
each instruction. Resource IDs are found in the pipeline_traits.hh and
must match those that are allocated in resource_pool.cc. Each resource
defines which commands are eligible to be processed for that particular
resource.

A pipeline stage processes an instruction by using the instruction
schedule schedule to see which resource it needs to process next.

## Front-End Schedules

Front End Schedules are composed of tasks that all instructions must
perform. This typically consists of instruction fetch and decode. A new
front end schedule is created every time a new instruction is created.

### Front-end Schedule Example

  - Front-end Schedule
      - The front-end schedule comprises of the IF and ID stages
          - IF
              - NPC is updated by the Fetch unit
              - Instruction fetch from the I-Cache is initiated
          - ID
              - Instruction fetch is completed by I-Cache
              - Instruction decode is performed by the Decode unit
              - Branch prediction is performed by the BPred unit
              - Target PC is updated by the Fetch unit

An example of this is in this code snippet from pipeline_traits.cc:

```
    InstStage *F = inst->addStage();
    InstStage *D = inst->addStage();

    // FETCH
    F->needs(FetchSeq, FetchSeqUnit::AssignNextPC);
    F->needs(ICache, CacheUnit::InitiateFetch);

    // DECODE
    D->needs(ICache, CacheUnit::CompleteFetch);
    D->needs(Decode, DecodeUnit::DecodeInst);
    D->needs(BPred, BranchPredictor::PredictBranch);
    D->needs(FetchSeq, FetchSeqUnit::UpdateTargetPC);
```

  - Note an InstStage object has been introduced to help create
    instruction schedules easier.

## Back-end Schedule

Back end schedules vary depending on the instruction type. Typically,
this consists of the pipeline after (or including) the decode stage
since we can identify whether an instruction is a load,store, branch,
etc. at that point.

### Back-end Schedule Example

  - Back-end Schedule
      - The back-end schedule comprises of the ID, EX, MEM, and WB
        stages
          - ID
              - For non-store instructions, the source registers, if
                any, are read by the RF Manager
              - For load instructions, address generation is performed
                by the AGEN unit and data read from the D-Cache is
                initiated
              - The rest of the instructions are executed in the
                execution units
                  - Single cycle operations are sent to the integer EXU
                  - Execution is initiated for the multicycle/pipelined
                    operations
          - EX
              - Execution is finished for the multicycle/pipelined
                operations
              - For load instructions, data read from the D-Cache is
                completed
              - For store instructions, the following tasks are
                performed
                  - The source registers are read by the RF manager
                  - Address generation is performed by the AGen unit
                  - Data write into the D-Cache is initiated
          - MEM
              - For store instructions, data write into the D-Cache is
                completed
          - WB
              - Destination registers are written into by the RF manager
              - The instruction is graduated by the Grad unit

Typical code might for the Execute stage might be:

```
   if ( inst->isNonSpeculative() ) {
        // skip execution of non speculative insts until later
    } else if ( inst->isMemRef() ) {
        if ( inst->isLoad() ) {
            X->needs(AGEN, AGENUnit::GenerateAddr);
        }
    } else {
        X->needs(ExecUnit, ExecutionUnit::ExecuteInst);
    }
```
