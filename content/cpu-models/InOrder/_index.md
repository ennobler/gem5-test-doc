---
title: "In Order"
date: 2018-05-13T18:51:37-04:00
draft: false
chapter: true
weight: 20
---

## Overview

The InOrder CPU model was designed to provide a generic framework to
simulate in-order pipelines with an arbitrary ISA and with arbitrary
pipeline descriptions. The model was originally conceived by closely
mirroring the O3CPU model to provide a simulation framework that would
operate at the "Tick" granularity. We then abstract the individual
stages in the O3 model to provide [generic pipeline
stages](InOrder_Pipeline_Stages "wikilink") for the InOrder CPU to
leverage in creating a user-defined amount of pipeline stages.
Additionally, we abstract each component that a CPU might need to access
(ALU, Branch Predictor, etc.) into a "resource" that needs to be
requested by each instruction according to the
[resource-request](InOrder_Resource-Request_Model "wikilink") model we
implemented. This will potentially allow for researchers to model custom
pipelines without the cost of designing the complete CPU from scratch.

For more information, please check the following documentation about the
InOrder model, browse the code, and also access the gem5-users@m5sim.org
(standard usage) or gem5-dev@m5sim.org (for developer) mailing lists:

  - [Pipeline Stages](InOrder_Pipeline_Stages "wikilink")
  - [Resource-Request
    Modeling](InOrder_Resource-Request_Model "wikilink")
  - [Instruction Schedules & Pipeline
    Descriptions](InOrder_Instruction_Schedules "wikilink")
  - [A Day in the Life of an Instruction in the InOrderCPU model (Not
    Completed)](InOrder_Tutorial "wikilink")
  - Other Links:
      - Soumyaroop Roy has been kind enough to provide a
        \[[http://www.csee.usf.edu/~sroy/techres/m5_tests/"test-status-page](http://www.csee.usf.edu/~sroy/techres/m5_tests/%22test-status-page)"\]
        of the M5-Inorder model in the work he has been doing.

## Current Development

**Latest versions of the InOrderCPU model can be found in the gem5-dev
repository**

  - [InOrder ToDo List](InOrder_ToDo_List "wikilink")
