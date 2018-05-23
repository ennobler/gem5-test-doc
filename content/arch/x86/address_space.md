---
title: "Address space"
date: 2018-05-13T18:14:38-04:00
draft: false
weight: 10
---


X86_64 is defined to support physical memory addresses up to 52 bits
long. Because M5 uses 64 bit integers for addresses, every physical
address has 12 extra bits which aren't directly accessible to the
software running on the simulated CPU. M5 uses those bits to
differentiate between different physical address spaces which are
allocated for various purposes. In order to maximize the space given to
the actual address portion of the address, the bits that select the
address space grown down from the MSB as they're allocated. In effect,
the index of the address space is just written into the most significant
bits of the address in reverse, with it's least significant bit in the
addresses most significant bit. The currently defined address spaces are
the following:

  - 0x0000000000000000 - Physical memory.
  - 0x8000000000000000 - IO ports.
  - 0xC000000000000000 - PCI config space.
  - 0x2000000000000000 - The registers of all the local APICs. Each one
    gets a 2 page chunk of this space.
  - 0xA000000000000000 - Interrupt messages. Each APIC gets a portion of
    this space as well. The various addresses work like memory mapped
    registers and allow the APICs to send each other IPIs.
