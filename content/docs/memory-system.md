---
title: "Memory system"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

gem5's memory system was designed to enable:

1.  Modularity and compartmentalisation through standard interfaces.
2.  Suitable interfaces for loosely-timed, approximately-timed and
    untimed transaction-level modelling.
3.  Flexibility to allow other memory interconnects besides a crossbar.
4.  A comprehensive set of building blocks, ranging from caches,
    crossbars, to full-blown DRAM controllers.

## Ports system

### MemObjects

All objects within a memory system inherit from `MemObject`. This class
adds the pure virtual functions `getMasterPort(const std::string &name)`
and `getSlavePort(const std::string &name)` which returns a port
corresponding to the given name. This interface is used to connect
memory objects together.

### Ports

Ports are used to interface memory objects to each other. They will
always come in pairs and we refer to the other port object as the peer.
A master port always connects to a slave port, with the master
initiating requests, and the slave providing responses. Every memory
object has to have at least one port to be useful.

There are two groups of functions in the port object. The `send*`
functions are called on the port by the object that owns that port. For
example to send a request packet in the memory system a CPU would call
`myPort->sendTimingReq(pkt)` to send a packet. Each send function has a
corresponding recv function that is called on the ports peer. So the
implementation of the `sendTimingReq()` call above would simply be
`peer->recvTimingReq(pkt)`. Using this method we only have one virtual
function call penalty but keep generic ports that can connect together
any memory system objects.

### Connections

In Python, Ports are first-class attributes of simulation objects, much
like Params. Two objects can specify that their ports should be
connected using the assignment operator. Unlike a normal variable or
parameter assignment, port connections are symmetric: `A.port1 =
B.port2` has the same meaning as `B.port2 = A.port1`.

Objects such as busses that have a potentially unlimited number of ports
use "vector ports". An assignment to a vector port appends the peer to a
list of connections rather than overwriting a previous connection.

### Port proxies

There are three types of port proxies that wrap the port interface and
are used for initialisation and introspection.

  - `PortProxy` provides easy to use methods for writing and reading
    physical addresses. It is only meant to load data into memory and
    update constants before the simulation begins.
  - `SETranslatingPortProxy` and `FSTranslatingPortProxy` provide the
    same methods as `PortProxy`, but the addresses passed to them are
    virtual addresses, and a translation is done to get the physical
    address.

## Packets

A Packet is used to encapsulate a transfer between two objects in the
memory system (e.g., the L1 and L2 cache). This is in contrast to a
Request where a single Request travels all the way from the requester to
the ultimate destination and back, possibly being conveyed by several
different Packets along the way.

Read access to many packet fields is provided via accessor methods which
verify that the data in the field being read is valid.

A packet contains the following all of which are accessed by accessors
to be certain the data is valid:

  - The address. This is the address that will be used to route the
    packet to its target (if the destination is not explicitly set) and
    to process the packet at the target. It is typically derived from
    the request object's physical address, but may be derived from the
    virtual address in some situations (e.g., for accessing a fully
    virtual cache before address translation has been performed). It may
    not be identical to the original request address: for example, on a
    cache miss, the packet address may be the address of the block to
    fetch and not the request address.
  - The size. Again, this size may not be the same as that of the
    original request, as in the cache miss scenario.
  - A pointer to the data being manipulated.
      - Set by `dataStatic()` and `dataDynamic()`, which controls if the
        data associated with the packet is freed when the packet is.
      - Allocated if not set by one of the above methods `allocate()`
        and the data is freed when the packet is destroyed. (Always safe
        to call).
      - A pointer can be retrived by calling `getPtr()` or
        `getConstPtr()`
      - `get()` and `set()` can be used to manipulate the data in the
        packet. The get() method does a guest-to-host endian conversion
        and the set method does a host-to-guest endian conversion.
  - A list of [Packet Command
    Attributes](Packet_Command_Attributes "wikilink") associated with
    the packet
  - A `SenderState` pointer which is a virtual base opaque structure
    used to hold state associated with the packet but specific to the
    sending device (e.g., an MSHR). A pointer to this state is returned
    in the packet's response so that the sender can quickly look up the
    state needed to process it. A specific subclass would be derived
    from this to carry state specific to a particular sending device.
  - A pointer to the request.

