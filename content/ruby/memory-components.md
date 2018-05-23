---
title: "Deprecated"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---


## System

This is a high level container for few of the important components of
the Ruby which may need to be accessed from various parts and components
of Ruby. Only **ONE** instance of this class is created. The instance of
this class is globally available through a pointer named
***g_system_ptr***. It holds pointer to the Ruby's profiler object.
This allows any component of Ruby to get hold of the profiler and
collect statistics in a central location by accessing it though
***g_system_ptr***. It also holds important information about the
memory hierarchy like Cache blocks size, Physical memory size and makes
them available to all parts of Ruby as and when required. It also holds
the pointer to the on-chip network in Ruby. Another important objects
that it hold pointer to is the Ruby's wrapper for the simulator event
queue (called RubyEventQueue). It also contains pointer to the simulated
physical memory (pointed by variable name ***m_mem_vec_ptr***). Thus
in sum, System class in Ruby just acts as a container to pointers to
some important objects of Ruby's memory system and makes them available
globally to all places in Ruby through exposing it self through
***g_system_ptr***.

### Parameters

1.  ***random_seed*** is seed to randomize delays in Ruby. This allows
    simulating multiple runs with slightly perturbed delays/timings.
2.  ***randomization*** is the parameter when turned on, asks Ruby to
    randomly delay messages. This falg is useful when stress testing a
    system to expose corner cases. This flag should NOT be turned on
    when collecting simulation statistics.
3.  ***clock*** is the parameter for setting the clock frequency
    (on-chip).
4.  ***block_size_bytes*** specifies the size of cache blocks in
    bytes.
5.  ***mem_size*** specifies the physical memory size of the simulated
    system.
6.  ***network*** gives the pointer to the on-chip network for Ruby.
7.  ***profiler*** gives the pointer to the profiler of Ruby.
8.  ***tracer*** is the pointer to the Ruby 's memory request tracer.
    Tracer is primarily used to playback memory request trace in order
    to warm up Ruby's caches before actual simulation starts.

### **Related files**

  - **src/mem/ruby/system**
      - **System.hh/cc**: contains code for the System
      - **RubySystem.py** :the corresponding python file with
        parameters.

## Sequencer

***Sequencer*** is one of the most important classes in Ruby, through
which every memory request must pass through at least twice -- once
before getting serviced by the cache coherence mechanism and once just
after being serviced by the cache coherence mechanism. There is one
instantiation of Sequencer class for each of the hardware thread being
simulated. For example, if we are simulating a 16-core system with each
core has single hardware thread context, then there would be 16
Sequencer objects in the system, with each being responsible for
*managing* memory requests (Load, Store, Atomic operations etc) from one
of the given hardware thread context. *ith* Sequencer object handles
request from only *ith* hardware context (in case of above example it is
*ith* core). Each Sequencer assumes that it has access to the L1
Instruction and Data caches that are attached to a given core.

Following are the primary responsibilities of Sequencer:

1.  Injecting and accounting for all memory request to the underlying
    cache hierarchy (and coherence).
2.  Resource allocation and accounting (e.g. makes sure a particular
    core/hardware thread does not have more than specified number of
    outstanding memory requests).
3.  Making sure atomic operations are handled properly.
4.  Making sure that underlying cache hierarchy and coherence protocol
    is making forward progress.
5.  Once a request is serviced by the underlying cache hierarchy,
    Sequencer is responsible for returning the result to the
    corresponding port of the frontend (i.e.M5).

### **Parameters**

1.  ***icache*** is the parameter where the L1 Instruction cache pointer
    is passed.
2.  ***dcache*** is the parameter where the L1 Data cache pointer is
    passed.
3.  ***max_outstanding_requests*** is the parameter for specifying
    maximum allowed number of outstanding memory request from a given
    core or hardware thread.
4.  ***deadlock_threshold*** is the parameter that specifies number of
    cycle (ruby cycles) after which if a given memory request is not
    satisfied by the cache hierarchy, a possibility of deadlock (or lack
    of forward progress) is declared.

### **Related files**

  - **src/mem/ruby/system**
      - **Sequencer.hh/cc**: contains code for the Sequencer
      - **Sequencer.py** :the corresponding python file with parameters.

