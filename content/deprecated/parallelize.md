---
title: "Parallelize gem5"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---


Parallelizing the simulation engine to run on shared-memory multicore
systems has long been a goal for several of us. Unfortunately, it rarely
makes it to the top of anyone's to-do list. This page tries to summarize
where we're at and which direction we're headed so that others who would
like to help can jump in and help (or provide feedback on our future
direction).

## Goals

1.  Our primary goal is to get decent speedup modeling moderate-size
    parallel configurations (say 16-64 cores) on small-scale
    shared-memory multi-core systems (say 4-8 cores). You can't even buy
    single-core systems anymore, and most developers use multi-core
    systems to accelerate compilations; our focus is on getting *some*
    benefit out of those additional cores when you'd like to reduce the
    latency of a single simulation. In particular, we are **not**
    focusing on:
      - Getting linear speedup (see [Wood & Hill "Cost-Effective
        Parallel
        Computing"](http://doi.ieeecomputersociety.org/10.1109/2.348002)
        on why that's not necessarily a worthwhile goal)
      - Speeding up simulations of systems with one or just a few cores
      - Scaling speedup to very large numbers of host cores (more than a
        typical developer would have available)
    <!-- end list -->
      -
        We're not opposed to these things, if someone wants to try;
        they're just not things we're focusing on. In particular,
        objections to our current plan based on "but that will never
        support X", where X is on the list above, are not likely to be
        considered convincing.
2.  Parallel simulation should be deterministic by default. This is a
    necessity for maintaining developer sanity. Modes that optionally
    forgo determinism to provide enhanced performance are OK, but they
    should not be the default mode of parallel execution.
3.  The parallelization scheme should be flexible enough to allow
    experimentation. There has been a mini-renaissance of parallel
    simulation work recently (e.g., SlackSim from USC and Graphite from
    MIT), and it would be good if gem5 could be a platform for further
    research on parallel simulation techniques.

## Status

  - In fall 2008 (\!) Nate revamped the Event objects so they are no
    longer explicitly associated with specific queues. Instead, the
    EventManager class (which is a base class of SimObject) controls
    which queue an event gets scheduled on. This allows each SimObject
    to schedule its events on a specific queue. At this point all
    SimObjects point to the single global event queue (mainEventQueue)
    though. See <http://repo.gem5.org/gem5/rev/b194a80157e2>.
  - Just before Christmas 2010, Steve did an initial implementation of
    multiple event queues. Because this work is still very preliminary,
    it is not committed, but is available as a patch on reviewboard
    [here](http://reviews.gem5.org/r/846). See the patch for further
    documentation.
  - At this rate, there will be another major development in early 2013
    :-).

## Next Steps

1.  Get feedback on multi-queue patch and enhance it.
2.  Figure out a synchronization strategy; my expectation is that our
    inter-object synchronization would revolve around Ruby
    MessageBuffers (or perhaps Ports?) so that we can have each CPU node
    and the interconnect running on separate threads.
3.  ???

## Nate's old plan

The sections below are the original contents of this page, written by
Nate in 2008 (before he made the EventManager change described above).
Several of these steps have already been taken care of, and many of the
future steps aren't exactly aligned with the current plan, but I (Steve)
thought I'd leave it here so we can look back occasionally and see if
we're missing anything.

### Parallelizing M5

Parallelizing M5 has been a long term goal of mine (nate) for quite some
time.

Here's my plan for going about making this happen:

1.  Get rid of the global mainEventQueue
2.  Add an EventQueue pointer to every SimObject and add
    schedule()/deschedule()/reschedule() functions to the Base SimObject
    to use that event queue pointer.
3.  Change all calls to event scheduling to use that EventQueue pointer.
    An example of this is something like this:
      - old:
          - new LinkDelayEvent(this, packet, curTick + linkDelay);
      - new:
          - Event \*event = new LinkDelayEvent(this, packet);
          - this-\>schedule(event, curTick + linkDelay);
4.  Remove the schedule/deschedule/reschedule functions on the Event
    object. Now, you must create an event and schedule it on an event
    queue.
      - See note below about outstanding issues.
5.  Add an EventQueue pointer to the SimObjectParams class
6.  We're going to keep the mainEventQueue, but it will be for certain
    global functions like managing barriers simulator exits and the
    like.
7.  Create EventQueues in python and pass the pointer to each SimObject
    via the Params struct for every object
      - In the first phase of implementing parallel M5, I do all of the
        steps up to this point, and just create one event queue (the
        mainEventQueue) and populate every sim object with that one
        event queue. This should essentially keep the status quo.
8.  Tell SCons to link M5 with -lpthread
9.  Create a set of wrapper classes for the pthread stuff since it would
    be nice to eventually support other mechanisms.
10. Add support for the python code to determine the number of CPU cores
    it has available (automatically using /proc maybe, but with the
    ability to override the number with a command line option).
11. Create a barrier event that can be used to synchronize sets of event
    queues
      - Initially, the barrier event will cause ticks to be run in
        lock-step, guaranteeing that all cycles are doing in order
12. Create a point-to-point synchronization events for controlling the
    slack between event queues
      - The plan is to rely mainly on these events for maintaining slack
        in the system.
      - One major idea with these events is that they will be squashed
        as frequently as possible to avoid synchronization.
      - It may make more sense to build this directly into the event
        queue, but the all-to-all nature of the synchronization may make
        this less desirable.
13. Create one event queue per thread and one thread per CPU core. Bind
    logical groups of objects to different EventQueues.
14. Create certain objects which can use multiple EventQueues.
      - This will be done on is the EtherLink object, allowing two
        separate systems to be simulated on two separate cores
      - Next, I'll do this on is the bus object so that each core can
        run on a different event queue
      - I'll probably also create some sort of etherlink like or
        etherbridge like object for connecting two arbitrary memory
        objects across event queues. This may be done instead of doing
        the bus directly.

### Outstanding Issues

I'd like to remove the queue pointer from the event object since there
is only one use case where you've scheduled an event and you don't know
which queue it's on if you want to de/reschedule it. It's for repeat
events like the SimLoopExitEvent.

Here are the options:

1.  Leave the queue pointer in the object
2.  Pass the queue pointer as a parameter to the process() function
3.  Record the queue pointer in just those objects that require it
4.  Create a new flag to the event called AutoRepeat, create a virtual
    function that can be called to determine the repeat interval, and
    add support for repeat in the event queue
5.  Create a thread local global variable called currentEventQueue. (I
    hate this idea)

I go back and forth as to the right thing to do. I'd really like to
avoid the queue pointer in all objects so we can keep events small, but
I guess it can easily be argued that I shouldn't keep that optimization
unless I know that it will pay off, but the only way to know if it will
pay off is to just do it. I also basically hate the last idea and it's
on the bottom of my list. One issue is that because of the committed
instruction queue, there can be more than one event queue in a given
thread.

__NOTOC__
