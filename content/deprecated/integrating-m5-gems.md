---
title: "Integrating m5 and gems"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

{|style="width:100%;text-align:center;white-space:nowrap;color:\#000"
|

<div style="font-size:202%;border:none;margin: 0;padding:.1em;text-align:center;color:#000">

The GEMS/M5 integration project

</div>

|}

## Sprint

We're having a coding sprint on January 13, 2009. The sprint begins at
9AM PST/11AM CST. We will begin with a phone call on Nate's conference
line and use IRC throughout the day.

### Goal

To get a "working" unified simulator by the end of the day.

### Tasks

  - Unified build environment using scons -- Arka w/ Nate and Steve
    supervising
  - Support system call emulation mode
  - Support full system mode
      - atomic support, especially load locked/store conditional --
        Derek
      - pio support
  - Deal with lack of first-class data support in Ruby
  - Configuration management
      - Option 1: Configuration checks between m5 and RubyConfig
      - Option 2: M5 front-end directly modifying ruby parameters
  - Testing infrastructure base on m5 infrastructure
      - Which tests?
  - What run modes to support? A fast Ruby-less or Ruby-lite mode?
  - Develop detailed list of future tasks

### Participants

  - Nathan Binkert (All Day)
  - Dan Gibson (All Day)
  - David Wood (All Day, except 12:30-2pm CST)
  - Derek Hower (All Day)
  - Steve Reinhardt (All Day except 10-10:30 PST)
  - Polina Dudnik (All Day)
  - Brad Beckmann (All Day)
  - Ali Saidi (All Day)
  - Arkaprava(Arka) Basu ( All Day)

### Communications

IRC channel: `irc.freenode.net` channel: `#m5dev`
IRC Client Recommendations:

