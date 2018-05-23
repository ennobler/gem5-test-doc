---
title: "Defining CPU models"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

## Overview

First, make sure you have basic understanding of how the CPU models
function within the M5 framework. A good start is the [CPU
Models](CPU_Models "wikilink") page.

This brief tutorial will show you how to create a custom CPU model
called 'MyCPU', which will just be a renamed version of the
AtomicSimpleCPU. After you learn how to compile and build 'MyCPU', then
you have the liberty to edit the 'MyCPU' code at your heart's content
without worrying about breaking any existing M5 CPU Models.

## Port C++ Code for MyCPU

The easiest way is to derive a new C++ class of your CPU Model from M5
CPU Models that are already defined and the easiest model to start with
is the 'AtomicSimpleCPU' located in the 'm5/src/cpu/simple' directory.

For this example, we'll just copy the files from the 'm5/src/cpu/simple'
and place them in our own CPU directory: m5/src/cpu/mycpu.

    me@mymachine:~/m5$ cd src/cpu
    me@mymachine:~/m5/src/cpu$ mkdir mycpu
    me@mymachine:~/m5/src/cpu$ cp -r simple/* mycpu

Now check the mycpu directory to make sure you've copied the files
successfully:

    me@mymachine:~/m5/src/cpu$ cd mycpu
    me@mymachine:~/m5/src/cpu/mycpu$ ls
    AtomicSimpleCPU.py  BaseSimpleCPU.py  SConscript  SConsopts  TimingSimpleCPU.py
    atomic.cc  atomic.hh  base.cc  base.hh timing.cc  timing.hh

Let's move the AtomicSimpleCPU.py object file to MyCPU.py and remove the
TimingSimpleCPU.py file. We'll edit MyCPU.py a little later:

    me@mymachine:~/m5/src/cpu/mycpu$ mv AtomicSimpleCPU.py MyCPU.py
    me@mymachine:~/m5/src/cpu/mycpu$ rm TimingSimpleCPU.py
    me@mymachine:~/m5/src/cpu/mycpu$ ls
    BaseSimpleCPU.py MyCPU.py  SConscript  SConsopts  atomic.cc  atomic.hh  base.cc  base.hh timing.cc  timing.hh

Since we want to change 'AtomicSimpleCPU' to 'MyCPU' we will just
replace all the names in the atomic.\* files and name them mycpu.\*
files:

    me@mymachine:~/m5/src/cpu/mycpu$ perl -pe s/AtomicSimpleCPU/MyCPU/g atomic.hh > mycpu.hh
    me@mymachine:~/m5/src/cpu/mycpu$ perl -pe s/AtomicSimpleCPU/MyCPU/g atomic.cc > mycpu.cc
    me@mymachine:~/m5/src/cpu/mycpu$ ls
    BaseSimpleCPU.py MyCPU.py SConscript SConsopts atomic.cc  atomic.hh  base.cc  base.hh
    mycpu.hh mycpu.cc timing.cc  timing.hh

The last thing you need to do is edit your mycpu.cc file to contain a
reference to the 'cpu/mycpu/mycpu.hh" header file instead of the
'cpu/simple/atomic.hh' file:

    #include "arch/locked_mem.hh"
    ...
    #include "cpu/mycpu/mycpu.hh"
    ...

NOTE: The AtomicSimpleCPU is really just based off the BaseSimpleCPU
(src/cpu/simple/base.hh) so your new CPU Model MyCPU is really a
derivation off of this CPU model. Additionally, the BaseSimpleCPU model
is derived from the BaseCPU (src/cpu/base.hh). As you can see, M5 is
heavily object oriented.

## Making M5 Recognize MyCPU

Now that you've created a separate directory and files for your MyCPU
code (i.e. m5/src/cpu/mycpu), there are a couple files that need to be
updated so that M5 can recognize M5 as a build option:

  - *m5/src/cpu/mycpu/MyCPU.py*: Edit your MyCPU python script file
    (e.g. MyCPU.py) so that your CPU can be recognized as a simulation
    object. For this example, we will just use the same code tha was
    previously in AtomicSimpleCPU.py file but replace the name
    'AtomicSimpleCPU' with 'MyCPU'.

<!-- end list -->

    from m5.params import *
    from m5 import build_env
    from BaseSimpleCPU import BaseSimpleCPU

    class MyCPU(BaseSimpleCPU):
        type = 'MyCPU'
        width = Param.Int(1, "CPU width")
        simulate_data_stalls = Param.Bool(False, "Simulate dcache stall cycles")
        simulate_inst_stalls = Param.Bool(False, "Simulate icache stall cycles")
        icache_port = Port("Instruction Port")
        dcache_port = Port("Data Port")
        physmem_port = Port("Physical Memory Port")
        _mem_ports = BaseSimpleCPU._mem_ports + \
                        ['icache_port', 'dcache_port', 'physmem_port']

  - *m5/src/cpu/mycpu/SConscript*: Edit the SConscript for your CPU
    model and add the relevant files that need to be built in here. Your
    file should only contain this code:

<!-- end list -->

    Import('*')

    if 'MyCPU' in env['CPU_MODELS']:
        Source('mycpu.cc')
        SimObject('MyCPU.py')
        TraceFlag('MyCPU')
        # make sure that SimpleCPU is part of the TraceFlag
        # Otherwise, DPRINTF may not work properly
        TraceFlag('SimpleCPU')
        Source('base.cc')
        SimObject('BaseSimpleCPU.py')

  - *m5/src/cpu/mycpu/SConsopts*: Edit the SConscript for your CPU model
    and add the relevant files that need to be built in here. Your file
    should only contain this code:

<!-- end list -->

    Import('*')

    all_cpu_list.append('MyCPU')
    default_cpus.append('MyCPU')

  - *m5/src/cpu/static_inst.hh*: Put a forward class declaration of
    your model in here

<!-- end list -->

    ...
    class CheckerCPU;
    class FastCPU;
    class AtomicSimpleCPU;
    class TimingSimpleCPU;
    class InorderCPU;
    class MyCPU;
    ...

  - *m5/src/cpu/cpu_models.py*: Add in CPU Model-specific information
    for the ISA Parser. The ISA Parser will use this when referring to
    the "Execution Context" for executing instructions. For instance,
    the AtomicSimpleCPU's instructions get all of their information from
    the actual CPU (since it's a 1 CPI machine). Thus, instructions only
    need to know the current state or "Execution Context" of the
    'AtomicSimpleCPU' object. However, the instructions in a O3CPU
    needed to know the register values (& other state) only known to
    that current instruction so it's "Execution Context" is the
    O3DynInst object. (check out the [ISA Description Language
    documentation](The_M5_ISA_description_language "wikilink") page for
    more details)

<!-- end list -->

    ...
    CpuModel('AtomicSimpleCPU', 'atomic_simple_cpu_exec.cc',
             '#include "cpu/simple/atomic.hh"',
             { 'CPU_exec_context': 'AtomicSimpleCPU' })
    CpuModel('MyCPU', 'mycpu_exec.cc',
             '#include "cpu/mycpu/mycpu.hh"',
             { 'CPU_exec_context': 'MyCPU' })
    ...

## Building MyCPU

Navigate to the M5 top-level directory and build your model:

    me@mymachine:~/m5/src/python/objects$cd ~/m5
    me@mymachine:~/m5$scons build/MIPS_SE/m5.debug CPU_MODELS=MyCPU
    scons: Reading SConscript files ...
    Checking for C header file Python.h... (cached) yes
    ...
    Building in /y/ksewell/research/m5-sim/newmem-clean/build/MIPS_SE
    Options file /y/ksewell/research/m5-sim/newmem-clean/build/options/MIPS_SE not found,
      using defaults in build_opts/MIPS_SE
    Compiling in MIPS_SE with MySQL support.
    scons: done reading SConscript files.
    scons: Building targets ...
    make_hh(["build/MIPS_SE/base/traceflags.hh"], ["build/MIPS_SE/base/traceflags.py"])
    Generating switch header build/MIPS_SE/arch/interrupts.hh
    Generating switch header build/MIPS_SE/arch/isa_traits.hh
    Defining FULL_SYSTEM as 0 in build/MIPS_SE/config/full_system.hh.
    Generating switch header build/MIPS_SE/arch/regfile.hh
    Generating switch header build/MIPS_SE/arch/types.hh
    Defining NO_FAST_ALLOC as 0 in build/MIPS_SE/config/no_fast_alloc.hh.
    ...
    g++ -o build/MIPS_SE/arch/mips/faults.do -c -pipe -fno-strict-aliasing -Wall -Wno-sign-compare -Werror -Wundef -ggdb3 -DTHE_ISA=MIPS_ISA -DDEBUG -DTRACING_ON=1 -Iext/dnet
    -I/usr/include/python2.4 -Ibuild/libelf/include -I/usr/include/mysql -Ibuild/MIPS_SE build/MIPS_SE/arch/mips/faults.cc
    g++ -o build/MIPS_SE/arch/mips/isa_traits.do -c -pipe -fno-strict-aliasing -Wall -Wno-sign-compare -Werror -Wundef -ggdb3 -DTHE_ISA=MIPS_ISA -DDEBUG -DTRACING_ON=1 -Iext/dnet
     -I/usr/include/python2.4 -Ibuild/libelf/include -I/usr/include/mysql -Ibuild/MIPS_SE build/MIPS_SE/arch/mips/isa_traits.cc
    g++ -o build/MIPS_SE/arch/mips/utility.do -c -pipe -fno-strict-aliasing -Wall -Wno-sign-compare -Werror -Wundef -ggdb3 -DTHE_ISA=MIPS_ISA -DDEBUG -DTRACING_ON=1 -Iext/dnet
    -I/usr/include/python2.4 -Ibuild/libelf/include -I/usr/include/mysql -Ibuild/MIPS_SE build/MIPS_SE/arch/mips/utility.cc
    ...
    cat build/MIPS_SE/m5.debug.bin build/MIPS_SE/m5py.zip > build/MIPS_SE/m5.debug
    chmod +x build/MIPS_SE/m5.debug
    scons: done building targets.

If you are compiling on a dual-core CPU, use this command-line to
speed-up the compilation:

    scons -j2 build/MIPS_SE/m5.debug CPU_MODELS=MyCPU

## Creating Configuration Scripts for MyCPU

Now that you have a M5 binary built for use with the MIPS Architecture
in a M5, MyCPU Model, you are almost ready to simulate. Note that the
standard M5 command line requires taht your provide at least a
configuration script for the M5 binary to use.

The easiest way to get up and running is to use the sample
Syscall-Emulation script: *configs/example/se.py*.

You'll note that line 9 of the se.py Python script imports the details
of what type of Simulation will be run (e.g. what CPU Model?) from the
Simulation.py file found here: m5/configs/common/Simulation.py. And
then, the Simulation.py file imports it's CPU Model options from the
Options.py file in the same directory. Edit those two files and your M5
binary will be ready to simulate.

  - *m5/configs/common/Options.py*: Add a option for your model in this
    file...

<!-- end list -->

    ...
    parser.add_option("--my_cpu", action="store_true", help="Use MyCPU Model")
    ...

  - *m5/configs/common/Simulation.py*: Edit the Simulation script to
    recognize your model as a CPU class. After your edits, the
    setCPUClass function should look like this:

<!-- end list -->

    ...
    def setCPUClass(options):

        atomic = False
        if options.timing:
            class TmpClass(TimingSimpleCPU): pass
        elif options.my_cpu:
            class TmpClass(MyCPU): pass
            atomic = True
        elif options.detailed:
            if not options.caches:
                print "O3 CPU must be used with caches"
                sys.exit(1)
            class TmpClass(DerivO3CPU): pass
        elif options.inorder:
            if not options.caches:
                print "InOrder CPU must be used with caches"
                sys.exit(1)
            class TmpClass(InOrderCPU): pass
        else:
            class TmpClass(AtomicSimpleCPU): pass
            atomic = True

        CPUClass = None
        test_mem_mode = 'atomic'

        if not atomic:
            if options.checkpoint_restore != None or options.fast_forward:
                CPUClass = TmpClass
                class TmpClass(AtomicSimpleCPU): pass
            else:
                test_mem_mode = 'timing'

        return (TmpClass, test_mem_mode, CPUClass)
    ...

## Testing MyCPU

Once you've edited the configuration scripts, you can run a M5
simulation from the top level directory with this command
    line:

    me@mymachine:~/m5$build/MIPS_SE/m5.debug configs/example/se.py --my_cpu --cmd=tests/test-progs/hello/bin/mips/linux/hello

NOTE: This binary path refers to the m5/tests directory.

## Summary

So the above "tutorial" showed you how to build your own CPU model in
M5. Now, it's up to you to customize the CPU Model as you like and do
your experiments\! Good luck\!
