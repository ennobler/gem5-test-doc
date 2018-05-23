---
title: "Doxygen Code Documentation"
date: 2018-05-13T18:51:37-04:00
draft: false
---

## Introduction

Doxygen allows users to quickly create documentation for our code by
extracting the relavent information from the code and comments. It is
able to document all the code structures including classes, namespaces,
files, members, defines, etc. Most of these are quite simple to
document, you only need to place a special documentation block before
the declaration. The Doxygen documentation within gem5 is processed
every night and the following web pages are generated:
[Doxygen](http://www.m5sim.org/docs/index.html)

## Using Doxygen

The special documentation blocks take the form of a javadoc style
comment. A javadoc comment is a C style comment with 2 \*'s at the
start, like this:

    /**
     * ...documentation...
     */

The intermediate asterisks are optional, but please use them to clearly
delineate the documentation comments.

The documentation within these blocks is made up of at least a brief
description of the documented structure, that can be followed by a more
detailed description and other documentation. The brief description is
the first sentence of the comment. It ends with a period followed by
white space or a new line. For example:

    /**
     * This is the brief description. This is the start of the detailed
     * description. Detailed Description continued.
     */

If you need to have a period in the brief description, follow it with a
backslash followed by a space.

    /**
     * e.g.\ This is a brief description with an internal period.
     */

Blank lines within these comments are interpreted as paragraph breaks to
help you make the documentation more readble.

## Special commands

Placing these comments before the declaration works in most cases. For
files however, you need to specify that you are documenting the file. To
do this you use the @file special command. To document the file that you
are currently in you just need to use the command followed by your
comments. To comment a separate file (we shouldn't have to do this) you
can supply the name directly after the file command. There are some
other special commands we will be using quite often. To document
functions we will use @param and @return or @retval to document the
parameters and the return value. @param takes the name of the paramter
and its description. @return just describes the return value, while
@retval adds a name to it. To specify pre and post conditions you can
use @pre and @post.

Some other useful commands are @todo and @sa. @todo allows you to place
reminders of things to fix/implement and associate them with a specific
class or member/function. @sa lets you place references to another piece
of documentation (class, member, etc.). This can be useful to provide
links to code that would be helpful in understanding the code being
documented.

## Example of Simple Documentation

Here is a simple header file with doxygen comments added.

    /**
     * @file
     * Contains an example of documentation style.
     */

    #include <vector>

    /**
     * Adds two numbers together.
     */
    #define DUMMY(a,b) (a+b)

    /**
     * A simple class description. This class does really great things in detail.
     *
     * @todo Update to new statistics model.
     */
    class foo
    {
      /** This variable stores blah, which does foo and has invariants x,y,z
             @warning never set this to 0
             @invariant foo
        */
       int myVar;

     /**
      * This function does something.
      * @param a The number of times to do it.
      * @param b The thing to do it to.
      * @return The number of times it was done.
      *
      * @sa DUMMY
      */
     int bar(int a, long b);


     /**
      * A function that does bar.
      * @retval true if there is a problem, false otherwise.
      */
     bool manchu();

    };

## Grouping

Doxygen also allows for groups of classes and member (or other groups)
to be declared. We can use these to create a listing of all
statistics/global variables. Or just to comment about the memory
hierarchy as a whole. You define a group using @defgroup and then add to
it using @ingroup or @addgroup. For example:

    /**
     * @defgroup statistics Statistics group
     */

    /**
      * @defgroup substat1 Statistitics subgroup
      * @ingroup statistics
      */

    /**
     *  A simple class.
     */
    class foo
    {
      /**
       * Collects data about blah.
       * @ingroup statistics
       */
      Stat stat1;

      /**
       * Collects data about the rate of blah.
       * @ingroup statistics
       */
      Stat stat2;

      /**
       * Collects data about flotsam.
       * @ingroup statistics
       */
      Stat stat3;

      /**
       * Collects data about jetsam.
       * @ingroup substat1
       */
      Stat stat4;

    };

This places stat1-3 in the statistics group and stat4 in the subgroup.
There is a shorthand method to place objects in groups. You can use @{
and @} to mark the start and end of group inclusion. The example above
can be rewritten as:

    /**
     * @defgroup statistics Statistics group
     */

    /**
      * @defgroup substat1 Statistitics subgroup
      * @ingroup statistics
      */

    /**
     *  A simple class.
     */
    class foo
    {
      /**
       * @ingroup statistics
       * @{
       */

      /** Collects data about blah.*/
      Stat stat1;
      /** Collects data about the rate of blah. */
      Stat stat2;
      /** Collects data about flotsam.*/
      Stat stat3;

      /** @} */

      /**
       * Collects data about jetsam.
       * @ingroup substat1
       */
      Stat stat4;

    };

It remains to be seen what groups we can come up with.

## Other features

Not sure what other doxygen features we want to use.

