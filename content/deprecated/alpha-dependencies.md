---
title: "Alpha Dependencies"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

## Purpose

The purpose of this page is to list the areas where M5 is dependent on
the alpha architecture, and discuss ways to remove or quarantine the
dependencies.

## Dependencies

### VPtr class

The VPtr class uses the page size for Alpha in an assert which
determines whether or not to an access spans page boundaries. Other than
this, this file is not alpha specific, and can be moved somewhere else.
My suggestion is that it moves to sim.

Gabe

  -
    It looks like the only place VPtr is currently used is in
    kern/linux_threadinfo.hh. Is VPtr compatible with the new memory
    system? --[Stever](User:Stever "wikilink") 22:22, 12 February 2006
    (EST)

### Pseudo Instructions

The implementation of the pseudo instructions doesn't seem to be very
alpha dependent, other than knowing how to get function parameters. This
might be pulled into an interface called abi.hh, for instance, which
knows how to speak in the appropriate architecture's ABI. The
psuedo_inst.hh and psuedo_inst.cc can use this interface, and live
outside of arch, probably in sim.

Gabe

It seems like this interface already exists, for the most part, in
arguments.hh.

\--[Gblack](User:Gblack "wikilink") 13:25, 10 February 2006 (EST)

After thinking about it more, there seems to be inherent problems in
implementing something like arguments.hh in a completely architecture
independent way. One problem is that you can't always return the same
thing for a particular argument without knowing certain things about it,
like if it's a floating point value, and the particulars that are
important are probably not completely consistent across architectures.
Also, accessing values beyond the ones held in registers means accessing
memory, and that introduces more complications. Instead, what I think
should be done is to move the parameter preperation into the decoder. In
other words, rather than just passing the execution context to the
pseudo instruction function, the execution context and whatever paramers
the function needs are passed in after being pulled from wherever, and
the function can just operate as standard C++ code with normal
arguments. The complication in the decoder would be minimal, and that
would allow the pseudo instructions to be reused for all of the
architectures.

\--[Gblack](User:Gblack "wikilink") 18:06, 10 February 2006 (EST)

  -
    I see you appear to have implemented this already... it looks good
    to me. --[Stever](User:Stever "wikilink") 22:09, 12 February 2006
    (EST)

### Misc Regs in CPU Model

There are some files in the model that access some of the miscellaneous
registers by name. The problem is that misc. regs differ per
architecture.

For instance, the MIPS has coprocessor0 registers which can be looked at
as a version of misc. regs that need to be accessed.

Also, when we deallocate or allocate context (or even copy and restore
data for syscalls) we copy some of the misc. regs by name.

To make this work for different archs. we either need to have all of
these cpu files for misc. regs in ISA-specific folders or access the
misc. regs by index (bounded by a size variable). Since there are only
maybe 3 or 4 files I think the former (ISA-specific folders) is a better
idea

Korey

Could you list which files specifically your talking about, preferably
with function names or even line numbers?

Gabe

  -
    The original idea was that every ISA would define its own
    MiscRegFile struct. Any piece of code that accesses individual regs
    in the struct would have to be ISA-specific.
    --[Stever](User:Stever "wikilink") 22:13, 12 February 2006 (EST)

I'm part way through defining the MiscRegs class that we decided to use,
which would have all of the misc regs in it and be ISA specific. Right
now it has the FPCR, Uniq, load locked flag/addr, and the IPRs in it. It
has methods readMiscReg and writeMiscReg defined on it. There are two
main problems I have encountered so far.

The first is deciding where to define the enums used to index into the
misc reg file. While the FPCR and Uniq registers already have a
dependence tag entry in the DependenceTag enum, the IPRs are in a
separate enum, and in a separate file. Because C++ doesn't let you just
expand an enum, it's difficult to choose where to define the misc regs.
For now I've left the definitions the same, where the FPCR, uniq regs
are defined in the DependenceTags (and I also added the load locked
regs), and the IPRs are defined in a separate enum. This requires the
two functions to take an input of an int, not a specific enum type,
which makes things a little less clear. However, for lack of a better
answer, I've defaulted to leaving the definitions the same. In the
future I may opt to put both the misc regs and the IPRs into one enum
which is defined for non-full system in the isa_traits.hh file, and
defined for full_system in isa_fullsys_traits.hh. It means that
certain names will be duplicated across the files, but hopefully it
doesn't happen too much and isn't changed often.

