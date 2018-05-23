---
title: "High-level flow"
date: 2018-05-12T21:26:55-04:00
draft: false
weight: 5
---
<pre>

    +-------------+
    | Make change |
    +------+------+
           |
           |
           v
    +------+------+
    | Post review |
    +------+------+
           |
           v
    +--------+---------+
    | Wait for reviews | <--------+
    +--------+---------+          |
           |                      |
           |                      |
           v                      |
      +----+----+   No     +------+------+
      |Reviewers+--------->+ Update code |
      |happy?   |          +------+------+
      +----+----+                 ^
           |                      |
           | Yes                  |
           v                      |
      +----+-----+   No           |
      |Maintainer+----------------+
      |happy?    |
      +----+-----+
           |
           | Yes
           v
    +------+------+
    | Submit code |
    +-------------+

</pre>
After creating your change to gem5, you can post a review on our Gerrit
code-review site: https://gem5-review.googlesource.com. Before being able to
submit your code to the mainline of gem5, the code is reviewed by others in the
community. Additionally, the maintainer for that part of the code must sign off
on it.


