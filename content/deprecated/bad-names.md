---
title: "Bad names"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---


We may have a couple of classes, variables, and functions that aren't
named too well. Some are too long, aren't descriptive, etc. List the
things that you don't like here, and try to give some suggestions for
alternates.

## Function names

  - setMiscRegWithEffect - setMiscReg (will have to rename current
    setMiscReg to setMiscRegNoEffect or something similar); setMisc
  - readMiscRegWithEffect - same as above

## Class names

## Variable names

## Command to replace string

`find . -name "*.cc" -o -name "*.hh" -o -name "*.isa" | grep -v SCCS |
xargs perl -pi -e "s/oldstring/newstring/"`
