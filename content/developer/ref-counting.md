---
title: "Refcounted Pointers"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

I'm putting this in a wiki because I keep asking Nate and I keep
forgetting his answer.

Question: Which STL structures might have problems with calling ref
counted pointer destructors? Or is it just certain operations like
clear()?

Answer: It's with operations like clear. There may be references on
them, or when you remove an element from a datastructure like a deque or
a queue. If they're implemented in terms of vector, they don't get freed
until they're reallocated. Another problem is if you're using a special
memory allocator that doesn't destroy your object.

These problems affect classes such as the hash_map, which probably uses
a vector as its data structure. Keep this in mind when calling erase()
or clear() on such structures, as you might have to manually set the Ref
counted pointer to NULL prior to erasing/clearing, or else they will be
sitting around taking up resources for a long time.

