---
title: "Mercurial Queues"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---
## Repository Management Problem

gem5 users typically opt to freeze their repository at a particular
changeset when starting a new research project. This approach has
several downsides:

  - It discourages users from contributing back any useful changes they
    may develop.
  - If a useful change is added upstream, it's a long, tedious process
    to update.

If a user chooses to keep their local repository up-to-date with the
source tree they typically use [named
branches](http://mercurial.selenic.com/wiki/Branch) and merge any
upstream changes into their branches. This approach also has its
downsides:

  - If any local change needs to be updated, it requires a separate
    commit.
  - If you have several small, unrelated changes, separate branches must
    be maintained.
  - Upstream changes must be merged into the local branches.

A tool that overcomes these problems is the [mercurial
queue](http://mercurial.selenic.com/wiki/MqExtension) extension.

## Mercurial Queues

The mercurial queue extension is a powerful tool that allows you to:

  - Manage small changes easily as a set of well-defined patches.
  - Edit previous patches without having a new commit.
  - Keep your local changes cleanly separated from upstream changes.
  - Prevent changes from being recorded in the project history until
    they are ready.

This guide will give a brief overview of the basic functionality of
mercurial queues, which should be enough information to enable you to
effectively manage your local changes and allow you to contribute them
to the reviewboard if you choose to do so. However, there are many more
advanced uses of mercurial queues that may be beneficial. See also the
[MQ tutorial on the Mercurial
wiki](http://mercurial.selenic.com/wiki/MqTutorial), the [Mozilla
developer page on
MQ](https://developer.mozilla.org/en-US/docs/Mercurial_Queues), and the
chapters on ["Managing change with Mercurial
Queues"](http://hgbook.red-bean.com/read/managing-change-with-mercurial-queues.html)
and ["Advanced uses of Mercurial
Queues"](http://hgbook.red-bean.com/read/advanced-uses-of-mercurial-queues.html)
from the [Mercurial book](http://hgbook.red-bean.com/read).

### Basic MQ commands

##### Help Command

  - `hg help mq` --- Gives a list of mercurial queue commands and a
    brief description of each.

##### Creating and handling patches

  - `hg qnew change1.patch -m "commit message"` --- Create a new patch
    named "change1.patch" with a commit message.
  - `hg qpop` --- Pop topmost patch off the queue.
  - `hg qpush` --- Push next patch in the series onto the queue.
  - `hg qrefresh` --- Add any local changes to the topmost patch.
  - `hg qfinish` --- Remove patch from the queue and make a permanent
    part of the repo history.

##### Checking the status of patches in the queue

  - `hg qapplied` --- List all applied patches in the queue.
  - `hg qseries` --- List all patches in the current series (this
    includes even patches that aren't applied).
  - `hg qdiff` --- Display the diff for the applied patch at the top of
    the queue
  - `ht qtop` --- List the patch at the top of the queue.

##### Adding patches from other queues

  - `hg qimport -e pre_existing.patch` --- Adds a pre-existing patch
    called "pre_existing.patch" to the local queue.

### Advanced mercurial queue usage

Here will give a list of some of the advanced uses of mercurial queues,
and provide pointers to more in-depth information about them.

  - **Queue Guards** - Guards will allow you to manage patches by
    placing "guards" on them, i.e., you may perform actions on a
    specific set of patches based on the guard(s) placed on them. See
    this
    [guide](http://hgbook.red-bean.com/read/advanced-uses-of-mercurial-queues.html)
    for more information.

<!-- end list -->

  - **Multiple Queues** - You can maintain multiple patch queues, this
    could be useful for grouping sets of related patches together, while
    keeping them separate from other queues. More info can be found
    [here](http://stevelosh.com/blog/2010/08/a-git-users-guide-to-mercurial-queues/#multiple-patch-queues).

<!-- end list -->

  - **Versioning Your Patch Queues** - You can even maintain the changes
    to your patches in your repository. This allows you to keep track of
    the change within your patches, use hg push/pull to share the
    patches with others, etc. More info about this feature is
    [here](http://stevelosh.com/blog/2010/08/a-git-users-guide-to-mercurial-queues/#versioned-patch-queues).

## Example Mercurial Queue Use

### Enable the MQ extension

To enable the mercurial queue extension, simply add the following to
your **.hgrc** file:

`[extensions]`
`hgext.mq =`

### Simple workflow with MQs

Here is a simple example outlining basic MQ usage:

    # clone a clean copy of gem5
    hg clone http://repo.gem5.org/gem5

    # initialize a new mercurial queue
    cd ./gem5
    hg init --mq

    # make some local changes and turn them into a patch
    hg qnew change1.patch -m "cpu: made some changes to the cpu model"

    # we have some more changes that we want to turn into a separate patch
    hg qnew change2.patch -m "cache: made some changes to the cache"

    # now you want to make some more changes and include them in change1
    # make sure change1 is at the top of the queue
    hg qtop

    >>> change2.patch

    # it's not, so we have to pop change2 off the queue
    hg qpop
    hg qtop

    >>> change1.patch

    # now it's the top patch. make the necessary changes and update
    hg qrefresh

    # re-apply change2
    hg qpush

    # let's check that all of our patches are applied
    hg qapplied

    >>> change1.patch
    >>> change2.patch

## Rebase Extension

The rebase extension is a useful tool that allows you too keep your
local changes "detached" from the mainstream repository while still
keeping them compatible with it. This extension will essentially reapply
your local changes, i.e., the changes in your patch queue, on top of the
up stream changes. More info about the rebase extension, and its
advanced uses, can be found
[here](http://mercurial.selenic.com/wiki/RebaseExtension).

### Example use of the rebase extension

To enable the rebase extension, simply add the following to your
**.hgrc** file:

`[extensions]`
`rebase =`

Suppose you have some patches applied in your local patch queue, then
you do a pull request from the upstream repo:

`hg pull -u`

Now, simply rebase your local changes:

`hg rebase`
