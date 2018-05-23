---
title: "Serialization"
date: 2018-05-13T18:51:37-04:00
draft: false
---

## Saving/Restoring object state

### Dealing with Events

### Handling SimObject Pointers

## Phases of Object Initialization

If restoring from a checkpoint, loadState(ckpt) is called on each
SimObject. The default implementation simply calls unserialize() if
there is a corresponding checkpoint section, so we get backward
compatibility for existing objects. However, objects can override
loadState() to get other behaviors, e.g., doing other programmed
initializations after unserialize(), or complaining if no checkpoint
section is found. (Note that the default warning for a missing
checkpoint section is now gone.)

If not restoring from a checkpoint, we call the new initState() method
on each SimObject instead. This provides a hook for state
initializations that are only required when \*not\* restoring from a
checkpoint.
