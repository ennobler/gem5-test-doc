---
title: "Heterogenous Systems"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

(from Gabe)

I was thinking about how to set up m5 to support other architectures,
and decision which has a major impact on that is whether we want to
support heterogeneous systems. Here are the arguements I saw for and
against.

**Against**

Adding support for heterogeneous systems will be a big change. There are
many systems of m5 that aren't ready to support something that's not
alpha, let alone two things. If there are heterogeneous systems, the
code which implements each must be present. That means that a system has
to be in place to generate the appropriate pieces, which could be using
templates, namespaces, or setting up standard interfaces and using a
building block style approach. In all cases, this would mean carefully
and totally cutting the pieces apart from one another, since it can't be
assumed that any one thing will be there.

**For**

Allowing heterogeneous systems will allow simulation of mixed systems,
such as a satellite embedded device talking to a server implemented on,
for instance, MIPS and SPARC architectures respectively. Another setup
which also becomes possible are asymmetric multiprocessing systems which
use different ISAs for the different computing elements. I believe the
Cell processor is an example of this. Having a heterogeneous
architecture won't be as hard to implement once there have been
sufficient changes to allow *non-alpha* architectures at all. I would
estimate that the majority of the work involved will be to remove the
alphacentric nature of existing code. Code set up in this way should be
more modular and non-specialized, which would lead to better
implementation overall.

**My thoughts**

I favor implementing a system which supports hetergeneous architectures,
since that will be almost the same thing as supporting other
architectures at all. I think being able to support multi ISA asymmetric
multiprocessor systems would be a useful feature. Also, modularizing the
code fully would make m5 more usable for us, and more approachable for
people who want to use it for other things.

-----

I thought this was a settled issue: as we fix all the things that need
to be fixed to enable multiple ISAs with a single ISA selected at
compile time, we will:

1.  not consciously do anything that makes it unreasonably difficult to
    add support for heterogeneous systems, and
2.  make specific features support heterogeneous systems if the amount
    of work is equivalent to or only slightly more than not adding that
    support, but
3.  not commit to supporting heterogeneous systems fully at this time
    since we don't need it. That is, if there are features that don't
    fall under rule \#2, (they're needed solely for heterogeneous
    systems and/or are significantly more effort), then we're not going
    to do them right now because we have more important things to work
    on.

But if there aren't any necessary features that fall under rule \#3,
then it's possible that heterogeneous system support will just fall out.

Steve

-----

The problem is that there is a possible contradiction between the first
and second points. There very well may be situations where we'll need to
go out of our way to make heterogeneous systems not require a lot of
effort later. For example, the ISA compilation strategy you outlined
won't work for hetergenous systems since the switch header will point to
only one ISA. A system built around that will be hard to change later.
The direction things are headed seems to be to not worry about
heterogeneous systems, which is ok by me, but we should resolve which of
those two points is dominant.

Gabe

-----

Anything that's ISA-specific will be encapsulated as a specific type
that knows what ISA it belongs to. If we want to get fancier we can have
the Python figure it out.

For example, FreeBSDSystem should have an AlphaFreeBSDSystem subclass to
add on the Alpha-specific parts (via abstract virtual functions etc.).

\--[141.213.120.65](User:141.213.120.65 "wikilink") 16:03, 7 February
2006 (EST)