The second problem is that the current readIpr function has a reference
to a Fault passed in. This is only used for two cases in readIpr: when a
write-only register is accessed, and when the index doesn't match any
IPR. In those cases it returns an UnimplementedOpcodeFault. However,
because readIpr calls now go through the readMiscReg function, the
readMiscReg function must take in a reference to a Fault. This means
that code that previously accessed the FPCR or Uniq register will have
to pass in a dummy Fault. Unforutnately because it's a reference, I
can't set the default argument for it to NULL. While not a huge problem,
it does make the code a bit messier for just two cases that only exist
in full system mode. --[Ktlim](User:Ktlim "wikilink") 16:37, 23 February
2006 (EST)

### ev5 IPRs

The file arch/alpha/ev5.hh is used directly in a few places in the
non-alpha portion of m5 where the functions setIpr and readIpr are
defined. These files are cpu/o3/cpu.hh and cpu/o3/regfile.hh. These
files shouldn't be tied to a alpha, and even more so to a specific type
of alpha. I think the best solution would be to move any Ipr related
functionality to an intentionally alpha specific version of the cpu,
which I believe cpu/o3/alpha_cpu.\[hh|cc\] is for. Unfortunately, there
doesn't seem to be any place to declare the actual storage location for
the IPRs other than in regfile.hh where they are currently.

To fix this, I propose that the way the CPU model is specialized is
change to be based on inheritance, rather than being based on templates.
That would allow an AlphaCPU (or similar) to have an IPR register file
declared internally to itself. The functions which manipulate the IPR
would be defined at that level as well.

\--[Gblack](User:Gblack "wikilink") 18:37, 12 February 2006 (EST)

  -
    Is this a general problem or an O3 CPU model design issue? Is the
    problem with IPR storage or the function call interface to access
    IPRs?

<!-- end list -->

  -
    The register file is going to be ISA-specific anyway (e.g., the
    number of regs and MiscRegFile will depend on the ISA), so I don't
    see how allocating storage for IPRs in regfile.hh is different. I
    would guess that every CPU has something along the line of IPRs even
    if they're called something different. Another possibility would be
    to allocate space for necessary IPRs inside of MiscRegFile.

<!-- end list -->

  -
    We had also decided previously that the IPRs that are really
    internal control registers (not just scratch space) should be
    associated with the things they control and not with the register
    file. For example, accesses to TLB control IPRs should be diverted
    directly to the TLB, as opposed to storing the values in a central
    IPR reg file and forcing the TLB to look there.
    --[Stever](User:Stever "wikilink") 22:19, 12 February 2006 (EST)

The register file will be ISA specific in certain parameters, like the
number and width of registers, but the IPR registers are more
fundementally specific to Alpha. By specific, I mean that all the
registers are referred to by name in the regfile.hh. This, especially in
its current form, shouldn't be in general purpose code. My understanding
is that the IPRs are basically control registers. If that's correct
(please let me know otherwise), those should be stored in the
MiscRegFile with the other control registers. It's not immediately clear
without looking at the code more where the readIpr and setIpr functions
would go in that situation.

\--[Gblack](User:Gblack "wikilink") 00:58, 13 February 2006 (EST)

  -
    As far as the cpu/o3/cpu.hh include of arch/alpha/ev5.hh, that's
    definitely in the wrong place. That was a relic of some old code I
    had used as a building block for full system mode. Considering that
    it's reading from an IPR to get the specific ITB/DTB ASN, it should
    probably be at the AlphaFullCPU level. I'm not too clear on what you
    mean by specialization through inheritance. The most derived level
    of CPU is the AlphaFullCPU which does have methods defined for
    handling the IPRs in it, although they just defer to the register
    file. If you wanted a separate misc reg file that's specific to each
    ISA, you could put it at this level, though as Steve has mentioned,
    we're still having discussions about where exactly to put all those
    other registers.

