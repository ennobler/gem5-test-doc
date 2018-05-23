---
title: "Adding Functionality"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 100
---
{{% notice warning %}}
This section needs to be updated; no mercurial; should reference Contribuing
{{% /notice %}}


If you find the need to modify or extend gem5, you may be tempted to
just start editing files in your local gem5 repository. While this
approach will work initially, it will cause problems if/when you decide
to update your copy of gem5 with changes that have been made since you
originally cloned the repository. It's very likely that you will want to
update your code to get bug fixes and new features. Thus we **strongly**
advise you to follow one of the following methods. It will save you a
great deal of time in the future and allow you to take advantage of new
gem5 versions without the error prone process of manually diffing and
patching versions.

There are two recommend ways to add your own functionality to gem5 and
retain the ability to revision control your own code: Mercurial Queues
(MQ) and the EXTRAS feature of our build system.

Mercurial Queues is the most powerful option, as it tracks changes you
make to the existing gem5 code as well as files you may add to the
source tree. It's also the recommended path for developing changes that
you contribute back to the public code base (see [Submitting
Contributions](Submitting_Contributions "wikilink")). For more
information, see [Managing Local Changes with Mercurial
Queues](Managing_Local_Changes_with_Mercurial_Queues "wikilink").

The [EXTRAS](Extras "wikilink") option is more limited, in that it only
allows you to compile additional files in to the gem5 code base, rather
than changing or overriding existing files. However, the code compiled
with EXTRAS is completely decoupled from the gem5 repository, and thus
can be managed separately (e.g., in a different Mercurial repository, or
using a completely different revision control system). EXTRAS can also
be used to incorporate code that can't be distributed with gem5 due to
licensing issues (e.g., the "encumbered"
[repository](repository "wikilink")). Often users end up using EXTRAS to
incorporate new SimObject models while they concurrently manage a
(ideally much smaller) set of changes using MQ. See the
[EXTRAS](Extras "wikilink") page for more details.
