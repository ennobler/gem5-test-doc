---
title: "Modular coherence"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---


## Introduction

We need a way to add/extend/replace cache coherence protocols in a
modular fashion. The original cache module had coherence factored out
into a separate object, but the interface was totally inadequate for
doing anything interesting. The redesigned memory system from 2.0
integrated coherence back in to the cache module as a temporary step to
eliminate the useless bloat caused by the original separation. It's now
time to go back and re-introduce this modularity, but to do it right
this time.

We are hugely inspired by SLICC, the coherence protocol description
language/compiler that's part of the [GEMS](http://www.cs.wisc.edu/gems)
simulator from Wisconsin. Following their example, we plan on developing
a similar domain-specific language (DSL) and compiler for coherence
protocols. SLICC is fairly closely tied to the GEMS framework, so it's
not possible to adopt it directly for M5. In addition, there are a
number of aspects of SLICC that we would like to change, partly based on
personal bias and partly based on areas where we think we can improve on
their design. To try and come up with the best solution we can, we will
approach this design as a **clean sheet**, and not automatically adopt
any part of SLICC by default. At the same time, SLICC is a terrific
working example, so many of our design decisions will probably end up
being "let's do this like SLICC, except...". Correspondingly, most of
the initial design requirements and preferences listed below are "things
we should do differently than SLICC", with the implicit assumption that
most of the issues not listed below will be handled similarly.

## Design Requirements

### Protocol inheritance

This is Brad's \#1 request for improving on SLICC. Related protocols
should be able to share common elements. New protocols that are minor
modifications or extensions of existing protocols should be describable
by inheriting from the existing protocol and describing only the
differences.

### Implicit hit path

The coherence protocol should not be involved on cache hits. Requiring a
call into the protocol object on hits will drastically reduce our
opportunities for performance optimization of this critical path in the
simulator. As is often the case, simulation reflects reality, and no
sane coherence protocol would intrude on the hit path of a real
implementation either. The Tag object is currently consulted on hits (to
enable replacement algorithm bookkeeping), so if someone desperately
needs to collect information on hits they can implement a new Tag object
to do that.

The implicit cache hit path will use common, protocol-independent access
permission bits to determine if a block is readable and/or writable and
determine whether an access is a hit accordingly. The hit path will set
a protocol-independent dirty bit on writes. This bit can be incorporated
into the protocol state, e.g., to differentiate clean exclusive vs.
modified states.

### Support internal cache timing

Though we currently don't model any internal cache timing, we probably
should eventually, and we want to make sure the language is up to the
task. For example, having a `getState()` function that we expect to
return immediately means that we can't explicitly model the tag array
access latency. It might be better to assume that the tag lookup happens
explicitly when a message is first processed and then assume that the
controller (and thus the coherence code) is handed both the message and
the tag state right from the start.

### Automatic and Powerful Protocol Debugging Support

Protocol designers will likely spend the majority of their time
debugging protocols. The protocol support should make debugging as easy
as possible. In particular, concise and informative auto generated
DPRINTFs are a high-priority requirement. The key is outputting debug
traces with enough information to completely follow all state
transitions while allowing a single (\< 1 GB) trace to span millions of
cycles. Furthermore, the protocols must detect invalid transition errors
as soon as they occur.

### Virtual Channels

To encounter and solve protocol deadlock issues the coherence protocols
must support virtual channels. Therefore the network connecting
different memory components must support virtual channels as well.

## Design Preferences

### Balance Explicit State Machines vs. Attributes

Ideally, we would like to find a balance of using an explicit
state-machine approach versus attributes. Currently, SLICC and M5
highlight the extremes of each approach. The current M5 coherence
protocol uses memory command attributes to allow clean code; see the
`MemCmd` class in src/mem/packet.{hh,cc}. For example, all requests
requiring an exclusive copy of a block (Write, ReadEx, Upgrade,
StoreCond, Swap) have the NeedsExclusive attribute set, so the handler
simply uses "if (req-\>needsExclusive())" to determine whether to check
for exclusivity and to request an exclusive copy if needed. While
attributes work well for simple and abstract protocols, it is likely an
attribute-only approach is insufficient to describe a complex and more
realistic protocol.

