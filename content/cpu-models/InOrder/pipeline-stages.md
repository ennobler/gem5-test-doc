---
title: "Pipeline stages"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

## Overview

Pipeline stages in the InOrder CPU are implemented as abstract
implementations of what a pipeline stage would be in any CPU model.
Typically, one would imagine a particularly pipeline stage being
responsible for:

(1) Performing specific operations such as "Decode" or "Execute" and
either

(2a) Sending that instruction to the next stage if that operation was
successful and the next stage's buffer has room for incoming
instructions

or

(2b) Keeping that instruction in the pipeline's instruction buffer if
that operation was unsuccesful or there is no room in the next stage's
buffer

The "PipelineStage" class maintains the functionality of (2a) and (2b)
but abstracts (1) out of the implementation. More specifically, no
pipeline stage is explicitly marked "Decode" or "Execute". Instead, the
PipelineStage class allows the instruction and it's corresponding
instruction schedule to define what tasks they want to do in a
particular stage.

Implementing the "PipelineStage" class in this manner allows for the
modeling of arbitrarily long pipelines (easy replication of pipeline
stages) as well as other potentially complex pipeline models.

*Relevant source files:*

  - first_stage.\[hh,cc\]
  - pipeline_stage.\[hh,cc\]
  - pipeline_traits.\[hh,cc\]
  - cpu.\[hh,cc\]

## Interstage Communication

Pipeline stages need to communicate with each other in a few scenarios:

  - Sending an instruction to the next stage
  - Sending a stall signal back to previous stages
  - Sending a squash (e.g. branch misprediction) signal back to previous
    stages

### Forwards Communication

This is the simplest case in which a PipelineStage has finished doing
all the processing it needs for an instruction and will place the
instruction in a queue (implemented through wires) for the next stage to
read from.

  - The information in the "wire" is taken from the "InterStageComm"
    struct found in "comm.hh".

### Backwards Communication

To handle backwards communication (stall/squash), the InOrderCPU uses
the concepts of a "wire" to pass information back to previous stages.
When a CPU is setup, each "wires" are defined with parameterized
latencies that govern how many ticks a stage must wait to retrieve the
information from that "wire". As an example, let's say Stage 3 wants to
send a squash signal to Stage 1. If the the user wants Stage 1 to see
information from Stage 3 the next clock tick, they would set that wire's
latency to 1. In this case, if Stage 3 wanted to squash instructions on
Cycle X, Stage 1 would operate undeterred on Cycle X, but notice the
squash signal on Cycle X+1.

## Pipeline Stage Processing

Each stage follows these generic steps each tick:

1.  Read stall and/or squash signals from stages further in the pipeline
      - If stall or squash is requested, then perform operation and do
        not process instructions that tick
      - If no stall or squash, continue processing.
2.  Select a thread to process instructions from
3.  Process all resource requests for an instruction and send
    instruction to next stage
      - If all resource requests are completed for an instruction,
        proceed to next instruction as long as the pipeline bandwidth is
        not used up.
      - If all resource requests are not completed for an instruction,
        save instruction in stage "skidBuffer" and stall remaining
        instructions in pipeline stage from that thread

### First Stage

The FirstStage derives the "PipelineStage" class and implements specific
operations that are needed for any first stage of the pipeline. Because
there is no stage in front of the FirstStage there are no instructions
to read from so the appropriate adjustments are made for this particular
stage to create a new instruction each pipeline tick instead of read it
from the previous stage.

## Implementations

In it's most basic incarnation, the In-Order model models a 5-stage
pipeline with the following stages:

  - Instruction Fetch (IF)
  - Instruction Decode (ID)
  - Execute (EX)
  - Memory Access (MEM)
  - Register Write Back (WB)

An example of a 9-stage pipeline is also available in the code
directory. To Test it out, you'll need to copy the overwrite the
"pipeline_traits.hh,cc" files with the 9-stage versions.

## Pipeline Customization

### Adding Your Own Stages

  - Python Configuration: TBD

<!-- end list -->

  - Instruction Schedule Information: Customizing the pipeline involves
    changing in what order an instruction will ask for it's resources.
    For example, the developer has the option to allow a load
    instruction to calculate it's address in stage 3 or stage 4. The
    developer would need to alter the instruction's schedule to enforce
    a particular pipeline configuration. Check out pipeline_traits.cc
    for more details on this process.
