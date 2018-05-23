---
title: "Introduction"
date: 2018-05-12T22:49:29-04:00
draft: false
weight: 3
---
gem5 is a modular discrete event driven computer system simulator platform. That means that:

1. gem5's components can be rearranged, parameterized, extended or replaced easily to suit your needs.
1. It simulates the passing of time as a series of discrete events.
1. Its intended use is to simulate one or more computer systems in various ways.
1. It's more than just a simulator; it's a simulator platform that lets you use as many of its premade components as you want to build up your own simulation system.

gem5 is written primarily in C++ and python and most components are provided under a BSD style license. It can simulate a complete system with devices and an operating system in full system mode (FS mode), or user space only programs where system services are provided directly by the simulator in syscall emulation mode (SE mode). There are varying levels of support for executing Alpha, ARM, MIPS, Power, SPARC, and 64 bit x86 binaries on CPU models including two simple single CPI models, an out of order model, and an in order pipelined model. A memory system can be flexibly built out of caches and crossbars. Recently the Ruby simulator has been integrated with gem5 to provide even more flexible memory system modeling.
There are many components and features not mentioned here, but from just this partial list it should be obvious that gem5 is a sophisticated and capable simulation platform. Even with all gem5 can do today, active development continues through the support of individuals and some companies, and new features are added and existing features improved on a regular basis.

#### Capabilities out of the box

gem5 is designed for use in computer architecture research, but if you're trying to research something new and novel it probably won't be able to evaluate your idea out of the box. If it could, that probably means someone has already evaluated a similar idea and published about it.
To get the most out of gem5, you'll most likely need to add new capabilities specific to your project's goals. gem5's modular design should help you make modifications without having to understand every part of the simulator.
As you add the new features you need, please consider contributing your changes back to gem5. That way others can take advantage of your hard work, and gem5 can become an even better simulator.


