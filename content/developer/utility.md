---
title: "Utility Code"
date: 2018-05-13T18:43:03-04:00
draft: false
---

### Bitfield functions

### BitUnion classes

### FastAlloc

FastAlloc has been removed from the code base since late May 2012.
Instead the build scripts will pick up tcmalloc (a malloc() version used
also at Google), the observed performance gain of replacing tcmalloc
over fastalloc is around 6%.

### IntMath

### panic, fatal, warn, inform, hack: which?

There are error and warning functions defined in `src/base/misc.hh`:
`panic()`, `fatal()`, `warn()`, `inform()`, and `hack()`. The first two
functional have nearly the same effect (printing an error message and
terminating the simulation), and the latter three print a message and
continue. Each function has a distinct purpose and use case. The
distinction is documented in the comments in the header file, but is
repeated here for convenience because people often get confused and use
the wrong one.

  - `panic()` should be called when something happens that should never
    ever happen regardless of what the user does (i.e., an actual m5
    bug). `panic()` calls `abort()` which can dump core or enter the
    debugger.
  - `fatal()` should be called when the simulation cannot continue due
    to some condition that is the user's fault (bad configuration,
    invalid arguments, etc.) and not a simulator bug. `fatal()` calls
    `exit(1)`, i.e., a "normal" exit with an error code.
  - `warn()` should be called when some functionality isn't necessarily
    implemented correctly, but it might work well enough. The idea
    behind a `warn()` is to inform the user that if they see some
    strange behavior shortly after a `warn()` the description might be a
    good place to go looking for an error.
  - `hack()` should be called when some functionality isn't implemented
    nearly as well as it could or should be but for expediency or
    history sake hasn't been fixed.
  - `inform()` Provides status messages and normal operating messages to
    the console for the user to see, without any connotations of
    incorrect behavior. For example it's used when secondary CPUs being
    executing code on ALPHA.

The reasoning behind these definitions is that there's no need to panic
if it's just a silly user error; we only panic if m5 itself is broken.
On the other hand, it's not hard for users to make errors that are
fatal, that is, errors that are serious enough that the m5 process
cannot continue.

### random number generation

### reference counting pointers