<!-- end list -->

  -
    For the cpu/o3/regfile.hh include of arch/alpha/ev5.hh, I'm not sure
    how to handle this. The solution will be linked to where exactly we
    decide to place all of the registers. If we continue to have one
    centralized location for all IPRs, then it may make sense for the
    regfile (or maybe a class derived from the regfile) to store those
    IPRs and provide methods to read and write to them. If it's
    decentralized, then the control for reading and writing those
    registers probably needs to go to the AlphaFullCPU. Some storage may
    still be in the regfile.hh (if we agree that all ISAs have some sort
    of "MiscRegFile" and "ipr" class that can be obtained through the
    ISA namespace).

<!-- end list -->

  -
    Not all IPRs are control registers. Some are just used to place
    values when faults happen, such as the virtual address register,
    which gets the faulting address written to it when the ITB or DTB
    fault. The problem is that while it's convenient to logically think
    of the IPRs as one big array of registers, that's not necessarily
    how they act or are accessed, and it creates some problems when
    moving over to using ExecContext as an interface and not for storing
    any state. --[Ktlim](User:Ktlim "wikilink") 14:31, 13 February 2006
    (EST)

### Faults

arch/alpha/faults.hh and arch/alpha/faults.cc describe some types of
faults which can occur, and provide a mapping to the names of the
faults. This mechanism needs to be pared back. One the one hand, if it's
too specific it doesn't allow for variations in faults in other
architectures. On the other hand, if it's general, it can force
generality on the faults in other architectures that need to be
specific. For instance, there is only one entry in faults.hh for an
Unimplemented Opcode fault, but in SPARC, there are 3 exceptions this
could map to. These are "illegal_instruction", "unimplemented_LDD",
and "unimplemented_STD". In another example, there is only 1 processor
reset fault in faults.hh, but SPARC defines 4 matching exceptions,
"watchdog_reset", "externally_initiated_reset",
"software_initiated_reset", and "power_on_reset".

In order to address these issues, I propose that faults.hh be reduced in
scope to just those exceptions which are -very- common across
architectures with almost no semantic variation, namely
"Machine_Check_Fault" and "No_Fault". The other faults can be
enumerated internally to the architecture, which is where they almost
always generated and where they are consumed. The exact mechanism this
happens with needs to change as well, since enums can't be extended once
they're defined. One way to do this would be to define constants inside
a namespace. A problem with this approach would be that it would be
difficult to assure that there were never collisions between the
different places faults were defined. Another approach would be to
create a Fault class which would be inherited by the more specialized
Faults. A pointer to this class could be passed around by code which is
unaware of the specialization while the Fault is enroute to where ever
the architecture deals with it.

\--[Gblack](User:Gblack "wikilink") 18:56, 12 February 2006 (EST)

  -
    I thought we had decided that we would have an ISA-independent base
    class that ISAs could derive from to define their own fault types
    (with associated state), and that functions would return fault
    object pointers, with a null pointer indicating no fault (thus
    eliminating the No_Fault value).

<!-- end list -->

  -
    I'd still like to see some attempt to organize common fault concepts
    in an ISA-independent fashion; so for example we could have an
    ISA-independent IllegalInstruction class, but SPARC could derive
    from that to create the three specific subclasses we need.
    --[Stever](User:Stever "wikilink") 22:25, 12 February 2006 (EST)

That's what I remember deciding. This is here for completeness.

\--[Gblack](User:Gblack "wikilink") 00:59, 13 February 2006 (EST)