## Requests

A request object encapsulates the original request issued by a CPU or
I/O device. The parameters of this request are persistent throughout the
transaction, so a request object's fields are intended to be written at
most once for a given request. There are a handful of constructors and
update methods that allow subsets of the object's fields to be written
at different times (or not at all). Read access to all request fields is
provided via accessor methods which verify that the data in the field
being read is valid.

The fields in the request object are typically not available to devices
in a real system, so they should normally be used only for statistics or
debugging and not as architectural values.

Request object fields include:

  - Virtual address. This field may be invalid if the request was issued
    directly on a physical address (e.g., by a DMA I/O device).
  - Physical address.
  - Data size.
  - Time the request was created.
  - The ID of the CPU/thread that caused this request. May be invalid if
    the request was not issued by a CPU (e.g., a device access or a
    cache writeback).
  - The PC that caused this request. Also may be invalid if the request
    was not issued by a CPU.

## Atomic/Timing/Functional accesses

There are three types of accesses supported by the ports.

1.  **Timing** - Timing accesses are the most detailed access. They
    reflect our best effort for realistic timing and include the
    modeling of queuing delay and resource contention. Once a timing
    request is successfully sent at some point in the future the device
    that sent the request will get a response. Timing and Atomic
    accesses can not coexist in the memory system. This is similar to
    the TLM nb_transport interface.
2.  **Atomic** - Atomic accesses are a faster than detailed access. They
    are used for fast forwarding and warming up caches and return an
    approximate time to complete the request without any resource
    contention or queuing delay. When an atomic access is sent the
    response is provided when the function returns. Atomic and timing
    accesses can not coexist in the memory system. This is similar to
    the TLM b_transport interface (without any blocking).
3.  **Functional** - Like atomic accesses functional accesses happen
    instantaneously, but unlike atomic accesses they can coexist in the
    memory system with atomic or timing accesses. Functional accesses
    are used for things such as loading binaries, examining/changing
    variables in the simulated system, and allowing a remote debugger to
    be attached to the simulator. The important note is when a
    functional access is received by a device, if it contains a queue of
    packets all the packets must be searched for requests or responses
    that the functional access is effecting and they must be updated as
    appropriate. The `Packet::checkFunctionl()` is responsible for this.

### Timing Flow control

Timing requests simulate a real memory system, so unlike functional and
atomic accesses their response is not instantaneous. Because the timing
requests are not instantaneous, flow control is needed. When a timing
packet is sent via `sendTiming()` the packet may or may not be accepted,
which is signaled by returning true or false. If false is returned the
object should not attempt to sent anymore packets until it receives a
`recvRetry()` call. At this time it should again try to call
`sendTiming()`; however the packet may again be rejected. Note: The
original packet does not need to be resent, a higher priority packet can
be sent instead.

### Response and Snoop ranges

Ranges in the memory system are handled by having all slave ports
provide an implementation for `getAddrRanges` This method returns
an`AddrRangeList` with addresses it responds to. When these ranges
change (e.g. from PCI configuration taking place) the device should call
`sendRangeChange` on its port so that the new ranges are propagated to
the entire hierarchy. This is precisely what happens during `init()`;
all memory objects call `sendRangeChange()`, and a flurry of range
updates occur until everyones ranges have been propagated to all busses
in the system.

## Tracing and traffic generation

The memory system has a number of components that facilitate detailed
tracing and traffic analysis, as well as traffic generation and trace
playback.

### Traffic analysis and memory trace capture

