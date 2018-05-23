---
title: "Directed tests"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

The Directed Tester is for generating a stream of requests, where two
consecutive requests are separated by a fixed number of bytes. The
requests can be a mix of reads and writes, or they can be invalidations.
This can be useful in testing a prefetcher or for generally testing a
coherence protocol implemented in Ruby. The source files related to the
tester are present in the directory **src/cpu/testers/directedtest**.
The file **configs/examples/ruby_direct_test.py** is used for
configuration and execution of the test. For example, the following
command can be used for testing --

    ./build/X86/gem5.opt ./configs/example/ruby_direct_test.py

Though one can specify many different options to the directed tester,
some of them are note
worthy.

| Parameter          | Description                                                        |
| ------------------ | ------------------------------------------------------------------ |
| **-n, --num-cpus** | Number of cpus injecting load/store requests to the memory system. |
| **-m, --maxtick**  | Number of cycles to simulate.                                      |
| **-l, --checks**   | Number of requests to be performed.                                |
| **--test-type**    | SeriesGetx / SeriesGets / Invalidate                               |
| **--random_seed** | Seed for initialization of the random number generator.            |

