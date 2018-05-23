---
title: "Visualizing w/Streamline"
date: 2018-05-13T18:51:37-04:00
draft: false
---

## What is Streamline?

ARM Streamline™ Performance Analyzer, is a component of ARM Development
Studio 5, DS-5™, intended to provide in-depth visibilities to the
software execution of the system and counters/statistics. Streamline can
be used to visualize a gem5 simulation, with Linux/Android process
information and gem5 statistics, with the help of some
post-processing.

![<File:gem5_ARM_Streamline_-Timeline.png>](gem5_ARM_Streamline_-Timeline.png
"File:gem5_ARM_Streamline_-Timeline.png")

## How do I get Streamline?

ARM provides a Community Edition of Streamline to all active members of
the gem5 community, as long as they agree to use it for analyzing gem5
outputs only and not use the license for commercial purposes.

Please visit [this
link](http://www.arm.com/products/tools/streamline-for-gem5.php) for
detailed instructions on obtaining Streamline.

## How do I view my gem5 run in Streamline?

**Note: The gem5/Streamline integration is currently only supported with
the ARM ISA.**

**1. Before running**

Currently Streamline works with Linux and Android guest OSes. The guest
kernel needs to have the m5struct.patch applied so that gem5 can access
certain information of threads/processes being simulated.

When running the simulation, the enable_context_switch_stats_dump
flag in LinuxArmSystem must be enabled. This will ensure that the gem5
statistics will be dumped at every context switch. Additionally, a
system.tasks.txt file will be dumped in the output directory, which
keeps track of the thread/process information.

**2. After running**

Run the post-processing script, util/streamline/m5stats2streamline.py,
to generate a Streamline project folder from gem5 stats. A
stat_config.ini file must be provided, which lists the gem5 stats to be
included. A couple of sample stat config files are included
(util/o3_stat_config.ini and util/atomic_stat_config.ini). Once the
.apc folder is generated, it can be viewed using
Streamline.

## More information

[Slides](http://www.gem5.org/wiki/images/9/9f/2012_12_01_gem5_workshop_Streamline.pdf)
from [the First Annual gem5 User Workshop](http://www.gem5.org/User_workshop_2012).

For any problem or comment, please email dam.sunwoo@arm.com.