The `CommMonitor` provides a range of monitoring capabilities, such as
histograms for bandwidth, latency, outstanding transactions and burst
size, address heat maps, etc. It also provides a well-defined probe
interface to analyze packets as they are transferred through the memory
system. Currently, gem5 comes with probes to trace memory accesses and
compute stack distance histograms. The monitor can be placed anywhere in
the memory system by connecting it as a form of extension cord between
two existing modules. For example `l2cache.mem_side = membus.slave`
would turn into `l2cache.mem_side = l2mon.slave` and `l2mon.master =
membus.slave`, assuming a monitor is already instantiated. Check
config.json or config.dot.pdf to ensure the monitors are instantiated as
expected.

To trace memory accesses, a `CommMonitor` needs to be attached to the
memory system and a `MemTraceProbe` needs to be attached to the
`CommMonitor`. To enable tracing in the probe, specify a `trace_file =
'mytrace.trc'` parameter. By appending '.gz' the trace will also be
gzipped. For example:
`monitor.trace = MemTraceProbe(trace_file="my_trace.trc.gz")`

The output trace is using Google's protobuf to get a compact trace with
a high-speed encoding/decoding. The trace can be dumped in
human-readable format by using the utility script
`util/decode_packet_trace.py`. The trace contains a time stamp, the
command and flags of the request, the physical address, the size, and a
generic ID. The traffic generator in gem5 also supports replaying these
traces.

### Traffic generation and trace playback

The `TrafficGen` module provides a generic framework for synthetic
traffic generation and trace playback. Each traffic generator is
independent, and has a single master port, through which requests are
issued, and responses received. The traffic generator does not care
about the data transported, and focuses solely on achieving a specific
spatial (address) and temporal (inter-transaction time) distribution.

The specific behaviour of the traffic generator is controlled by a
text-based configuration file. For an example of such a file, see
`tests/quick/se/70.tgen/tgen-simple-mem.cfg`. In essence the traffic
generator is a probabilistic state-transition diagram, where each stats
is a specific generator behaviour. The states, such as LINEAR, RANDOM,
TRACE and DRAM can be combined into an elaborate graph, and each state
also has a large range of configuration options.

## Packet allocation protocol

The protocol for allocation and deallocation of Packet objects varies
depending on the access type. (We're talking about low-level C++
`new`/`delete` issues here, not anything related to the coherence
protocol.)

  - *Atomic* and *Functional* : The Packet object is owned by the
    requester. The responder must overwrite the request packet with the
    response (typically using the `Packet::makeResponse()` method).
    There is no provision for having multiple responders to a single
    request. Since the response is always generated before
    `sendAtomic()` or `sendFunctional()` returns, the requester can
    allocate the Packet object statically or on the stack.

<!-- end list -->

  - *Timing* : Timing transactions are composed of two one-way messages,
    a request and a response. In both cases, the Packet object must be
    dynamically allocated by the sender. Deallocation is the
    responsibility of the receiver (or, for broadcast coherence packets,
    the target device, typically memory). In the case where the receiver
    of a request is generating a response, it *may* choose to reuse the
    request packet for its response to save the overhead of calling
    `delete` and then `new` (and gain the convenience of using
    `makeResponse()`). However, this optimization is optional, and the
    requester must not rely on receiving the same Packet object back in
    response to a request. Note that when the responder is not the
    target device (as in a cache-to-cache transfer), then the target
    device will still delete the request packet, and thus the responding
    cache must allocate a new Packet object for its response. Also,
    because the target device may delete the request packet immediately
    on delivery, any other memory device wishing to reference a
    broadcast packet past point where the packet is delivered must make
    a copy of that packet, as the pointer to the packet that is
    delivered cannot be relied upon to stay valid.

## Two memory system models: Classic and Ruby

The gem5 simulator includes two different memory system models, Classic
and Ruby, that incorporate the above mentioned general memory system
components. As the name suggests, the Classic memory system model is
inherited from the previous M5 simulator, while the Ruby memory system
model is based on the GEMS memory system model of the same name. Each
model is complementary and both have unique advantages and
disadvantages. This sub-section introduces each model and compares their
functionality. Then the next two respective top-level sections describe
each model in detail.

### Classic memory system

