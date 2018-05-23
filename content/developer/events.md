---
title: "Events"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

This page provides some high-level information about Events in M5
including how to create, schedule, and remove them.

The simplest way to use events is to create an EventWrapper which lets
you wrap an arbitrary function (that is based on a SimObject with an
event that can occur). An illustrative example is probably the easiest
way to describe the issue:

    class Bar : public SimObject
    {
        void foo();
        EventWrapper<Bar, &Bar::foo> fooEvent;
    };

When the class Bar is constructed the `SimObject` (this) pointer must be
passed to the `fooEvent` initializer. This creates an event with default
priority that hasn’t been scheduled. If you want to change the priority
you need to pass that to the initializer as well `fooEvent(this, false,
priority);`

Inside `Bar`, you can call `schedule(fooEvent, curTick +
some_delay_of_interest);` which will schedule the event for
some_delay_of_interest ticks in the future.

To see if the event is scheduled you can call `fooEvent.scheduled()` and
to see when it is scheduled you can call `fooEvent.when();`

If you want to deschedule the event (so it never fires) you can
deschedule like: `deschedule(fooEvent)`. Finally, if you want to change
when an event is scheduled for you can call `reschedule(fooEvent,
new_time);`

Additionally, there is a separate lower-level `Event` class that the
`EventWrapper` builds upon, however it’s use is generally not required.

## Events

### Event queue

### EventManager objects (I don’t know a lot about these)

### Event objects

### Time sync

