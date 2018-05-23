title: "I/O Base Classes"
date: 2018-05-13T18:51:37-04:00
draft: false
---

The base classes in `src/dev/io_device.*` allow devices to be created
with reasonable ease. The classes and virtual functions that must be
implemented are listed below. Before reading the following it will help
to be familiar with the [Memory_System](Memory_System "wikilink").

### PioPort

The PioPort class is a programmed I/O port that all devices that are
sensitive to an address range use. The port takes all the memory access
types and roles them into one `read()` and `write()` call that the
device must respond to. The device must also provide the
`addressRanges()` function with which it returns the address ranges it
is interested in. An extra `sendTiming()` function is implemented which
takes an delay. In this way the device can immediately call
`sendTiming(pkt, time)` after processing a request and the request will
be handled by the port even if the port bus the device connects to is
blocked. Because of this a PIO device should not call
`Port::sendTiming()` only `PioPort::sendTiming()` should be used.
`sendTiming()` causes a new `PioPort::SendEvent` is created and when the
event time is reached the packet is sent. If the send is not successful
then the packet is placed on a the transmit list and resent when
`recvRetry()` is called.

If desired a device could have more than one PIO port. However in the
normal case it would only have one port and return multiple ranges when
the `addressRange()` function is called. The only time multiple PIO
ports would be desirable is if your device wanted to have separate
connection to two memory objects.

### PioDevice

This is the base class which all devices senstive to an address range
inherit from. There are three pure virtual functions which all devices
must implement `addressRanges()`, `read()`, and `write()`. The magic to
choose which mode we are in, etc is handled by the PioPort so the device
doesn't have to bother.

Parameters for each device should be in a Params struct derived from
`PioDevice::Params`.

### BasicPioDevice

Since most PioDevices only respond to one address range `BasicPioDevice`
provides an `addressRanges()` and parameters for the normal pio delay
and the address to which the device responds to. Since the size of the
device normally isn't configurable a parameter is no used for this and
anything that inherits from this class is expected to write it's size
into `pioSize` in its constructor.

### DmaPort

The DmaPort is used only for device mastered accesses. The
`recvFunctional()`, and `recvAtomic()` methods are defined to panic as
the device should never receive a request on this port. the
`recvTiming()` method must be available to responses (nacked or not) to
requests it makes. The port has two public methods `dmaPending()` which
returns if the dma port is *busy (e.g. It is still trying to send out
all the pieces of the last request).* All the code to break requests up
into suitably sized chunks, collect the potentially multiple responses
and respond to the device is accessed through `dmaAction()`. A command,
start address, size, completion event, and possibly data is handed to
the function which will then execute the completion events `process()`
method when the request has been completed. Internally the code uses
`DmaRequestState` to manage what blocks it has received and to know when
to execute the completion event.

### DmaDevice

This is the base class from which a DMA non-pci device would inherit
from, however none of those exist currently within M5. The class does
have some methods `dmaWrite(), dmaRead()` that select the appropriate
command from a DMA read or write operation.

### PCI devices

### Explanation of platforms and systems, how they’re related, and what they’re each for
