---
title: "Segmentation"
date: 2018-05-13T18:17:58-04:00
draft: false
weight: 60
---

## Segment bases

When computing an address for a load or store, the segment base is added
in before the address is sent to the CPU to actually perform the access.
This has several advantages. First, because there are no alignment
restrictions on segment bases, the virtual (pre-segmentation) address
for an access could be aligned but produce an unaligned linear
(post-segmentation) address. The opposite could also happen where an
unaligned access becomes aligned. Once outside of the instruction, the
majority of M5 doesn't know about segmentation and wouldn't be able to
handle those sorts of situations. In CPUs like the O3 model which can do
store to load forwarding, accesses to the same virtual address are
expected to refer to the same piece of memory. By applying segment bases
before the CPU gets the address, that remains true for x86. Because the
base address for a particular segment isn't always used, both the
intended base and the effective base are stored. The effective base is
the value that's actually added into a virtual address.

## Limit and attribute checks

Limit and attribute checks are performed by the TLB. The TLB is a
convenient place to perform any type of address validation which could
result in a fault, including these sorts of checks.

## Implemented segments

### Architected user segments

These segments are for code or data and are specified by the ISA.

  - CS - Code segment.
  - DS - Default data segment.
  - ES - Data segment. Implicitly used by string instructions.
  - SS - Stack segment.
  - FS, GS - Extra data segments.

### M5 internal data segments

These segments are specific to M5 and are used internally.

  - HS - A temporary segment register.
  - LS - A segment register which always has a flat segment with base
    address 0.

### Architected system segments

These segments are for system data structures and are specified by the
ISA..

  - TSL (LDT) - Local descriptor table.
  - TSG (GDT) - Global descriptor table.
  - TR (TSS) - Task state segment.
  - IDTR (IDT) - Interrupt descriptor table.

### Emulation memory

The MS segment is not actually a segment. Instead, it's a flag to the
TLB that a particular address should be decoded as an access to an
alternative address space. All alternative address spaces are 32 bits
wide, so the upper 32 bits are used to specify which one to use. The
following prefixes have been defined:

  - 0x100000000 - Originally planned for calls to CPUID. CPUID will
    likely be implemented as a special function unit/function, so this
    may never be used.
  - 0x200000000 - Access to an MSR. The MSR number needs to be scaled by
    the size of a MiscReg so that M5 doesn't get confused and think
    adjacent MSRs overlap.
  - 0x300000000 - IO port access.
