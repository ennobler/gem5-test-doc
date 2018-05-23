---
title: "Future Serialization"
date: 2018-05-13T18:51:37-04:00
draft: false
---

The current serialization system uses a .ini file format. Each section
refers to a specific object instance, and the values within a section
are the serialized data for that instance. The section heading is the
name() of the object and all hierarchy is essentially flattened.

During the serialization process, the file is directly written out as
objects are serialized, without any intermediate buffering. The
serialization proceeds by doing the global serialization then by
serializing each SimObject. Objects that are to be serialized are simply
passed the stream to which they are to output their information. This
process leads to several problems:

  - It is difficult to serialize user types that are not SimObjects.
      - On a related note, it is difficult to serialize a contained user
        type while already serializing another object. (Because we're
        directly writing the stream.)
  - Alternative serialization formats can't easily be supported
  - Improvements to the framework require changes all over the place.

Here are some proposed solutions.

  - General fixes
      - Keep a list of serializable objects that is Separate from
        SimObjects. This allows us to create non SimObject serializable
        objects.
      - We should probably support serializing python objects.

<!-- end list -->

  - On the output (serialization) side:
      - The serialize function should simply take a "Checkpoint" object
        as an input, not an ostream.
      - The various paramOut type functions should not be templated the
        way they are. They should be implemented like
        ostream::operator\<\< is. i.e. without a required template
        argument, arrayParamOut should be no different than paramOut,
        etc. By doing this, paramOut can be easily overloaded for new
        types and containers.
      - The Checkpoint object should just build a dictionary of a list
        of tuples, the outer dict is for each object, and the list of
        tuples are the serialized keys, values.
      - Python should access the dict and write out the data.
      - Support pointers to other sections (for sub objects)
  - On the input (unserialization) side:
      - Don't take both a Checkpoint \* and a section name. Instead take
        a CheckpointSection object which is a dict of the serialized
        data. CheckpointSection should have a Checkpoint \* in it.
      - Fix the paramIn functions as I've described fixing the paramOut
        functions.

Questions:

  - Do we want to keep the .ini format as the default? It could be a bit
    difficult to support user defined types this way.