There were complications implementing the Fault classes as described
above for various reasons including ones having to do with fault
statistics and fault vectors. The currently implemented system has a set
of system wide Faults which are generated by non-isa code, and a second
set which are maintained by the ISA. Eventually, the Faults should be
pulled out of everything but the CPU and the ISA, and be defined and
maintained by the ISA. The CPU will have a set of interface functions
which will return a small subset of faults for certain situations, which
will allow the CPU to pass some faults directly to the ISA.

\--[Gblack](User:Gblack "wikilink") 01:53, 16 February 2006 (EST)

### ExecContext and CPU models

In various parts of the XC and the CPUs there are very Alpha-specific
code segments. For example, in the ExecContext, the read and write
functions will set/check the lock addr and lock flag when doing accesses
in full system mode. The SimpleCPU has interrupt code for Alpha (and
probably more specifically the ev5) in its tick() function. What was our
plan for handling these?

In the CPU model I had planned to abstract away the functions that were
specific to Alpha into the AlphaISA itself. In FastCPU (mostly a clone
of SimpleCPU), the interrupt code that existed in SimpleCPU is replaced
by a call to TheISA::checkInterrupts(this). The other two places that
were mostly ISA specific were clearing the zero registers, and calling
the proper trap function, both of which are abstracted away to the ISA.

Should we try something similar for ExecContext? It might be somewhat
more difficult because the code in ExecContext has control statements in
it, such as "return NoFault". We'd have to check for specific return
values from the call to the ISA, and handle them appropriately, which
somewhat defeats the idea of removing the ISA dependences. Perhaps we'll
just punt for now and leave it as it is, where the code is guarded by a
\#if defined(TARGET_ALPHA).

Also, there are ISA specific structures within both the XC and the CPUs.
There are AlphaITB and DTB pointers in both. I think we agreed upon (or
maybe just Nate suggested) having some sort of semi-opaque ISA object
that holds onto ISA specific objects such as the Alpha ITB and DTB. It
could live in the CPU and anything that needs it could have a pointer to
it. We'd have to define a specific interface for it so that the ITB and
DTB can be used.

### Object Loader flexibility

Right now, the object loader expects that all the binaries it reads will
be for Alpha, and determines what architecture to use based on the type
of file, ie. ecoff is Tru64, elf is Linux, and aout is PAL code. Solaris
supports all three formats, and Linux supports at least elf and aout.
There is also obviously a need to load binaries for other architectures
as well. What I think would be a good solution would be for the loader
to determine from the binary what sort of file it is, ie what
architecture it wants to run on and what os it's for. If those things
exist and work in that combination, m5 reports what it will use and
starts the program appropriately. If that combination isn't supported,
then m5 can panic. I'm most familiar with elf, and for elf it would be
relatively easy to determine what the requirements of the binary are. Is
it harder for the other formats? Also, how could we determine if a
particular combination of OS and architecture will work? We'll need to
do that regardless if we want to load object files safely.

\--[Gblack](User:Gblack "wikilink") 04:02, 1 March 2006 (EST)

  -
    The framework is already there for deriving both the architecture
    and the OS from the binary: there are enums in ObjectFile to report
    both of these. It doesn't (quite) assume that all the binaries it
    sees are Alpha; that's just the only one it recognizes right now
    :-). The purpose of the tryFile() method is to see if the object is
    of the given type, and if so, create an ObjectFile object that knows
    the appropriate ISA and OS it's for.

<!-- end list -->

  -
    Note that the ecoff tryFile() only reports Tru64 if it's an Alpha
    binary, otherwise it gives up. So a SPARC Solaris ecoff file would
    not be a problem, particularly if that's the only SPARC ecoff
    platform we know of. Similarly for elf; there's a check in there for
    the Alpha but Ali commented it out because it's "not official". I
    don't know if he found something that uses a different machine code
    or not. Ali?