The Classic memory system model provides gem5 a fast, flexible and
easily configurable memory system at the cost of detail in the coherency
interactions. All objects within the Classic model inherit from
`MemObject` and connect together using ports. In particular, ports
support direct point-to-point connections between two `MemObjects` and
buses/crossbars connect two or more `MemObjects` together. Cache
coherence is maintained using an abstract MOESI snooping protocol where
state-transitions due to snoops occur instantaneously. Using this
methodology, the Classic memory system model has the following
advantages and disadvantages:

  - *Advantages*

<!-- end list -->

1.  **Fast Forwarding** - The Classic model supports atomic accesses,
    which as stated above, are faster than detailed accesses. This mode
    of operation is especially advantageous when one needs to fast
    forward to interesting parts of the execution. It is also the mode
    used when running gem5 in KVM mode.
2.  **Speed** - Not only does the Classic model support fast atomic
    accesses, but its timing accesses are relatively fast as compared to
    Ruby.
3.  **Ease of Configuration** - By simply modifying the python
    configuration, the Classic model allows one to create an arbitrary
    memory hierarchy. The Classic's abstract cache coherence protocol
    automatically extends to any memory hierarchy as long as it is
    composed of caches, crossbars, and bridges.

<!-- end list -->

  - *Disadvantages*

<!-- end list -->

1.  **Cache Coherence Flexibility** - While the Classic model allows one
    to create arbitrary systems composed of caches, crossbars, and cpus,
    the Classic model is restricted to its abstract MOESI snooping
    protocol. Modifying the protocol requires significant effort.
2.  **Cache Coherence Fidelity** - By not modeling transient states, the
    Classic model does not model protocol contention as detailed as the
    Ruby model.

### Ruby memory system

In contrast to the Classic model, the Ruby memory system model
sacrifices simulation speed to provide gem5 a flexible infrastructure
capable of accurately simulating a wide variety of memory systems. In
particular, Ruby supports a domain specific language called SLICC
(Specification Language for Implementing Cache Coherence) where one can
define many different types of cache coherence protocols. Essentially
SLICC defines the cache, memory, and dma controllers as individual
per-memory-block state machines that together form the overall protocol.
By defining the controller logic in a higher level language, SLICC
allows different protocols to incorporate the same underlining state
transition mechanisms with minimal programmer effort.

Unlike the Classic model, Ruby does not connect all objects together
using ports. Instead, ports only connect cpus and devices to the memory
system via the `RubyPort` object. Then within the Ruby memory system,
all objects are connected to each other via `MessageBuffers`.
MessageBuffers are similar to ports in that they provide objects a
standard communication interface. However, MessageBuffers include a
queue that stores messages while ports do not. Messages cannot be
enqueued and dequeued at the same simulated cycle and thus communication
across a message buffer is not instantaneous. The result is
MessageBuffers only support timing accesses and cannot support the
instantaneous atomic and functional accesses.

To summarize, the Ruby memory system model has the following advantages
and disadvantages:

  - *Advantages*

<!-- end list -->

1.  **Cache Coherence Flexibility** - Utilizing SLICC, Ruby can
    implement a wide variety of cache coherence protocols, from
    directory to snooping protocols and several points in between.
2.  **Fidelity** - Ruby accurately models both cache coherence and
    network related features in the memory system. In particular,
    SLICC's message trigger event methodology accurately models
    transient state timing. Also the Garnet network model integrated in
    Ruby accurately models network contention and flow control.

<!-- end list -->

  - *Disadvantages*

<!-- end list -->

1.  **Fast Forwarding** - Ruby does not support atomic accesses and thus
    does not have reasonable fast forwarding capability.
2.  **Speed** - As compared to the Classic model, Ruby is relatively
    slow. This is especially true when using the Garnet network model.
3.  **Ease of Configuration** - While SLICC is a powerful tool to model
    a wide variety of cache coherence protocols, the resulting protocols
    are optimized for a specific cache hierarchy configuration. As such,
    it is difficult to simply extend protocols to another level of
    cache.

