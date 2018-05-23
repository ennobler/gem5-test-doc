---
title: "Cache Replacement Policies"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

gem5 has multiple implemented replacement policies. Each one uses its
specific replacement data to determine a replacement victim on
evictions.

All of the replacement policies prioritize victimizing invalid blocks.

Altough most of the replacement policies don't need a counter of the
number of references to an entry, or the insertion timestamp, they are
by default updated by every replacement policy, due to their usage in
the statistics.

## Random

The simplest replacement policy; it does not need replacement data, as
it randomly selects a victim among the candidates.

## Least Recently Used (LRU)

Its replacement data consists of a last touch timestamp, and the victim
is chosen based on it: the oldest it is, the more likely its respective
entry is to be victimized.

## Bimodal Insertion Policy (BIP)

The [Bimodal Insertion
Policy](https://dl.acm.org/citation.cfm?id=1250709) is similar to the
LRU, however, blocks have a probability of being inserted as the MRU,
according to a bimodal throttle parameter (btp). The highest btp is, the
highest is the likelihood of a new block being inserted as MRU.

## LRU Insertion Policy (LIP)

The [LRU Insertion Policy](https://dl.acm.org/citation.cfm?id=1250709)
consists of a LRU replacement policy that instead of inserting blocks
with the most recent last touch timestamp, it inserts them as the LRU
entry. On subsequent touches to the block, its timestamp is updated to
be the MRU, as in LRU. It can also be seen as a BIP where the likelihood
of inserting a new block as the most recently used is 0%.

## Most Recently Used (MRU)

The Most Recently Used policy chooses replacement victims by their
recency, however, as opposed to LRU, the newest the entry is, the more
likely it is to be victimized.

## Least Frequently Used (LFU)

The victim is chosen using the reference frequency. The least referenced
entry is chosen to be evicted, regardless of the amount of times it has
been touched, or how long has passed since its last touch.

## First-In, First-Out (FIFO)

The victim is chosen using the insertion timestamp. If no invalid
entries exist, the oldest one is victimized, regardless of the amount of
times it has been touched.

## Second-Chance

The Second-Chance replacement policy is similar to FIFO, however entries
are given a second chance before being victimized, being re-inserted at
the end of the FIFO if its second chance bit is set. When an entry is
re-inserted, its second chance bit is cleared, and the entry is threated
as in the FIFO policy (unless it is touched again, in which case its
second chance bit is set again).

## Not Recently Used (NRU)

Not Recently Used (NRU) is an approximation of LRU that uses a single
bit to determine if a block is going to be re-referenced in the near or
distant future. If the bit is 1, it is likely to not be referenced soon,
so it is chosen as the replacement victim. When a block is victimized,
all its co-replacement candidates have their re-reference bit
incremented.

## Re-Reference Interval Prediction (RRIP)

[Re-Reference Interval Prediction
(RRIP)](https://dl.acm.org/citation.cfm?id=1815971) is an extension of
NRU that uses a re-reference prediction value to determine if blocks are
going to be re-used in the near future or not. The higher the value of
the RRPV, the more distant the block is from its next access. From the
original paper, this implementation of RRIP is also called Static RRIP
(SRRIP), as it always inserts blocks with the same RRPV.

## Bimodal Re-Reference Interval Prediction (BRRIP)

[Bimodal Re-Reference Interval Prediction
(BRRIP)](https://dl.acm.org/citation.cfm?id=1815971) is an extension of
RRIP that has a probability of not inserting blocks as the LRU, as in
the Bimodal Insertion Policy. This probability is controlled by the
bimodal throtle parameter (btp).