<!-- end list -->

  -
    Right now m5 does auto-detect the OS and generate the correct
    syscall emulation object based on the executable (see
    LiveProcess::create() in sim/process.cc). This is not hard since the
    executable is a parameter to the Process object which is what
    determines the syscall emulation. This would also be the right place
    to detect an unsupported ISA/OS combination (assuming there are
    combinations that the loader recognizes but we don't support).

<!-- end list -->

  -
    I had thought about automatically generating the correct CPU ISA
    based on the binary as well; I agree that would be cool. It would be
    significantly harder though since we would have to defer creating
    the CPU object until it knows what binary it will be running. This
    dependence doesn't fit at all in our current initialization scheme,
    which assumes that we can create all the objects in one pass and
    then make another pass if necessary to resolve init-time dependences
    among objects. That is, having the \*creation\* of one object (i.e.,
    the CPU) depend on some post-creation initialization of another
    object (the Process object) just doesn't fit the mold. So even
    though it would be cool, I think in the short term anyway we should
    rely on users to specify AlphaCPU or SparcCPU or MipsCPU in their
    config file, and then just generate an error if they mess up. The
    number of people that will be doing cross-ISA experiments is
    probably vanishingly small anyway.
    --[Stever](User:Stever "wikilink") 08:22, 1 March 2006 (EST)

<!-- end list -->

  -

      -
        From what I've seen so far, the changes to the ELF loader code
        have been very small. The only things necessary were adding a
        Mips 'enum value' in object_file.hh, removing the panic that m5
        throws when it sees a 32-bit ELF in the elf_object.cc file, and
        finally calling the ElfObject constructor with the enum value I
        specified earlier. (This is also assuming one has already made
        edits so that you load a <your arch here>LinuxProcess too). The
        GELF library looks to be a real lifesaver since it handles both
        32 & 64-bit formats. The ELF file seems to load correctly, so
        I'm wondering am I overlooking something I need to change?

<!-- end list -->

  -

      -
        You need to do something in sim/process.cc to create a
        MipsLinuxProcess once you've detected that it's a MIPS binary
        (along with whatever it takes to set up the MipsLinuxProcess
        object).

<!-- end list -->

  -
    I've created a new bridging header called "process.hh" which brings
    in all the different types of processes for a particular
    architecture. This is because not every OS is available for every
    ISA, so they can't invidividually be bridging, and including each
    individaully all the time brings in alot of other ISA specific
    files. In each process.hh, there is also a "createProcess" function,
    which is used to generate the appropriate process object from an
    object file or to panic if the object file doesn't fit with the ISA.
    \--[Gblack](User:Gblack "wikilink") 22:08, 2 March 2006 (EST)

<!-- end list -->

  -
    A problem which has come up with the libelf as distributed with m5
    is that it is has different constants defined than the one installed
    locally on my computers. The local version is used first, which
    causes compilation errors when certain constants are used,
    specifically the ones I need for 64 bit/v9 SPARC. Several possible
    solutions are to update the version used in m5 so that it is in line
    with the local version, remove the version in m5 and use only the
    local version, or force m5 to use only it's own internal version.
    Some problems with using the internal version only have been that
    gcc, despite what seems to be in the documentation, is not allowing
    m5 to direct it to find it's internal version first. It is also not
    possible to force the correct version by changing the include
    directives or their paths because there are includes inside the file
    which use \<\>s and the normal paths. Preprocessing the files so
    that they don't have includes also doesn't work, because the macros
    which define the constants it uses go away, and the issue of getting
    m5 to find the correct header files during this step is the same as
    in the original.
    \--[Gblack](User:Gblack "wikilink") 22:08, 2 March 2006 (EST)

<!-- end list -->

  -
    gcc needs to have both the path to the libelf directory, and that
    path including the libelf directory, indicated with the -I option to
    find the m5 version of libelf always. Also, in libelf, the line
    "\#undef __LIBELF_HEADER_ELF_H" in lib/sys_elf.h.in needs to
    be commented out to prevent libelf from including the system header
    elf.h, which is provided with glibc, and to instead use its internal
    elf_repl.h. Making these changes, as well as updating libelf to
    have a complete list of constants, has cleared up the current
    problems with that library and its headers.
    \--[Gblack](User:Gblack "wikilink") 02:04, 4 March 2006 (EST)

