---
title: "GEMS transition to gem5"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

This page covers the relevant changes that have been made to SLICC since
the last and final GEMS release (2.1). It can be used as a guide to port
SLICC protocols that formerly worked in GEMS to gem5.

# Reasoning

Most changes to the SLICC language were made in order to support new
features of gem5, including the parameterized object model (SimObject),
atomic instruction support, and others. All syntactic changes are
relatively simple and mechanical.

# High Level Changes

The major conceptual change that occurred in the gem5 transition relates
to the classes that SLICC generates. Previously, SLICC would generate a
C++ class for each machine as well as a Chip class that served as a
container for objects that roughly belonged "on-chip". The Chip class
was responsible for defining, constructing, and initializing each
machine (e.g., L1Cache_Controller) and related objects.

In the new version of SLICC for gem5, **the Chip class no longer
exists**. The only C++ object that SLICC generates is the machine (and
associated helper structs/classes like States). Notably, this means that
non-generated code is now responsible for orchestrating the construction
and initialization of machines.

In addition to removing the Chip class, the SLICC machines are also more
modular now. Each machine can define parameters that are set at
configuration time, allowing for the construction of heterogeneous
memory system configurations. Also because of the enhanced
configuration, any object used by a SLICC machine that can itself be
configured (i.e., is a SimObject) must be passed as a parameter rather
than created by the machine so that the object can be initialized
properly by the configuration system. You'll notice this, for example,
in machines that accept a Sequencer object as a parameter. Parameters to
a protocol are set in Python at configuration time (more details below).

# Details

## Protocol Parameters

SLICC machine definitions have been modified to fit better into the gem5
object model. Parameters are defined between the machine definition and
body, like so:

`machine(Name, desc) `
`  : `<parameter list>
`{ body }`

Parameters can be any type SLICC knows about, whether it be a primitive
or external_type. Typically, parameters will include latencies,
SimObjects used by the machine (e.g., Sequencer, CacheMemory), and
possibly other protocol-specific parameters.

Parameters are declared with a type, and can optionally take a default
value:

`machine(L1Cache, "MSI Directory L1 Cache CMP")`
`  : Sequencer * sequencer,`
`    CacheMemory * L1IcacheMemory,`
`    CacheMemory * L1DcacheMemory,`
`    int l2_select_num_bits,`
`    int l1_request_latency = 2,`
`    int l1_response_latency = 2,`
`    int to_l2_latency = 1`
` {  ...  }`

Each protocol has an associated Python file in configs/ruby/ that is
responsible for passing parameters to controllers. For example, below is
a snippit from configs/ruby/MESI_CMP_directory.py that corresponds to
the machine definition above:

`l1_cntrl = L1Cache_Controller(version = i,`
`                              cntrl_id = cntrl_count,`
`                              sequencer = cpu_seq,`
`                              L1IcacheMemory = l1i_cache,`
`                              L1DcacheMemory = l1d_cache,`
`                              l2_select_num_bits = l2_bits)`

Any parameter that is a SimObject (e.g., l1d_cache) is converted to a
pointer type before being passed to SLICC.

## Latencies

Previously, latencies were in an enqueue function as a string
corresponding to a parameter in the Ruby configuration file. Now,
latencies are integer parameters and the latency= portion of enqueue
takes an integer rather than a
string.

`enqueue(requestIntraChipL1Network_out, RequestMsg, latency=l1_request_latency) { ... }`

## Atomic Instruction Support

Because Ruby/gem5 must now correctly account for data, SLICC protocols
have support for handling atomic (Read-Modify-Write) accesses. Most of
that functionality is hidden in the generated code, but SLICC protocol
writers are responsible for identifying which field in a message
contains the address to block on. This is done using the block_on
parameter to the peek function used in an in port. For example, the
line

`peek(responseIntraChipNetwork_in, ResponseMsg, block_on="Address") { ... }`

tells the generated controller that when an atomic access occurs, to use
the field ResponseMsg:Address as the address for comparison.

## State Declarations

States are now declared using a special state_declaration function
rather than a generic enumeration function as was done previously.
Additionally, AccessPermissions are now directly tied to states. For
example:

`state_declaration(State, desc="...", default="L1Cache_State_I") {`
`   NP, AccessPermission:Invalid, desc="Not present in either cache";`
`   I, AccessPermission:Invalid, desc="a L1 cache entry Idle";                                `
`   S, AccessPermission:Read_Only, desc="a L1 cache entry Shared";                                     `
`   E, AccessPermission:Read_Only, desc="a L1 cache entry Exclusive";                                 `
`   M, AccessPermission:Read_Write, desc="a L1 cache entry Modified", format="!b"; `
`}`

The format= option is also new to SLICC, and does \<TODO: what does this
do?\>.

## Entry Types and Casting

Cache entry definitions are now properly abstracted. Rather than relying
on the convention that all protocols name fields of their cache entries
the same, the common parts of a cache entry needed by CacheMemory to
function (e.g., Address or locked) are declared in the non-generated
AbstractCacheEntry class. Entries defined in a SLICC machine must
inherit from that abstract class in order to be usable by CacheMemory.

As a consequence of inheriting from an abstract object inside of SLICC,
we've also added a mechanism to cache to the correct entry type using
static_cast.

Example

`structure(Entry, desc="...", interface="AbstractCacheEntry") {`
`  State CacheState,  desc="cache state"; `
`  ...`
`}`

An entry can later be
used:

`  Entry getL1DCacheEntry(Address addr), return_by_pointer="yes" {`
`    Entry L1Dcache_entry := static_cast(Entry, "pointer", L1DcacheMemory[addr]);`
`    return L1Dcache_entry;`
`  }`
