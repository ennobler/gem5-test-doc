---
title: "ISA code parsing"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

To a large extent, the power and flexibility of the ISA description
mechanism stem from the fact that the mapping from a brief instruction
definition provided in the decode block to the resulting C++ code is
performed in a general-purpose programming language (Python). (This
function is performed by the "instruction format" definition described
above in [Format
definitions](The_M5_ISA_description_language#Format_definitions "wikilink").)
Technically, the ISA description language allows any arbitrary Python
code to perform this mapping. However, the parser provides a library of
Python classes and functions designed to automate the process of
deducing an instruction's characteristics from a brief description of
its operation, and generating the strings required to populate
declaration and decode templates. This library represents roughly half
of the code in isa_parser.py.

Instruction behaviors are described using C++ with two extensions:
bitfield operators and operand type qualifiers. To avoid building a full
C++ parser into the ISA description system (or conversely constraining
the C++ that could be used for instruction descriptions), these
extensions are implemented using regular expression matching and
substitution. As a result, there are some syntactic constraints on their
usage. The following two sections discuss these extensions in turn. The
third section discusses operand parsing, the technique by which the
parser automatically infers most instruction characteristics. The final
two sections discuss the Python classes through which instruction
formats interact with the library: `CodeBlock`, which analyzes and
encapsulates instruction description code; and the instruction object
parameter class, `InstObjParams`, which encapsulates the full set of
parameters to be substituted into a template.

### Bitfield operators

Simple bitfield extraction can be performed on rvalues using the `<:>`
postfix operator. Bit numbering matches that used in global bitfield
definitions (see [Bitfield
definitions](The_M5_ISA_description_language#Bitfield_definitions "wikilink")).
For example, `Ra<7:0>` extracts the low 8 bits of register `Ra`.
Single-bit fields can be specified by eliminating the latter operand,
e.g. `Rb<31:>`. Unlike in global bitfield definitions, the colon cannot
be eliminated, as it becomes too difficult to distinguish bitfield
operators from template arguments. In addition, the bit index parameters
must be either identifiers or integer constants; expressions are not
allowed. The bit operator will apply either to the syntactic token on
its left, or, if that token is a closing parenthesis, to the
parenthesized expression.

### Operand type qualifiers

The effective type of an instruction operand (e.g., a register) may be
specified by appending a period and a type qualifier to the operand
name. The list of type qualifiers is architecture-specific; the `def
operand_types` statement in the ISA description is used to specify it.
The specification is in the form of a Python dictionary which maps a
type extension to type name. For example, the Alpha ISA definition is as
follows:

    def operand_types {{
        'sb' : 'int8_t',
        'ub' : 'uint8_t',
        'sw' : 'int16_t',
        'uw' : 'uint16_t',
        'sl' : 'int32_t',
        'ul' : 'uint32_t',
        'sq' : 'int64_t',
        'uq' : 'uint64_t',
        'sf' : 'float',
        'df' : 'double'
    }};

Thus the Alpha 32-bit add instruction addl could be defined as:

    Rc.sl = Ra.sl + Rb.sl;

The operations are performed using the types specified; the result will
be converted from the specified type to the appropriate register value
(in this case by sign-extending the 32-bit result to 64 bits, since
Alpha integer registers are 64 bits in size).

Type qualifiers are allowed only on recognized instruction operands (see
[\#Instruction operands](#Instruction_operands "wikilink")).

### Instruction operands

Most of the automation provided by the parser is based on its
recognition of the operands used in the instruction definition code.
Most relevant instruction characteristics can be inferred from the
operands: floating-point vs. integer instructions can be recognized by
the registers used, an instruction that reads from a memory location is
a load, etc. In combination with the bitfield operands and type
qualifiers described above, most instructions can be described in a
single line of code. In addition, most of the differences between
simulator CPU models lies in the operand access mechanisms; by
generating the code for these accesses automatically, a single
description suffices for a variety of situations.

The ISA description provides a list of recognized instruction operands
and their characteristics via the `def operands` statement. This
statement specifies a Python dictionary that maps operand strings to a
five-element tuple. The elements of the tuple specify the operand as
follows:

1.  the operand class, which must be one of the strings "IntReg",
    "FloatReg", "Mem", "NPC", or "ControlReg", indicating an integer
    register, floating-point register, memory location, the next program
    counter (NPC), or a control register, respectively.
2.  the default type of the operand (an extension string defined in the
    `def operand_types` block),
3.  a specifier indicating how specific instances of the operand are
    decoded (e.g., a bitfield name),
4.  a string or triple of strings indicating the instruction flags that
    can be inferred when the operand is used, and
5.  a sort priority used to control the order of operands in
    disassembly.

For example, a simplified subset of the Alpha ISA operand traits map is
as follows:

    def operands {{
        'Ra': ('IntReg', 'uq', 'RA', 'IsInteger', 1),
        'Rb': ('IntReg', 'uq', 'RB', 'IsInteger', 2),
        'Rc': ('IntReg', 'uq', 'RC', 'IsInteger', 3),
        'Fa': ('FloatReg', 'df', 'FA', 'IsFloating', 1),
        'Fb': ('FloatReg', 'df', 'FB', 'IsFloating', 2),
        'Fc': ('FloatReg', 'df', 'FC', 'IsFloating', 3),
        'Mem': ('Mem', 'uq', None, ('IsMemRef', 'IsLoad', 'IsStore'), 4),
        'NPC': ('NPC', 'uq', None, ( None, None, 'IsControl'), 4)
    }};

The operand named `Ra` is an integer register, default type `uq`
(unsigned quadword), uses the `RA` bitfield from the instruction,
implies the `IsInteger` instruction flag, and has a sort priority of 1
(placing it first in any list of operands).

For the instrucion flag element, a single string (such as `'IsInteger'`
implies an unconditionally inferred instruction flag. If the flag
operand is a triple, the first element is unconditional, the second is
inferred when the operand is a source, and the third when it is a
destination. Thus the `('IsMemRef', 'IsLoad', 'IsStore')` element for
memory references indicates that any instruction with a memory operand
is marked as a memory reference. In addition, if the memory operand is a
source, the instruction is marked as a load, while if the operand is a
destination, the instruction is marked a store. Similarly, the `(None,
None, 'IsControl')` tuple for the NPC operand indicates that any
instruction that writes to the NPC is a control instruction, but
instructions which merely reference NPC as a source do not receive any
default flags.

Note that description code parsing uses regular expressions, which
limits the ability of the parser to infer the nature of a partciular
operand. In particular, destination operands are distinguished from
source operands solely by testing whether the operand appears on the
left-hand side of an assignment operator (`=`). Destination operands
that are assigned to in a different fashion, e.g. by being passed by
reference to other functions, must still appear on the left-hand side of
an assignment to be properly recognized as destinations. The parser also
does not recognize C compound assignments, e.g., `+=`. If an operand is
both a source and a destination, it must appear on both the left- and
right-hand sides of `=`.

Another limitation of regular-expression-based code parsing is that
control flow in the code block is not recognized. Combined with the
details of how register updates are performed in the CPU models, this
means that destinations cannot be updated conditionally. If a particular
register is recognized as a destination register, that register will
always be updated at the end of the `execute()` method, and thus the
code must assign a valid value to that register along each possible code
path within the block.

### The CodeBlock class

An instruction format requests processing of a string containing
instruction description code by passing the string to the CodeBlock
constructor. The constructor performs all of the needed analysis and
processing, storing the results in the returned object. Among the
CodeBlock fields are:

  - `orig_code`: the original code string.
  - `code`: a processed string containing legal C++ code, derived from
    the original code by substituting in the bitfield operators and
    munging operand type qualifiers (s/\\./_/) to make valid C++
    identifiers.
  - `constructor`: code for the constructor of an instruction object,
    initializing various C++ object fields including the number of
    operands and the register indices of the operands.
  - `exec_decl`: code to declare the C++ variables corresponding to the
    operands, for use in an execution emulation function.
  - `*_rd`: code to read the actual operand values into the
    corresponding C++ variables for source operands. The first part of
    the name indicates the relevant CPU model (currently simple and dtld
    are supported).
  - `*_wb`: code to write the C++ variable contents back to the
    appropriate register or memory location. Again, the first part of
    the name reflects the CPU model.
  - `*_mem_rd`, `*_nonmem_rd`, `*_mem_wb`, `*_nonmem_wb`: as above, but
    with memory and non-memory operands segregated.
  - `flags`: the set of instruction flags implied by the operands.
  - `op_class`: a basic guess at the instruction's operation class (see
    OpClass) based on the operand types alone.

### The InstObjParams class

Instances of the InstObjParams class encapsulate all of the parameters
needed to substitute into a code template, to be used as the argument to
a template's `subst()` method (see Template definitions).

    class InstObjParams(object):
        def __init___(self, parser,
                      mem, class_name, base_class = '',
                      snippets = {}, opt_args = []):

The first three constructor arguments populate the object's `mnemonic`,
`class_name`, and (optionally) `base_class` members. The fourth
(optional) argument is a CodeBlock object; all of the members of the
provided CodeBlock object are copied to the new object, making them
accessible for template substitution. Any remaining arguments are
interpreted as either additional instruction flags (appended to the
`flags` list inherited from the CodeBlock argument, if any), or as an
operation class (overriding any `op_class` from the CodeBlock).