In contrast, SLICC uses different explicit events and states to signify
all easily visible protocol differences. The advantage of this approach
is all differences in events and states can be easily tracked in the
protocol. Therefore many errors in complicated protocols can be found
with easily traceable invalid transition errors, rather than hard to
isolate incorrect attribute handling errors. The disadvantage is the
language specification files are quite long and verbose, and are
susceptible to cut-and-paste errors. SLICC does support attributes as
well, but they are only utilized within actions.

Overall, the best solution may be to support for both models, using an
explicit state-machine model for the general coherence controller
structure, but providing explicit support for attributes as well so that
minor variations along common paths can be implemented via small
if-conditions on a common state-machine transition.

### Encode Protocol-Independent State in Protocol-Specific State

Ideally, we would like the encode the cache state to combine both the
protocol-independent state (access attributes, etc.) and
protocol-specific state. For example, M5 currently supports
CacheBlkStatusBits in src/mem/cache/blk.hh which for the most part are
protocol independent (except the dirty bit). The block state model needs
to be expanded to allow protocol-specific state, but we should consider
continuing to use encoding tricks to avoid redundantly storing both the
block access bits and the full coherence state separately. This encoding
means that setting the protocol-level cache state implicitly changes the
protocol-independent access permissions, and vice versa (e.g., setting
the dirty bit implicitly modifies the protocol-level state
appropriately). Therefore, with protocol-independent access permissions
encoded in the state, the cache access path can determine hits and
misses without invoking the cache controller logic and without requiring
separate logic to set permissions.

In contrast, SLICC doesn't directly associate semantics with requests or
states, and you end up with code like
this:

`     // Set permission`
`     if ((state == State:I) || (state == State:MI_A) || (state == State:II_A)) {`
`       changePermission(addr, AccessPermission:Invalid);`
`     } else if (state == State:S || state == State:O) {`
`       changePermission(addr, AccessPermission:Read_Only);`
`     } else if (state == State:M) {`
`       changePermission(addr, AccessPermission:Read_Write);`
`     } else {`
`       changePermission(addr, AccessPermission:Busy);`
`     }`

### Support Hardware Prefetching without Requiring Specific Protocol Support

Another advantage with encoding protocol-independent state within the
cache state, is hardware prefetching can possibly be supported in all
designs without requiring specific protocol support. We should be
careful to ensure hardware prefetches don't impede demand requests.
Hopefully, we can enforce this policy without impacting the protocol
specific logic. In contrast, GEMS requires each protocol to explicitly
handle hardware prefetches separately.

### No new imperative language

Let's not invent a new language for imperative computation. I admit that
the "treat C++ code blocks as strings" approach of the ISA description
language is a bit cheesy, but for the most part it works. The main
drawback is that you can't parse the code to analyze its behavior
(though the ISA parser extracts a lot of information in an imperfect way
using regexps; see [Code parsing](Code_parsing "wikilink")), but I think
we'll have even less need of that here than the ISA parser does.

Even if we do feel the need to fully parse imperative computations, or
to restrict what people can do in them, I'd rather see us do a subset of
C++ than invent something new. (That might actually be useful if we can
reuse the C++-subset parser for the ISA description language.)
Unfortunately, this is easier said than done; one big challenge is
getting the context set up right so the parser can figure out which
symbols are types, preprocessor macros are expanded, etc.

Note that some of the SLICC design notes talk about wanting to make the
language synthesizable to hardware, which is a reasonable motivation for
them to have a limited imperative language. We don't have that goal, so
there's much less reason to be restrictive.

## Design Challenges

### Interfacing generic with protocol-specific bits

Though it's reasonable to specialize caches and interconnects based on
the coherence protocol, we don't want to recompile every object that
connects to the memory system (CPUs, devices, etc.) for the protocol
too. Yet these devices do need to send out and respond to basic memory
requests (read, write, instruction fetch, etc.). One possible solution
is to leverage inheritance, and have a base coherence protocol that
defines the set of messages which all CPUs and devices can use. This
base protocol will probably be abstract (if only by having all the
required functions call `fatal()`). Then generic memory system clients
can depend only on the base protocol, and as long as the derived
protocols extend it in a compatible fashion, everything will work.

