---
title: "Branch-delay slots"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

Since MIPS and SPARC use branch delay slots, we're faced with an
interesting issue on how to implement them correctly. There are two
issues: basic support for branch delay slots, and support for
conditionally executed delay-slot instructions (SPARC "annulled" delay
slots).

## Basic functionality

In the context of M5 instruction execution, PC is the current
instruction's PC and NPC is the PC of the next instruction. Conventional
non-delayed branches (as in Alpha) write to NPC to change the next
instruction executed. Conceptually all non-branch instructions also
update NPC to PC+4, though currently this is implemented in the CPU
model and not in the ISA. (Obviously this would have to change if we had
variable-length instructions.) Thus between each pair of instruction
executions we simply set PC to NPC to advance the program counter.

In delayed-branch architectures, we need an additional PC, NNPC, which
is the PC of the "next next" instruction. A simple delayed branch can be
implemented by writing the target address to NNPC instead of NPC.
Non-branch instructions set NNPC to NPC+4. Between each pair of
instruction executions we set PC to NPC and NPC to NNPC. Note that this
model (I think) automatically supports some of the funky SPARC delayed
branch semantics, such as having a branch in the delay slot of another
branch. In MIPS, executing a branch in a branch delay slot results in
UNDETERMINED behavior.

## Conditional delay slot instructions

Things get more complicated when the delay-slot instruction is
effectively predicated on the branch direction. SPARC supports
"annulled" branches in which the delay-slot instruction is not executed
if the branch is not taken. (Actually for unconditional branches the
annul bit means never execute the delay slot instruction, which is kind
of the opposite of what it means for conditional branches.)

Apparently MIPS is even more complex, with bits that allow the
delay-slot instruction to be predicated in either direction (execute
only if taken or only if not taken).(Korey, can you fill in some
specifics here?)

  -
    I dont think MIPS allows the delay-slot instructions to be
    predicated. (I may have miscommunicated a bit on my original post)
    MIPS actually has separate opcodes to distinguish between branches
    where the delay-slot is always executed and instructions where the
    delay-slot is conditionally instructed. Additionally, I gather that
    there is no difference between how MIPS and SPARC treats these
    delay-slot instructions. The only difference is between the
    annulled-bit and the opcode to specify which "handling" is required.
    Luckily, the decoder/ISA scheme makes the annulled-bit/opcode
    difference a non-issue. --[Ksewell](User:Ksewell "wikilink") 18:37,
    16 February 2006 (EST)

<!-- end list -->

  -

      -
        By predicated I just meant conditionally executed. The only
        other possibility that MIPS might have that SPARC doesn't is a
        flavor of branch that only executes the delay slot instruction
        if the branch is \*not\* taken (as opposed to only if the branch
        \*is\* taken). --[Stever](User:Stever "wikilink") 19:52, 16
        February 2006 (EST)

