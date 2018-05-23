---
title: "Simulation Scripts"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

Simulation scripts control the configuration and execution of gem5
simulations. The gem5 simulator itself is basically passive; on invoking
gem5, it simply executes the user's simulation script, and performs
actions only when called by the script.

Simulation scripts are written in Python and executed by the Python
interpreter. Currently, the interpreter is linked into the gem5
executable, but for most purposes the script's execution should be
indistinguishable from invoking the Python interpreter directly.

A typical simulation script has two phases: a
[configuration](#Configuration "wikilink") phase, where the target
system is specified by constructing and interconnecting a hierarchy of
Python simulation objects; and a [simulation](#Simulation "wikilink")
phase, where the actual simulation takes place. Simulation scripts can
also define command-line [options](#Options "wikilink") which allow
users to control either or both of these phases.

## Configuration

The simulated system is built from a collection of simulator objects, or
SimObjects. The script describes the simulated system by describing the
SimObjects to be instantiated, their parameters, and their
relationships. Specifically, as the program contained in the script file
executes, it should create a hierarchy of Python objects that mirror the
SimObjects to be created for the simulation. The easiest way to get
started is to use an existing script file. Several examples are provided
in the configs directory.

### Python classes

gem5 provides a collection of Python object classes that correspond to
its C++ simulation object classes. These Python classes are defined in a
Python module called "m5.objects". The Python class definitions for
these objects can be found in .py files in src, typically in the same
directory as their C++ definitions.

The first step in specifying a SimObject is to instantiate a Python
object of the corresponding class. To make the Python classes visible,
the configuration file must first import the class definitions from the
m5 module as follows:

    from m5.objects import *

A Python object is instantiated by writing the class name (which, by our
convention, starts with an uppercase letter) followed by a pair of
parentheses. Thus the following code instantiates a SimpleCPU object and
assigns it to the Python variable cpu:

    cpu = SimpleCPU()

SimObject parameters are specified using Python attributes (Python
terminology for object fields or members). These attributes can be set
at any time using direct Python assignments or at instantiation time
using keywords inside the parentheses. The following instantiation:

    cpu = SimpleCPU(clock = '2GHz', width = 2)

is thus equivalent to:

    cpu = SimpleCPU()
    cpu.clock = '2GHz'
    cpu.width = 2

Parameter assignments are partially validated at the time the assignment
is performed. The attribute name (e.g., clock) must be a defined
parameter for the SimObject class, and the right-hand side of the
assignment must be of (or convertible to) the correct type for that
parameter. The m5 module defines a large number of domain-specific
string-to-value conversions, allowing expressions such as '2GHz' and
'64KB' for clock rates and memory sizes, respectively. A complete list
of valid parameter types and units can be found here: [Python Parameter
Types](Python_Parameter_Types "wikilink").

The complete list of parameters for a given SimObject class (along with
their types, default values, and brief descriptions) can be found by
looking at its Python class definition located somewhere in the src
directory. An attribute labeled Param.X defines a parameter of type X,
while an attribute labeled VectorParam.X defines a parameter requiring a
vector (Python list) whose elements must be of type X. Note that
parameters are inherited: for example, the clock parameter for the
SimpleCPU object above is not specified in the SimpleCPU class
definition in SimpleCPU.py, but is inherited from SimpleCPU's Python
base class BaseCPU (specified in BaseCPU.py).

Connections among SimObjects are formed by using a reference to one
SimObject as a parameter value in the construction of a second
SimObject. For example, a CPU's instruction and data caches are
specified by naming cache SimObjects as the values of the CPU's icache
and dcache parameters, respectively.

### The configuration hierarchy

To simplify the description of large systems, the overall simulation
target specification is organized as a hierarchy (tree). Each node in
the tree is a SimObject instance. Even if a SimObject is instantiated in
Python, it will not be constructed for the simulation unless it is part
of this hierarchy. The program must create a special object root of
class Root to identify the root of the hierarchy. When the configuration
program completes execution, the tree rooted at root is walked
recursively to identify objects to construct. Children are added to
SimObjects using the same syntax as setting parameters, i.e., by
assigning to Python object attributes. The SimpleCPU object created
above can be instantiated by making it a child of the root object as
follows:

    root = Root()
    root.cpu = cpu

As with parameters, children can be assigned at instantiation time using
keyword assignment within parentheses. As a result, the instantiation of
the CPU and attaching it to the root node can be done in a single line:

    root = Root(cpu = SimpleCPU(clock = '2GHz', width = 2))

SimObjects may also become children when they are assigned to a
parameter of another SimObject. For example, creating a cache object and
assigning it to a CPU's icache or dcache parameter makes the cache
object a child of the CPU object in the configuration hierarchy. This
effect only occurs for SimObjects that are not in the hierarchy; a
SimObject that is already part of the hierarchy is not re-parented when
it is assigned to another SimObject's parameter.

The configuration hierarchy determines the final name of each
instantiated object. The name is formed from the path from root to the
particular object (not including root itself), joining elements with
'.'. For example, consider the following configuration:

    my_cpu = SimpleCPU(clock = '2GHz', width = 2)
    my_cpu.icache = BaseCache(size = '32KB', assoc = 2)
    my_cpu.dcache = BaseCache(size = '64KB', assoc = 2)
    my_system = LinuxSystem(cpu = my_cpu)
    root = Root(system = my_system)

In this case, the resulting SimObjects will have the internal names
system, system.cpu, system.cpu.icache, and system.cpu.dcache. These
names will be used in statistics output, etc. The names my_cpu and
my_system are simply Python variables; they can be used within Python
to set attributes, add children, etc., but they are not visible to the
C++ portion of the simulation. Note that a Python object can be accessed
using its configuration hierarchy path from within Python by prepending
root. .

A child attribute can also accept a vector of SimObjects. As with
vector-valued parameters, these vectors are expressed as Python lists,
for example, `system.cpu = [ SimpleCPU(), SimpleCPU() ]`. In Python,
these objects can be accessed using standard list index notation (e.g.,
`system.cpu[0]`). The internal names for the objects are formed by
directly appending the index to the attribute name (e.g.,
`system.cpu0`).

In detail, the semantics of assigning to SimObject attributes are as
follows:

If the attribute name identifies one of the SimObject's formal
parameters, then the value on the right-hand side is converted to the
parameter's type. An error is raised if this conversion cannot be
performed. If the value is a SimObject that is not associated with the
configuration hierarchy, that SimObject also becomes a child of the
SimObject whose attribute is being assigned. The parameter name is used
as the final element in the assigned SimObject's name.

If the attribute name does not correspond to a formal parameter and the
right-hand value is a SimObject or a list of SimObjects, those
SimObject(s) become children of the SimObject whose attribute is being
assigned. The attribute name is used as the final element in the
assigned SimObject's name.

If the attribute name does not correspond to a formal parameter and the
right-hand value is not a SimObject, an error is raised.

### Inheritance and late binding

SimObject instances inherit both parameters and values from the classes
they instantiate. A key feature of the configuration system is that
value inheritance is largely late binding, that is, values are
propagated to instances when the hierarchy is instantiated, not when the
instance is created. As a result, a value can be set on a class
parameter after instances have been created, and the instances will
receive the more recent parameter value (as long as the parameter has
not been explicitly overridden on those instances). The following
example demonstrates this behavior:

    # Instantiate some CPUs.
    scpu1 = SimpleCPU()
    scpu2 = SimpleCPU()
    fcpu1 = FullCPU()
    fcpu2 = FullCPU()

    # Since BaseCPU is a common base class for SimpleCPU and FullCPU, the
    # following statement will cause all of the above CPUs to have a clock
    # rate of 1GHz.
    BaseCPU.clock = '1GHz'

    # The following statement sets the width of both scpu1 and scpu2 to 4.
    SimpleCPU.width = 4

    # We can override the clock rate for a specific CPU.  Note that this
    # assignment will have the same effect whether it is before or after
    # the BaseCPU.clock assignment above.
    fcpu2.clock = '2GHz'

### Subclassing

Users can define new SimObject classes by deriving from existing gem5
classes. This feature can be useful for providing classes with differing
sets of parameter values. These subclasses are defined using standard
Python class syntax:

    class CrazyFastCPU(FullCPU):
        rob_size = 10000
        width = 100
        clock = '10GHz'

Users can also subclass or instantiate the SimObject class directly,
e.g., `obj = SimObject()`. These Python objects will not generate C++
SimObjects, but can be assigned children. They can be useful to create
internal nodes in the configuration hierarchy that represent collections
of SimObjects but do not correspond to C++ SimObjects themselves.

### Relative references

In many situations, SimObject parameters have obvious default values
that cannot be explicitly named in the general case. For example, many
I/O devices need a pointer to the enclosing system's physical memory
object or to the enclosing system object itself. Similarly, a cache's
default latency might be expressed most conveniently in terms of the
clock period of the attached CPU. However, the path names of those
objects will vary from configuration to configuration. gem5's
configuration system solves this problem by providing relative reference
objects. These are "proxy" objects that stand in for real objects and
are resolved only after the entire hierarchy is constructed.

The m5 module provides two relative reference objects: Self and Parent.
An attribute reference relative to Self resolves to the referencing
object, while Parent is resolved by iteratively traversing up the
hierarchy (towards root), starting at the parent of the referencing
object, until a suitable match is found. A key feature of these objects
is that resolution is relative to the final referencing object instance,
not where the assignment is performed. Thus they can be assigned as
default values to parameters in a SimObject class definition, and will
be resolved independently for each instance that derives from that
class.

For example, it is convenient to set the default clock speed of a CPU
object to be the clock speed of the enclosing system; thus in a
homogenous multiprocessor there is no need to explicitly set the clock
rate on each CPU. We achieve this by setting the default value for the
CPU's clock parameter to be Parent.clock. During the final instantiation
phase, an access to the CPU's clock parameter will be resolved by
iterating up the hierarchy, starting at the CPU's parent, until an
object with a clock parameter is found. The value of this parameter
(which will be recursively resolved if it is also a relative reference)
will be assigned to the CPU's clock parameter.

For further flexibility, Self and Parent can take a special attribute,
any, which instructs the resolution mechanism to find any value of the
appropriate type, either a hierarchy node itself or a parameter of a
node. The most common usage of this feature is to use Parent.any for a
SimObject-valued parameter. For example, many devices use Parent.any as
a default value to locate the enclosing system object or its physical
memory object. To avoid ambiguity, an error will be raised if Parent.any
could resolve to multiple values at the same level of the hierarchy.

## Simulation

Once the simulation script has created the desired configuration in
Python, it can move ahead with the actual simulation. At this point, the
Python script must start interacting with gem5's C++ simulation core.
These interactions occur via Python function calls provided by the m5
module that get translated into C++ function calls. In order to access
these functions, the simulation script must import the m5 package as
follows:

`import m5`

This line is typically placed at the top of the script, along with the
"`from m5.objects import *`" line mentioned above.

Instantiating the C++ object hierarchy is as simple as calling the
`instantiate()` function and passing it the root object of the
hierarchy, by convention called '`root`':

`m5.instantiate(root)`

Once the C++ object hierarchy has been instantiated, actual simulation
can begin. The `simulate()` function invokes the C++ event loop. By
default, this function will simulate forever, or until some other factor
causes the simulation loop to exit (such as the target program calling
`exit()` or a CPU reaching the `max_insts_any_thread` limit). If the
simulate function is passed a positive integer argument, it will
simulate at most that number of additional ticks, but may exit sooner if
another cause arises first. In any case, the `simulate()` function will
return an event object that represents the reason for exiting. The
object can be queried via its `getCause()` method for a string
explaining that reason. A very simple yet user-friendly simulation
script may end in the following two lines:

`exit_event = m5.simulate()`
`print 'Exiting @ tick', m5.curTick(), 'because', exit_event.getCause()`

This example also uses the m5 function `curTick()`, which returns the
current simulation tick value (the value of the C++ variable `curTick`).

The simulation loop can be called multiple times, each time simulating
until some reason causes a return to Python. For example, a script could
extend the previous example to print a progress indication every million
ticks as follows:

`while 1:`
`    exit_event = m5.simulate(1000000)`
`    if exit_event != 'simulate() limit reached':`
`        break`
`    print 'Simulation reached tick', m5.curTick()`
`print 'Exiting @ tick', m5.curTick(), 'because', exit_event.getCause()`

## Options

The options given on the command line after the script name (see
[Running gem5](Running_gem5 "wikilink")) are passed to the simulation
script in the same manner that command-line arguments are passed to
standard Python scripts (i.e. via `sys.argv`). These options allow a
single script to be configurable in user-defined ways. Our example
scripts in configs/example use script options to select the CPU model
(simple vs. detailed) used within an otherwise similar configuration.

Because options are passed to the script in the standard Python fashion,
script files can use standard Python tools to parse options. We
generally use the
[optparse](http://docs.python.org/lib/module-optparse.html) module from
the Python standard library.

Here is an example snippet of a simulation script that does option
parsing using optparse, then uses the parsed option flags to select a
CPU model:

    parser = optparse.OptionParser()

    parser.add_option("-d", "--detailed", action="store_true")
    parser.add_option("-t", "--timing", action="store_true")

    (options, args) = parser.parse_args()

    if options.timing:
        cpu = TimingSimpleCPU()
    elif options.detailed:
        cpu = DetailedO3CPU()
    else:
        cpu = AtomicSimpleCPU()

A secondary benefit of using the optparse module is that all available
script options can be listed by using the "-h" flag, e.g.:

`gem5.opt `

<script>

\-h

just as `gem5.opt -h` lists all available gem5 options (see [Running
gem5](Running_gem5 "wikilink")).
