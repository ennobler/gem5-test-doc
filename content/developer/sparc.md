---
title: "SPARC Curiousities"
date: 2018-05-13T18:51:37-04:00
draft: false
---

## Ancillary State Registers

  - Floating-Point State Register (FSR) - holds 4 sets of condition
    codes, exception status, trap enable, rounding direction, etc. Which
    means we need to rename it, but we need to copy parts of the
    previous state into it every time.

<!-- end list -->

  - Y Register - Why ohh why did the make this register. Holds upper 32
    bits for some multiple/div operations. For multiply the upper 32
    bits of the result is placed in here and the entire result is placed
    in the dest register. For some divides the upper 32 bits are read
    from here and then afterwards the state is unpredictable. Depending
    on how common these depricated operations are, we may be able to
    flush after each one instead of renaming the register.

<!-- end list -->

  - Condition Code Register (CCR) - 8 bit register 4 bits for 64bit CC,
    4 bits for 32bit CC. We'll have to rename them and in the case of
    sparc, they are all written at once so we don't have to rename some
    of them.

<!-- end list -->

  - Address space identifies (ASI) - Distinguishes between different
    address space, what kind of translation is done, is the MMU used,
    also used to map control register for the processor. 255 ASIs 128 of
    which are unprivileged, the rest required privilidge or
    hyperpriviledge. Accessing a ASI you don't have privileges for
    results in a trap. ASI register is used when you issue a load/store
    alternate instruction. This could be preformance critical and we may
    want to rename it.

<!-- end list -->

  - TICK Register - Counts Strand? Ticks, starting at 0 when the
    processor boots. Upper most bit controls if unprivileged software
    can read the bit. Make instruction that writes serializing?

<!-- end list -->

  - Floating-Point Register State (FPRS) - enable/disable the FPU.
    serializing?

<!-- end list -->

  - Performace Control Register (PCR) - don't implement?

<!-- end list -->

  - Performance Instrumentation Conter (PIC) - don't implement?

<!-- end list -->

  - General State Register (GSR) - holds info for VIS (SIMD)
    instructions. lower 32 bits flags and such, upper 32 bits set by
    BMASK instruction and used by BSHUFFLE instruction. Low bits can
    probably serialize on being written, may want to rename upper bits?

<!-- end list -->

  - SOFTINT - read current interrupts, write to cause an software
    interrupt, two bits to enable/disable timer interrupts based of TICK
    register. Make writes serializing? Has two pseudo registers that
    writes to them either set or clear bits in the actually register.

<!-- end list -->

  - Tick Compare (TICK_CMPR) - when lower 63 bits match lower 63 bits
    of TICK register level 14 interrupt occurs (if everything is
    enabled) bit 63 + SOFTINT.tim and PIL determine if interrupt
    actually happens. Writes serializing?

<!-- end list -->

  - STICK,STICK_CMPR - same as tick, tick compare but synchornized
    across all processors.

## Priviliged State Registers

Writes always serializing?

  - TPC\[1..MAXTL\] - PC from the previous trap level, written on trap,
    based on current TL only one readable.
  - TNPC\[1..MAXTL\] - NPC from the previous trap level, written on
    trap, based on current TL only one readable.
  - Trap State (TSTATE)\[1..MAXTL\] - various bits from registered
    copied in when a trap occurs. GL, CCR, ASI, PSTATE, and CWP.
  - Trap Type (TT)\[1..MAXTL\] - Specified what causes the trap, based
    on current TL only one readable.
  - Trap Base Address (TBA) - virtual address of the trap table.
  - Processor State (PSTATE) - Memory ordering model, address modes,
    another FPU enable bit, etc
  - TL - controls which of the above registers is accessed, values you
    can write into it dependent on current mode, autoincremented on trap
  - PIL - Like IPL on alpha
  - GL - chooses which set of global registers are enabled, values you
    can write into it dependent on current mode, autoincremented on trap
  - Hyperprivileged State (HPSTATE) - various little bits.
  - Hyperprivileged Trap State (HTSTATE) - like Trap state, but for the
    HPSTATE register
  - Hyperprivileged Interrupt Pending (HINTP) - if an STICK and
    HSTICK_CMPR are equal. (timer interrupt for Hypervisor).
  - Hyperpriviliged Trap Based Address ( HTBA) - same as TBA for HP
    state
  - Hyperpriviliged Implementation Version Register (HVER) - read only
    bits that describes the current version, manufacturer, MAXGL, MAXTL,
    MAXWIN.
  - Hyperpriviliged Tick Compare (HSTICK_CMPR) - like STICK CMPR for
    hyperprivileged code.

## Instructions

  - CASA/CASXA - Compare and swap - IF: rs2 == MEM\[rs1\]:
    MEM\[rs1\]\[31..0\] \<-\> rd\[31..0\]; rd\[63..32\] \<- 0 ELSE:
    rd\[31..0\]\<- MEM\[rs1\]\[31..0\] ; rd\[63..32\] \<- 0
  - FLUSH - make writes to program memory visable in the icache.
  - (LD|ST)TD - Load/Store twin double word instructions operate on
    128bits loading/storing them from two adjacent registers in one
    atomic operation so you don't have to do any locking for a TLB
    update. yay.

