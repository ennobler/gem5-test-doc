---
title: "Alternate Sources"
date: 2018-05-12T21:16:55-04:00
draft: false
---

The latest gem5 source code is
available via our Git repository host at
<https://gem5.googlesource.com>. It is **strongly** recommend that you
get a copy of gem5 by using git. You can get more info about the
repository and git [here](Repository "wikilink"). In additional to the
main git repository, there is a mirror on
[GitHub](https://github.com/gem5/gem5) (we can't currently accept pull
requests on GitHub) and a [Mercurial mirror](http://repo.gem5.org). Keep
in mind that the mirrors are read only. New code can only be submitted
to the main git repository.

#### Mercurial mirror

**NOTE:** The Mercurial mirror is read-only.

  - Install Mercurial if you don't have it already. This is available in
    the mercurial package on Ubuntu and OSX Brew.

<!-- end list -->

  - Clone the repository: `hg clone `<http://repo.gem5.org/gem5>

<!-- end list -->

  - After you clone the repository you can update it by typing `hg pull`
    and `hg update`.

#### TAR dumps

If you want to download gem5 without installing Mercurial, you can get a
tarball. But it will be more difficult to merge in changes when you need
to update to new version. Tagged stable versions can be downloaded from
[GitHub](https://github.com/gem5/gem5/releases).



