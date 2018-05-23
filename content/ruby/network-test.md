---
title: "Network test"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

This is a dummy cache coherence protocol that is used to operate the
ruby network tester. The details about running the network tester can be
found [here](Ruby_Network_Test "wikilink").

### Related Files

  - **src/mem/protocols**
      - **Network_test-cache.sm**: cache controller specification
      - **Network_test-dir.sm**: directory controller specification
      - **Network_test-msg.sm**: message type specification
      - **Network_test.slicc**: container file

### Cache Hierarchy

This protocol assumes a 1-level cache hierarchy. The role of the cache
is to simply send messages from the cpu to the appropriate directory
(based on the address), in the appropriate virtual network (based on the
message type). It does not track any state. Infact, no CacheMemory is
created unlike other protocols. The directory receives the messages from
the caches, but does not send any back. The goal of this protocol is to
enable simulation/testing of just the interconnection network.

### Stable States and Invariants

| States | Invariants                        |
| ------ | --------------------------------- |
| **I**  | Default state of all cache blocks |

### Cache controller

  - Requests, Responses, Triggers:
      - Load, Instruction fetch, Store from the core.

The network tester (in src/cpu/testers/networktest/networktest.cc)
generates packets of the type **ReadReq**, **INST_FETCH**, and
**WriteReq**, which are converted into **RubyRequestType:LD**,
**RubyRequestType:IFETCH**, and **RubyRequestType:ST**, respectively, by
the RubyPort (in src/mem/ruby/system/RubyPort.hh/cc). These messages
reach the cache controller via the Sequencer. The destination for these
messages is determined by the traffic type, and embedded in the address.
More details can be found [here](Ruby_Network_Test "wikilink").

  - Main Operation:
      - The goal of the cache is only to act as a source node in the
        underlying interconnection network. It does not track any
        states.
      - On a **LD** from the core:
          - it returns a hit, and
          - maps the address to a directory, and issues a message for it
            of type **MSG**, and size **Control** (8 bytes) in the
            request vnet (0).
          - Note: vnet 0 could also be made to broadcast, instead of
            sending a directed message to a particular directory, by
            uncommenting the appropriate line in the *a_issueRequest*
            action in Network_test-cache.sm
      - On a **IFETCH** from the core:
          - it returns a hit, and
          - maps the address to a directory, and issues a message for it
            of type **MSG**, and size **Control** (8 bytes) in the
            forward vnet (1).
      - On a **ST** from the core:
          - it returns a hit, and
          - maps the address to a directory, and issues a message for it
            of type **MSG**, and size **Data** (72 bytes) in the
            response vnet (2).
      - Note: request, forward and response are just used to
        differentiate the vnets, but do not have any physical
        significance in this protocol.

### Directory controller

  - Requests, Responses, Triggers:
      - **MSG** from the cores

<!-- end list -->

  - Main Operation:
      - The goal of the directory is only to act as a destination node
        in the underlying interconnection network. It does not track any
        states.
      - The directory simply pops its incoming queue upon receiving the
        message.

### Other features

  -   - This protocol assumes only 3 vnets.
      - It should only be used when running the ruby network test.