Note that this solution will constrain how we implement inheritance: we
can't do it entirely in the DSL and have it spit out flattened C++, as
we'll need this base protocol to be separate in C++ so that the
compilation of CPU and device models can be independent of the specific
protocol.

### System Configuration Issues

(Steve I'm not sure what public information you want to discuss here.)

### Fixed vs. flexible aspects of cache organization

At some level, all the coherence protocols have to fit into a common
framework; we don't want each protocol having to define from scratch
what it means to be a cache. Figuring out where the line is between the
fixed components of the common framework and the pieces that need to be
felxibly defined by the coherence protocol is a challenge, particularly
in these areas:

#### MSHR Model

From brief discussions with Brad, it sounds like caches in SLICC
protocols have limited ability to buffer requests to busy blocks.
Multiple read requests may get buffered/merged by building a bitmask of
requesters, but otherwise requests are either NACKed or recycled to the
top of the input queue. In contrast, M5 has an MSHR model in which each
MSHR has a finite number of "targets", each of which can buffer a
request to a busy block. This model encompasses both the handling of
overlapping requests from an out-of-order CPU in an L1 cache, handling
requests from multiple CPUs/L1s in an L2 cache, and so on. Depending on
the system configuration, it is expected that these MSHRs can be sized
to handle the worst-case number of outstanding accesses from upstream
CPUs/caches.

The MSHR model seems to provide a nice consistent framework for handling
multiple requests; once a response arrives, the buffered target requests
are replayed almost as if they had just been received, that is, in
effect the same message is re-processed against the new post-response
cache state as if that state had been there when the request was first
received. While this model seems like it should work for a variety of
protocols, do we want to hard-code it as part of the framework? If not,
how do we expose this kind of structure in the protocol description
language in such a way that its behavior can be modified?

One possible shortcoming of the MSHR model is that it would need to be
enhanced to take advantage of networks & protocols that support
broadcast or multicast responses.

#### Cache Controller to Cache Object Ratio

There are a few cases where SLICC uses a single controller to manage
multiple caches, including separate Harvard-style L1 I and L1 D caches,
as well as exclusive L1/L2 caches. It's unclear if this is really
desirable or is an artifact of SLICC not easily supporting a more
decoupled approach. Conceptually it seems both more straightforward and
more realistic to restrict a single controller to managing a single
cache, but we don't want to make our model less flexible than SLICC
unless we're really sure that's the right thing to do.

## Basic Design

There's not much here yet, unfortunately.

### Simulator implementation

The coherence protocol implementation (which will eventually be
auto-generated from the DSL description) should consist of one or more
C++ objects. The cache and interconnect models will be modified to take
a coherence protocol object as a template parameter. The protocol
object(s) will define types used to define the allowable commands,
define the protocol-specific state included in Packet and CacheBlk
classes, etc.

  - Should the C++ protocol object(s) form an inheritance hierarchy that
    mirrors the inheritance relationship of the protocols in the DSL?

### Domain-specific language

We should write the parser in PLY and the compiler in Python because we
know them, and to avoid introducing any new tool dependencies.

## Development Plan

1.  Separate the current coherence protocol from the cache module into a
    separate policy object (or objects). This will give us an initial
    interface definition and an idea of what kind of code the
    description language will need to generate.
2.  Implement a few different protocols using this interface, expanding
    and modifying it as needed. Ideally these protocols will be
    significantly different from the base protocol, e.g., directory
    based, point-to-point, etc. This will help us flesh out the
    interface, and get started on implementing features that we'll need
    for more advanced protocols (such as virtual channels) that will be
    reflected in the description language but are largely orthogonal to
    it. Some candidates would be Geoff Blake's LogTM implementation or a
    manual translation of an existing SLICC protocol from the GEMS
    distribution. A side benefit will be that those of us who need a
    particular protocol can implement it at this stage and not have the
    description language be on their critical path. There is the down
    side that there will be some extra effort to reimplement these
    protocols in the description language, but if our language is as
    concise as it should be then this effort should be minimal.
3.  Design and implement the description language and associated
    compiler. This work can overlap with the previous step somewhat, but
    we should avoid getting too far ahead of ourselves in case the
    previous step uncovers some features that require significant
    modifications to the language structure.

\--[Brad](User:bbeckman "wikilink") 13:41, 18 July 2008 (EDT)