The "formal semantics" of SPARC delayed branches are shown in Table 13
of Section 6.3.4 of the SPARC v9 reference manual (p. 100 in
\[<http://www.sparc.com/standards/SPARCV9.pdf>| this pdf\]). Basically
there are four possibilities, plus two special cases for returning from
traps. Following the SPARC conventions, "B" is an unconditional branch
while "Bcc" is a conditional branch, and "Tcc" is a conditional trap.
(Yes, SPARC has a branch that's unconditionally not taken... don't ask
why.)

| SPARC Instructions                                                               | NPC        | NNPC           |
| -------------------------------------------------------------------------------- | ---------- | -------------- |
| Non-control-transfer instructions, non-taken non-annulled B & Bcc, non-taken Tcc | NPC        | NPC + 4        |
| Taken Bcc, taken non-annulled B, CALL, RETURN, JMP                               | NPC        | EA             |
| Non-taken annulled B & Bcc                                                       | NPC + 4    | NPC + 8        |
| Taken annulled B, taken Tcc                                                      | EA         | EA + 4         |
| DONE                                                                             | TNPC\[TL\] | TNPC\[TL\] + 4 |
| RETRY                                                                            | TPC\[TL\]  | TNPC\[TL\]     |

\-DONE and RETRY are two flavors of "return from exception" in SPARC.
--[Stever](User:Stever "wikilink") 19:52, 16 February 2006
(EST)

| MIPS Instructions                                                                             | NPC        | NNPC           |
| --------------------------------------------------------------------------------------------- | ---------- | -------------- |
| Non-control-transfer instructions, non-taken Branches (B) & Branch-Likely(Bl), non-taken Trap | NPC        | NPC + 4        |
| Taken Bl, taken B, CALL, RETURN, JMP                                                          | NPC        | EA             |
| Non-taken B & Bl                                                                              | NPC + 4    | NPC + 8        |
| Taken B, Bl, taken Trap                                                                       | EA         | EA + 4         |
| ERET, DERET                                                                                   | TNPC\[TL\] | TNPC\[TL\] + 4 |

\-ERET and DERET are the MIPS return from exceptions. DERET is return
from debug exception.--[Ksewell](User:Ksewell "wikilink") 16:32, 17
February 2006 (EST)

It seems like the easiest way to implement this is to expose NPC and
NNPC to the ISA definition and let each instruction set either or both
of these explicitly as necessary (with the default behavior being no
change to NPC and NNPC = NPC + 4). Note that this model should work just
fine for FastCPU and SimpleCPU where each instruction executes its
semantic definition atomically (no pipelining), without any additional
state beyond NNPC.

I think the best way to handle this for pipelined (incl. out-of-order)
execution is to treat it as an extension of branch prediction. Right now
we predict NPC and catch mispredictions by comparing the predicted &
actual NPC values. For SPARC and MIPS we'll have to predict both NPC and
NNPC and catch mispredictions by comparing the predicted & actual values
of both of them as well. (It appears that comparing the predicted &
actual NNPC values might be enough, since looking at the table it's hard
to see a case where you could get NNPC right but NPC wrong. But I
wouldn't count on that.) The prediction mechanism really just needs one
additional bit to distinguish among the first four cases above as
opposed to the current two cases (NPC = PC+4 vs. NPC = EA).

  -
    I believe it's impossible to get the NNPC right and NPC wrong since
    when the CPU model sees a branch it makes the prediction for that
    branch in the fetch stage. If that branch is taken then the other
    instructions aren't even allowed into the pipeline.
    --[Ksewell](User:Ksewell "wikilink") 18:37, 16 February 2006 (EST)

<!-- end list -->

  -

      -
        I'm more worried about a situation where there's a branch in a
        branch delay slot or a return from exception into a branch delay
        slot or something like that... things that MIPS generally
        doesn't have to worry about but SPARC does. Just to be safe we
        should always compare both. --[Stever](User:Stever "wikilink")
        19:52, 16 February 2006 (EST)

## Other ideas

1\. Keep some type of "condition-code"-type register(s) in the miscRegs
file.

2\. For branch delay slot instructions, we can add an extra source
register to them. The source register would simply be the destination
register of the preceding branch.

I think \#2 would work and the extra source register could be added in
the rename stage...

  -
    The catch here is that just looking at the target may not be
    sufficient, depending on how you do it. For example if you had this:

`    0x1000: beq 0x1008`
`    0x1004: add`
`    0x1008: sub`

  -
    then the 'add' could look at the branch target (NPC) but it would be
    0x1008 regardless of whether the branch was taken or not taken.

The only drawback is that it wouldnt necessarily allow a branch delay
slot instruction to execute the same cycle as it's branch and that kind
of defeats the purpose of having delay slot instructions.

  -
    Just to clarify: delay slots really make sense only for single-issue
    in-order pipelines without BTBs, where the delay slot instruction
    executes in the cycle \*after\* the branch, hiding the branch-taken
    bubble that comes from not having a BTB. In this situation the
    squashing delayed branches are not that hard to implement, as the
    branch always resolves before the delay slot instruction hits
    writeback, so you can execute it speculatively and just not write
    back its result. They were never intended to be executed in the same
    cycle as the branch.

## CPU independent implementation

Should code for these be implemented in the CPU with \#define MIPS or
\#define SPARC ... Or should there be architecture-specific files and/or
functions for this branch-delay code?

  -
    It seems like the main question is do we need an NNPC or not. We
    could probably cover most cases by putting in an NNPC but not using
    it when we don't need it (e.g. in Alpha), though this could be
    confusing. Unless we do extra work to maintain NNPC in Alpha though
    there will be a difference when it comes to doing branch predictions
    in pipelined CPU models (see above). If we can generalize both MIPS
    and SPARC to a unified model, then we could have a single flag that
    enables or disables that model and set it in MIPS and SPARC and
    leave it cleared in Alpha.

<!-- end list -->

  -
    Would it be possible for the PC to be considered a control register
    which lives in the ISA? There could be special getPC/stepPC/whatever
    functions to hook into for the CPU, and NPC and NNPC could be
    maintained totally internally to the ISA and wouldn't exist if they
    aren't needed. I'm not sure how this complicates things for the CPU
    model, though. --[Gblack](User:Gblack "wikilink") 13:26, 16 February
    2006 (EST)

<!-- end list -->

  -
    It's not entirely clear what would be less complex. Regardless of
    whether or not the PC is a control register, there will still have
    to be differences between the branch prediction in a pipelined
    model. One problem with the PC and NPC and NNPC living totally in
    the ISA is with dynamic instructions. Currently they hold a variety
    of information that is necessary to keep track of with the
    instruction, such as PC and NextPC. If you were to push the
    PC/NPC/NNPC to the ISA level, then you'd also have to define within
    the ISA some sort of struct that is all the ISA state information
    that needs to be carried along with a dynamic instruction. DynInst
    already has quite a bit of indirection to it, so I'm a bit hesitant
    to add another level of indirection. It may also be difficult to get
    the interface correct; for certain registers (such as the PC) you'd
    have to call xc-\>getISAState()-\>setPC(pc), while other registers
    would just be xc-\>setIntReg(5, 1000). It'd be nice to keep them
    uniform so that the programmer doesn't have to keep checking if the
    register is in the ISA or can be accessed normally. On the other
    hand, this ISA-dependent state struct may be useful to store, for
    example, updates to IPRs which won't happen until commit time. I'm
    really not sure at the moment...it could be useful, or it could just
    be more complication. --[Ktlim](User:Ktlim "wikilink") 15:44, 16
    February 2006 (EST)

<!-- end list -->

  -
    I'm fine with RegFile being an ISA-dependent type. As long as it
    stays that way then each ISA can choose whether to have an NNPC or
    not. They can all export get/set NNPC functions and the Alpha
    versions can panic. If we were going to have ten different ISAs with
    8 different PC/NPC schemes it might be worth more effort, but if
    there are just two schemes and no third one on the horizon that I
    can see it's not worth the effort to get all abstract about it,
    especially for something so common and potentially performance
    sensitive. --[Stever](User:Stever "wikilink") 21:09, 16 February
    2006 (EST)

## Hazard Clearing Instructions

MIPS has hazard clearing instructions which clear instruction and
execution hazards (most of the hazards have to do with the special
system (or CP0) registers). MIPS execution hazards are defined as "those
created by the execution of one instruction, and seen by the execution
of another instruction" and instruction hazards as " those created by
the execution of one instruction, and seen by the instruction fetch of
another instruction". A table of the hazard clearing instructions is
presented below and a list of all the hazard cases (e.g.
disable-interrupt (producer) to interrupt-instruction(consumer)) can be
found in [7.0 of vol.III of the MIPS
manual](http://www.mips.com/content/Documentation/MIPSDocumentation/ProcessorArchitecture/ArchitectureProgrammingPublicationsforMIPS32/MD00090-2B-MIPS32PRA-AFP-02.50.pdf?agree=yes%7Csection).

The reason I mention this on this WIKI page is because the jalr_hb and
jr_hb instructions require the hazard clearing to take place after the
branch delay-slot gets executed. I was thinking to do one of two things:

1\. Schedule a CPU event for cycle N+2 (where N is the cycle of the jump
instruction executing). However, this solution quickly breaks down if
the branch delay-slot instruction takes more than one cycle.

2\. Create a flag in the CPU for a pending hazard clearing.

With respect to the Simple CPU model, I am not sure if a complicated
solution is needed in the first place so I would be up for implementing
\#2. Currently, I have these hazard clearing instructions call a
function called "clear_exe_inst_hazards()". This function can be
configured later depending on our design decision.

Thoughts???