### **More detailed operation description**

In this section, we will describe the operations of Sequencer in more
details.

  - **Injection of memory request to cache hierarchy, request accounting
    and resource allocation:**

<!-- end list -->

  -
    The entry point for a memory request to the Sequencer is method
    called ***makeRequest***. This method is called from the
    corresponding RubyPort (Ruby's wrapper for M5 port). Once it gets
    the request, Sequencer checks for whether required resource
    limitations need to be enforced or not. This is done by calling a
    method called ***getRequestStatus***. In this method, it is made
    sure that a given Sequencer (i.e. a core or hardware thread context)
    can does NOT issue multiple simultaneous requests to same cache
    block. If the current request indeed for a cache block for which
    another request is still pending from the same Sequencer then the
    current request is not issued to the cache hierarchy and instead the
    current request wait for the previous request from the same cache
    block to satisfied first. This is done in the code by checking for
    two requesting accounting table called *m_writeRequestTable* and
    *m_readRequestTable* and setting the status to
    *RequestSatus_Aliased*. It also makes sure that number of
    outsatnding memory request from a given Sequencer does not
    overshoot.
    If it is found that the current request won't violate any of the
    above constrained then the request is registered for accounting
    purposes. This is done by calling the method named
    ***insertRequest***. In this method, depending upon the type of the
    request, an entry for the request is created in the either
    *m_writeRequestTable* or *m_readRequestTable*. These two tables
    keep record for write request and read requests, respectively, that
    are issued to the cache hierarchy by the given Sequencer but still
    to be satisfied. Finally the memory request is finally pushed to the
    cache hierarchy by calling the method named ***issueRequest***. This
    method is responsible of creating the request structure that is
    understood by the underlying SLICC generated coherence protocol
    implementation and cache hierarchy. This is done by setting the
    request type and mode accordingly and creating an object of class
    *RubyRequest* for the current request. Finally, L1 Instruction or L1
    Data cache accesses latencies are accounted for an the request is
    pushed to the Cache hierarchy and the coherence mechanism for
    processing. This is done by *enqueue*-ing the request to the pointer
    to the mandatory queue (*m_mandatory_q_ptr*). Every request is
    passed to the corresponding L1 Cache controller through this
    mandatory queue. Cache hierarchy is then responsible for satisfying
    the request.

<!-- end list -->

  - ''' Deadlock/lack of forward progress detection: '''

<!-- end list -->

  -
    As mentioned earlier, one other responsibility of the Sequencer is
    to make sure that Cache hierarchy is making progress in servicing
    the memory requests that have been issued. This is done by
    periodically waking up and scanning through the
    *m_writeRequestTable* and *m_readRequestTable* tables. which holds
    the currently outstanding requests from the given Sequencer and
    finds out which requests have been issued but have not satisfied by
    the cache hierarchy. If it finds any unsatisfied request that have
    been issues more than *m_deadlock_threshold* (parameter) cycles
    back, it reports a possible deadlock by the Cache hierarchy. Note
    that, although it reports possible deadlock, it actually detects
    lack of forward progress. Thus there may be false positives in this
    deadlock detection mechanism.

<!-- end list -->

  - *' Send back result to the front-end and making sure Atomic
    operations are handled properly:*'

<!-- end list -->

  -
    Once the Cache hierarchy satisfies a request it calls Sequencer's
    **''readCallback**'' or *writeCallback*''' method depending upon the
    type of the request. Note that the time taken to service the request
    is automatically accounted for through event scheduling as the
    ***readCallback*** or '**'writeCallback**' are called only after
    number of cycles required to satisfy the request has been accounted
    for. In these two methods the corresponding record of the request
    from the '' m_readRequestTable'' or *m_writeRequestTable* is
    removed. Also if the request was found to be part of a Atomic
    operations (e.g. RMW or LL/SC), then appropriate actions are taken
    to make sure that semantics of atomic operations are respected.
    After that a method called ***hitCallback*** is called. In this
    method, some statics is collected by calling functions on the Ruby's
    profiler. For write request, the data is actually updated in this
    function (and not while simulating the request through the cache
    hierarchy and coherence protocol). Finally ***ruby_hit_callback***
    is called ultimately sends back the packet to the front-end,
    signifying completion of the processing of the memory request from
    Ruby's side.

## CacheMemory and Cache Replacement Polices

This module can model any **Set-associative Cache structure** with a
given associativity and size. Each instantiation of the following module
models a **single bank** of a cache. Thus different types of caches in
system (e.g. L1 Instruction, L1 Data , L2 etc) and every banks of a
cache needs to have separate instantiation of this module. This module
can also model Fully associative cache when the associativity is set to
1. In Ruby memory system, this module is primarily expected to be
accessed by the SLICC generated codes of the given Coherence protocol
being modeled.

### **Basic Operation**

This module models the set-associative structure as a two dimensional
(2D) array. Each row of the 2D array represents a set of in the
set-associative cache structure, while columns represents ways. The
number of columns is equal to the given associativity (parameter), while
the number of rows is decided depending on the desired size of the
structure (parameter), associativity (parameter) and the size of the
cache line (parameter). This module exposes six important
functionalities which Coherence Protocols uses to manage the caches.

1.  It allows to query if a given cache line address is present in the
    set-associative structure being modeled through a function named
    ***isTagPresent***. This function returns *true*, iff the given
    cache line address is present in it.
2.  It allows a lookup operation which returns the cache entry for a
    given cache line address (if present), through a function named
    ***lookup***. It returns NULL if the blocks with given address is
    not present in the set-associative cache structure.
3.  It allows to allocate a new cache entry in the set-associative
    structure through a function named ***allocate***.
4.  It allows to deallocate a cache entry of a given cache line address
    through a function named ***deallocate***.
5.  It can be queried to find out whether to allocate an entry with
    given cache line address would require replacement of another entry
    in the designated set (derived from the cache line address) or not.
    This functionality is provided through ***cacheAvail*** function,
    which for a given cache line address, returns True, if NO
    replacement of another entry the same set as the given address is
    required to make space for a new entry with the given address.
6.  The function ***cacheProbe*** is used to find out cache line address
    of a victim line, in case placing a new entry would require
    victimizing another cache blocks in the same set. This function
    returns the cache line address of the victim line given the address
    of the address of the new cache line that would have to be
    allocated.

### **Parameters**

There are four important parameters for this class.

1.  ***size*** is the parameter that provides the size of the
    set-associative structure being modeled in units of bytes.
2.  ***assoc*** specifies the set-associativity of the structure.
3.  ***replacement_policy**'' is the name of the replacement policy
    that would be used to select victim cache line when there is
    conflict in a given set. Currently, only two possible choices are
    available (***PSEUDO_LRU**'' and ***LRU***).
4.  Finally, ***start_index_bit*** parameter specifies the bit
    position in the address from where indexing into the cache should
    start. This is a tricky parameter and if not set properly would end
    up using only portion of the cache capacity. Thus how this value
    should be specified is explained through couple of examples. Let us
    assume the cache line size if 64 bytes and a single core machine
    with a L1 cache with only one bank and a L2 cache with 4 banks. For
    the CacheMemory module that would model the L1 cache should have
    ***start_index_bit*** set to log2(64) = 6 (this is the default
    value assuming 64 bytes cache line). This is required as addresses
    passed around in the Ruby is *full address* (i.e. equal to the
    number of bits required to access any address in the physical
    address range) and as the caches would be accessed in granularity of
    cache line size (here 64 bytes), the lower order 6 bits in the
    address would be essentially 0. So we should discard last 6 bits of
    the given address while calculating which set (index) in the set
    associative structure the given address should go to. Now let's look
    into a more complicated case of L2 cache, which has 4 banks. As
    mentioned previously, this modules models a single bank of a
    set-associative cache. Thus there will be four instantiation of the
    CacheMemory class to model the whole L2 cache. Assuming which cache
    bank a request goes to is statically decided by the low oder log2(4)
    = 2 bits of the *cache line address*, the value of the bits in the
    address at the position *6* and *7* would be same for all accesses
    coming to a given bank (i.e. a instance of CacheMemory here). Thus
    indexing within the set associative structure (CacheMemory instance)
    modeling a given bank should use address bits 8 and higher for
    finding which set a cache block should go to. Thus
    ***start_index_bit*** parameter should be set to 8 for the banks
    of L2 in this example. If *erroneously* if this is set 6, only a
    fourth of desired L2 capacity would be utilized \!\!\!

### **More detailed description of operation**

As mentioned previously, the set-associative structure is modeled as a
2D array in the CacheMemory class. The variable ***m_cache*** is this
all important 2D array containing the set-associative structure. Each
element of this 2D array is derived from type ***AbstractCacheEntry***
class. Beside the minimal required functionality and contents of each
cache entry, it can be extended inside the Coherence protocol files.
This allows CacheMemory to be generic enough to hold any type of cache
entry as desired by a given Coherence protocol as long as it derives
from ***AbstractCacheEntry*** interface. The ***m_cache*** 2D array has
number of rows equal to the number of sets in the set-associative
structure being modeled while the number of columns is equal to
associativity.

As should happen in any set-associative structure, which set (row) a
cache entry should reside is decided by part of the cache block address
used for indexing. The function ***addressToCacheSet*** calculates this
index given an address. The *way* in which a cache entry reside in its
designated set (row) is noted in the a hash_map object named
***m_tag_index***. So to access an cache entry in the set-associative
structure, first the set number where the cache block should reside is
calculated and then ***m_tag_index*** is looked-up to find out the way
in which the required cache block resides. If an cache entry holds
invalid entry or its empty then its set to *NULL* or its permission is
set to *NotPresent*.

One important aspect of the Ruby's caches are the segregation of the
set-associative structure for the cache and its replacement policy. This
allows modular design where structure of the cache is independent of the
replacement policy in the cache. When a victim needs to be selected to
make space for a new cache block (by calling ***cacheProbe*** function),
***getVictim*** function of the class implementing replacement policy is
called for the given set. ***getVictim*** returns the way number of the
victim. The replacement policy is updated about accesses by calling
***touch*** function of the replacement policy, which allows it to
update the access recency. Currently there are two replacement policies
are supported -- LRU and PseudoLRU. LRU policy has a straight forward
implementation where it keeps track of the absolute time when each way
within each set is accessed last time and it always victimizes the entry
which was last accessed furthest back in time. PseudoLRU implements a
binary-tree based Non-Recently-Used policy. It arranges the ways in each
set in an implicit ***binary tree*** like structure. Every node of the
binary tree encodes the information which of its two subtrees was
accessed more recently. During victim selection process, it starts from
the root of the tree and traverse down such that it chooses the subtree
which was touched *less* recently. Traversal continues until it reaches
a leaf node. It then returns the id of the leaf node reached.

### **Related files**

  - **src/mem/ruby/system**
      - **CacheMemory.cc**: contains CacheMemory class which models a
        cache bank
      - **CacheMemory.hh**: Interface for the CacheMemory class
      - **Cache.py**: Python configuration file for CacheMemory class
      - **AbstractReplacementPolicy.hh**: Generic interface for
        Replacement policies for the Cache
      - **LRUPolicy.hh**: contains LRU replacement policy
      - **PseudoLRUPolicy.hh**: contains Pseudo-LRU replacement policy
  - **src/mem/ruby/slicc_interface**
      - **AbstarctCacheEntry.hh**: contains the interface of Cache Entry

## DMASequencer

This module implements handling for DMA transfers. It is derived from
the RubyPort class. There can be a number of DMA controllers that
interface with the DMASequencer. The DMA sequencer has a
protocol-independent interface and implementation. The DMA controllers
are described with SLICC and are protocol-specific.

**TODO: Fix documentation to reflect latest changes in the
implementation.**

*Note:*

1.  <span style="color:#006400">*At any time there can be only 1 request
    active in the DMASequencer.*</span>
2.  <span style="color:#006400">*Only ordinary load and store requests
    are handled. No other request types such as Ifetch, RMW, LL/SC are
    handled*</span>

### Related Files

  - **src/mem/ruby/system**
      - **DMASequencer.hh**: Declares the DMASequencer class and
        structure of a DMARequest
      - **DMASequencer.cc**: Implements the methods of the DMASequencer
        class, such as request issue and callbacks.

### Configuration Parameters

Currently there are no special configuration parameters for the
DMASequencer.

### Basic Operation

A request for data transfer is split up into multiple requests each
transferring cache-block-size chunks of data. A request is active as
long as all the smaller transfers are not completed. During this time,
the DMASequencer is in a busy state and cannot accepts any new transfer
requests.

DMA requests are made through the **makeRequest** method. If the
sequencer is not busy and the request is of the correct type (LD/ST), it
is accepted. A sequence of requests for smaller data chunks is then
issued. The **issueNext** method issues each of the smaller requests. A
data/acknowledgment callback signals completion of the last transfer and
triggers the next call to **issueNext** as long as all of the original
data transfer is not complete. There is no separate event scheduler
within the DMASequencer.

## Memory Controller

This module simulates a basic DDR-style memory controller. It models a
**single channel**, connected to any number of DIMMs with any number of
ranks of DRAMs each. The following picture shows an overview of the
memory organization and connections to the memory controller. General
information about memory controllers can be found
[here](http://en.wikipedia.org/wiki/Memory_controller).

![mc_overview.jpg](mc_overview.jpg "mc_overview.jpg")

'' Note: ''

1.  <span style="color:#006400">*The product of the memory bus cycle
    multiplier, memory controller latency, and clock cycle
    time(=1/processor frequency) gives a first-order approximation of
    the latency of memory requests in time. The Memory Controller module
    refines this further by considering bank & bus contention, queueing
    effects of finite queues, and refreshes.* </span>
2.  <span style="color:#006400">*Data sheet values for some components
    of the memory latency are specified in time (nanoseconds), whereas
    the Memory Controller module expects all delay configuration
    parameters in cycles. The parameters should be set appropriately
    taking into account the processor and bus frequencies.*</span>
3.  <span style="color:#006400">*The current implementation does not
    consider pin-bandwidth contention. Infinite bandwidth is
    assumed.*</span>
4.  <span style="color:#006400">*Only closed bank policy is currently
    implemented; that is, each bank is automatically closed after a
    single read or write.*</span>
5.  <span style="color:#006400">*The current implementation handles only
    a single channel. If you want multiple address/data channels, you
    need to instantiate multiple copies of this module.* </span>
6.  <span style="color:#006400">*This is the only controller that is NOT
    specified in SLICC, but in C++.*</span>

**Documentation source**: Most (but not all) of the writeup in this
section is taken verbatim from documentation in the gem5 source files,
rubyconfig.defaults file of GEMS, and a ppt created by Andy Phelps on
Jan 18, 2008.

### Related Files

  - **src/mem/ruby/system**
      - **MemoryControl.hh**: This file declares the Memory Controller
        class.
      - **MemoryControl.cc**: This file implements all the operations of
        the memory controller. This includes processing of input
        packets, address decoding and bank selection, request scheduling
        and contention handling, handling refresh, returning completed
        requests to the directory controller.
      - **MemoryControl.py**: Configuration parameters

### Configuration Parameters

  - **dimms_per_channel**: Currently the only thing that matters is
    the number of ranks per channel, i.e. the product of this parameter
    and **ranks_per_dimm**. But if and when this is expanded to do
    FB-DIMMs, the distinction between the two will matter.

<!-- end list -->

  - *Address Mapping*: This is controlled by configuration parameters
    **banks_per_rank**, **bank_bit_0**, **ranks_per_dimm**,
    **rank_bit_0**, **dimms_per_channel**, **dimm_bit_0**. You
    could choose to have the bank bits, rank bits, and DIMM bits in any
    order. For the default values, we assume this format for addresses:
      - Offset within line: \[5:0\]
      - Memory controller \#: \[7:6\]
      - Bank: \[10:8\]
      - Rank: \[11\]
      - DIMM: \[12\]
      - Row addr / Col addr: \[top:13\]

If you get these bits wrong, then some banks won't see any requests; you
need to check for this in the .stats output.

  - **mem_bus_cycle_multiplier**: Basic cycle time of the memory
    controller. This defines the period which is used as the memory
    channel clock period, the address bus bit time, and the memory
    controller cycle time. Assuming a 200 MHz memory channel (DDR-400,
    which has 400 bits/sec data), and a 2 GHz processor clock,
    mem_bus_cycle_multiplier=10.

<!-- end list -->

  - **mem_ctl_latency**: Latency to returning read request or
    writeback acknowledgement. Measured in memory address cycles. This
    equals tRCD + CL + AL + (four bit times) + (round trip on channel) +
    (memory control internal delays). It's going to be an approximation,
    so pick what you like. *Note: The fact that latency is a constant,
    and does not depend on two low-order address bits, implies that our
    memory controller either: (a) tells the DRAM to read the critical
    word first, and sends the critical word first back to the CPU, or
    (b) waits until it has seen all four bit times on the data wires
    before sending anything back. Either is plausible. If (a), remove
    the "four bit times" term from the calculation above.*

<!-- end list -->

  - **rank_rank_delay**: This is how many memory address cycles to
    delay between reads to different ranks of DRAMs to allow for clock
    skew.

<!-- end list -->

  - **read_write_delay**: This is how many memory address cycles to
    delay between a read and a write. This is based on two things: (1)
    the data bus is used one cycle earlier in the operation; (2) a
    round-trip wire delay from the controller to the DIMM that did the
    reading. Usually this is set to 2.

<!-- end list -->

  - **basic_bus_busy_time**: Basic address and data bus occupancy. If
    you are assuming a 16-byte-wide data bus (pairs of DIMMs
    side-by-side), then the data bus occupancy matches the address bus
    occupancy at 2 cycles. But if the channel is only 8 bytes wide, you
    need to increase this bus occupancy time to 4 cycles.

<!-- end list -->

  - **mem_random_arbitrate**: By default, the memory controller uses
    round-robin to arbitrate between ready bank queues for use of the
    address bus. If you wish to add randomness to the system, set this
    parameter to one instead, and it will restart the round-robin
    pointer at a random bank number each cycle. If you want additional
    nondeterminism, set the parameter to some integer n \>= 2, and it
    will in addition add a n% chance each cycle that a ready bank will
    be delayed an additional cycle. Note that if you are in
    mem_fixed_delay mode (see below), mem_random_arbitrate=1 will
    have no effect, but mem_random_arbitrate=2 or more will.

<!-- end list -->

  - **mem_fixed_delay**: If this is nonzero, it will disable the
    memory controller and instead give every request a fixed latency.
    The nonzero value specified here is measured in memory cycles and is
    just added to MEM_CTL_LATENCY. It will also show up in the stats
    file as a contributor to memory delays stalled at head of bank
    queue.

<!-- end list -->

  - **tFAW**: This is an obscure DRAM parameter that says that no more
    than four activate requests can happen within a window of a certain
    size. For most configurations this does not come into play, or has
    very little effect, but it could be used to throttle the power
    consumption of the DRAM. In this implementation (unlike in a DRAM
    data sheet) TFAW is measured in memory bus cycles; i.e. if TFAW = 16
    then no more than four activates may happen within any 16 cycle
    window. Refreshes are included in the activates.

<!-- end list -->

  - **refresh_period**: This is the number of memory cycles between
    refresh of row x in bank n and refresh of row x+1 in bank n. For
    DDR-400, this is typically 7.8 usec for commercial systems; after
    8192 such refreshes, this will have refreshed the whole chip in 64
    msec. If we have a 5 nsec memory clock, 7800 / 5 = 1560 cycles. The
    memory controller will divide this by the total number of banks, and
    kick off a refresh to somebody every time that amount is counted
    down to zero. (There will be some rounding error there, but it
    should have minimal effect.)

<!-- end list -->

  - **Typical Settings for configuration parameters**: The default
    values are for DDR-400 assuming a 2GHz processor clock. If instead
    of DDR-400, you wanted DDR-800, the channel gets faster but the
    basic operation of the DRAM core is unchanged. Busy times appear to
    double just because they are measured in smaller clock cycles. The
    performance advantage comes because the bus busy times don't
    actually quite double. You would use something like these values:

<!-- end list -->

  -

      -
        mem_bus_cycle_multiplier: 5
        bank_busy_time: 22
        rank_rank_delay: 2
        read_write_delay: 3
        basic_bus_busy_time: 3
        mem_ctl_latency: 20
        refresh_period: 3120

### Basic Operation

![mc_data_struct.jpg](mc_data_struct.jpg "mc_data_struct.jpg")

  - **Data Structures**

Requests are enqueued into a single input queue. Responses are dequeued
from a single response queue. There is a single bank queue for each DRAM
bank (the total number of banks is the number of DIMMs per channel x
number of ranks per DIMM x number of banks per rank). Each bank also has
a busy counter. tFAW shift registers are maintained per rank.

  - **Timing**

![mc_addr_command_timing.jpg](mc_addr_command_timing.jpg
"mc_addr_command_timing.jpg")
![mc_addr_command_timing_back_to_back.jpg](mc_addr_command_timing_back_to_back.jpg
"mc_addr_command_timing_back_to_back.jpg")

The “Act” (Activate) and “Rd” (Read) commands (or activate and write)
always come as a pair, because we are modeling posted-CAS mode. (In
non-posted-CAS, the read or write command would be scheduled separately
later.) We do not explicitly model the separate commands; we simply say
that the address bus occupancy is 2 cycles.

Since the data bus is also occupied for 2 cycles at a fixed offset in
time as shown above, we do not need to explicitly model it; memory
channel occupancy is still 2 cycles.

For back-to-back requests the data for the 2nd request could be delayed
due to the following reasons:

  -   - Read happens from a different rank
      - Read is followed by a write
      - 2nd request has a busy bank
      - Basic request time \> 2 (e.g. needs 8 data phits)

<!-- end list -->

  - **Scheduling and Bank Contention**

The **wakeup** function, and in turn, the **executeCycle** function is
tiggered once every memory clock cycle.

Each memory request is placed in a queue associated with a specific
memory bank. This queue is of finite size; if the queue is full the
request will back up in an (infinite) common queue and will effectively
throttle the whole system. This sort of behavior is intended to be
closer to real system behavior than if we had an infinite queue on each
bank. If you want the latter, just make the bank queues unreasonably
large.

The head item on a bank queue is issued when all of the following are
true:

1.  The bank is available
2.  The address path to the DIMM is available
3.  The data path to or from the DIMM is available

Note that we are not concerned about fixed offsets in time. The bank
will not be used at the same moment as the address path, but since there
is no queue in the DIMM or the DRAM it will be used at a constant number
of cycles later, so it is treated as if it is used at the same time.

We are assuming "posted CAS"; that is, we send the READ or WRITE
immediately after the ACTIVATE. This makes scheduling the address bus
trivial; we always schedule a fixed set of cycles. For DDR-400, this is
a set of two cycles; for some configurations such as DDR-800 the
parameter tRRD forces this to be set to three cycles.

We assume a four-bit-time transfer on the data wires. This is the
minimum burst length for DDR-2. This would correspond to (for example) a
memory where each DIMM is 72 bits wide and DIMMs are ganged in pairs to
deliver 64 bytes at a shot.This gives us the same occupancy on the data
wires as on the address wires (for the two-address-cycle case).

The only non-trivial scheduling problem is the data wires. A write will
use the wires earlier in the operation than a read will; typically one
cycle earlier as seen at the DRAM, but earlier by a worst-case
round-trip wire delay when seen at the memory controller. So, while
reads from one rank can be scheduled back-to-back every two cycles, and
writes (to any rank) scheduled every two cycles, when a read is followed
by a write we need to insert a bubble. Furthermore, consecutive reads
from two different ranks may need to insert a bubble due to skew between
when one DRAM stops driving the wires and when the other one starts.
(These bubbles are parameters.)

This means that when some number of reads and writes are at the heads of
their queues, reads could starve writes, and/or reads to the same rank
could starve out other requests, since the others would never see the
data bus ready. For this reason, we have implemented an anti-starvation
feature. A group of requests is marked "old", and a counter is
incremented each cycle as long as any request from that batch has not
issued. If the counter reaches twice the bank busy time, we hold off any
newer requests until all of the "old" requests have issued.