| Operating System | Client                                    |
| ---------------- | ----------------------------------------- |
| Mac OS X         | [Colloquy](http://colloquy.info/)         |
| Windows          | [mIrc](http://www.mirc.com/)              |
| Linux/Unix       | [graphical client](http://www.xchat.org/) |
| Linux/Unix       | [text client](http://www.bitchx.com/)     |

## Repository

<ssh://m5sim.org//repo/gem5>

## Ruby-Side Short Term Tasks

  - ONGOING: References to Ruby's configuration parameters directly via
    their global names should be changed to reference them through
    static calls to RubyConfig instead. This can be done in small or
    large chunks, as time allows.
      - Time est: Hours
      - Difficulty: Trivial

<!-- end list -->

  - ONGOING: Find and remove random unused transactional memory
    remnants. Grep for XACT_, xact_, etc.
      - Time est: (up to) Hours
      - Difficulty: Trivial (Should be trivial at this stage)

<!-- end list -->

  - Convert Ruby's Profiler.\* to use M5's registered statistics
    interface.
      - Time est: Hours
      - Difficulty: Moderate (requires learning)

<!-- end list -->

  - Convert Ruby's warmup functionality to operate under M5's so-called
    'atomic' memory mode.
      - Time est: Hours
      - Difficulty: Easy-Moderate

<!-- end list -->

  - DONE Investigate compression for Ruby's warmup traces. We removed
    gzstream because of an incompatible license, but if we ever want to
    use warmup traces, we will probably want them compressed, somehow.
      - Time est: Minutes-hours
      - Difficulty: Easy-Moderate
          - COMMENT (ATTN: Derek) : We need to decide whether we need
            traces for Ruby now that we can use M5 atomic mode. If so
            and Derek still wants to keep compressing and decompressing
            traces, the current source of zlib available at
            <http://www.zlib.net/> contains three contrib/iostream
            folders implementing C++ wrappers around zlib. The license
            should be appropriate.

<!-- end list -->

  - Investigate the differences between Ruby's DRAM model and M5's DRAM
    model. Prepare a brief textual summary of how they differ (what
    functionality does one have and not the other).
      - Time est: Minutes-hours
      - Difficulty: Easy

<!-- end list -->

  - Integrate Ruby's random tester into M5 test framework
      - Time est: Minutes-hours
      - Difficulty: Easy-Moderate

<!-- end list -->

  - Establish a connection from Ruby to M5's physical memory or move
    memory totally into Ruby
      - Time est: Minutes-hours
      - Difficulty: Easy-Moderate

## Long Term Tasks

1.  Ruby-side: Data in caches (tentative: Polina)
      -
        I don't see a huge need for this, but if it is forced on us,
        it'll be good to have a junior grad student (JGS) do it.
        Basically we have to revive the old DATA_BLOCK flag, which
        worked in the 'research tree' when I joined the group. I've
        turned it on for my own reasons in the past, discovered it was
        broken, and put no effort into fixing it.
2.  Ruby-side: Python configuration adaptation
      -
        M5 uses a very different configuration system than Ruby, and
        Ruby's was based heavily on the Simics CLI. I have a hack in
        place, so that it is possible to change some Ruby parameters at
        runtime (any at compile-time), but that should be temporary --
        we should switch entirely to M5's style. It will little more
        than a lot of grep'ping and such, but again its a good
        familiarization excercise. At the same time, we can prune some
        fat from the configuration parameters.
3.  Ruby-side: M5 fast/timing mode support
      -
        Timing mode is Ruby's normal operation. Its possible that we can
        use 'fast' mode to warm caches (as we currently do from gzipped
        traces). This will require some new coding here and there, as
        well as testing.
          -
            As a point of terminology, the 'fast' mode is called
            'atomic' mode in M5 (since memory transactions complete
            atomically), not to be confused with support for
            processor-atomic memory operations discussed below. There's
            also a third 'functional' mode (see \[Memory System\#Access
            Types\]). I'd think that some support for functional mode
            would be required for syscall emulation, though I don't see
            where that is in Daniel's tarball (RubyMemoryPort has
            recvTiming() but not recvFunctional()). Using atomic mode
            for warmup would be a nice addition but isn't critical,
            presuming that you never needed it before (or did you do
            warmup via a different method?). -- Steve
4.  Ruby-side: Atomic support (Derek Hower)
      -
        This may be important for us in the long run. We'll do some kind
        of horrible nasty hack in the sprint, but we'll want something
        flexible, generic, and elegant in the long run. We'll need a
        clever JGS to make that happen. This will actually be quite
        challenging to integrate into existing protocols, as we need
        something like an M-locked state to really get the timing right.
        At the same time, we might also implement true write merging
        (read-merge-write timing), as an option (the other option is
        subblocked caches with per-subblock ECC).
          -
            I think there are three different issues here:
            1.  Support for Alpha LL/SC (which the M5 code confusingly
                calls "locked" operations).
            2.  Support for "normal" atomic RMW ops (SPARC swaps, etc.).
            3.  Support for uncached RMW ops (e.g., bus locking; x86
                only).
            Number 1 will require some hacking, though it can mostly be
            done outside the coherence protocol. (There are a few
            efficiency and potential livelock things you want to do at
            the coherence level, but I believe they're optional for just
            getting things to work.) Number 2 should be trivial the way
            M5 does them, as the swap operation is sent to the cache and
            only requires exclusive block access (though Dan is correct
            that if you want to get the timing precise it's a little
            trickier... but if you really wanted to be realistic you
            wouldn't be doing the operation at the cache anyway, I don't
            think). Number 3 is a big pain but it's not needed until we
            get further along with x86 FS mode (M5 doesn't do it
            currently either); just wanted to raise that spectre to get
            people used to the idea. -- Steve
5.  Ruby-side: Timing of uncached accesses( Arka)
      -
        Basically, we need add an 'isUncacheable' flag to network
        messages and modify SLICC to generate cache controllers that
        ignore messages with the isUncacheable flag set. That should
        effectively force the messages to traverse their normal miss
        path. As an optimization (depending on interconnect, etc.), we
        can add special routing capability to move straight to the
        memory controller and/or off-chip bridge. If we \*add\* some
        notion of an off-chip actor, that is.
          -
            Is modifying SLICC required? I would have thought that
            uncached accesses could be handled entirely in Ruby, but my
            understanding of the division between Ruby and SLICC is
            probably faulty. -- Steve
              -
                I think this can be done entirely in the Ruby sequencer.
                Unless there is a showstopper I'm not seeing now,
                modifying SLICC to handle this would be a little
                overkill. --Derek
6.  Ruby-side: Fix Directory Memory (tentative: Polina)
      -
        DirectoryMemory.C implements 'generic' directory data by
        allocating permanent directory state for all cache blocks in the
        physical memory. This is fine, except it is stored as an array
        of DirectoryEntry\*. The size of that array is MAX_ADDRESS /
        CACHE_LINE_SIZE_BYTES. That, too, is fine, except when the
        physical memory space isn't contigous. E.g. when addresses start
        at 0, run to 0x0ff, then resume at 0x100000 through 0x1000ff. It
        happens -- for instance when we simulate really large machines
        with huge complex backplanes. It /can/ happen in M5, too. Ruby
        also seems to leak memory because of this... even though its not
        really a leak. The solution is to move the whole data structure
        from array-based to some kind of balanced tree. Its been on the
        list of things to do for a long long time.

__NOTOC__
