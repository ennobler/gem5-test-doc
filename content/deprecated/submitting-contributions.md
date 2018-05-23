---
title: "Deprecated submitting contributions"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

# OLD CONTRIBUTION DOCUMENTATION

**All of the contribution details have been moved into the gem5 source
tree. The information below is out of date\!**

If you've made changes to gem5 that might benefit others, we strongly
encourage you to contribute those changes to the public gem5 repository.
There are several reasons to do this:

  - Once your changes are part of the main repo, you no longer have to
    merge them back in every time you update your local repo. This can
    be a huge time savings\!
  - Once your code is in the main repo, other people have to make their
    changes work with your code, and not the other way around.
  - Others may build on your contributions to make them even better, or
    extend them in ways you did not have time to do.
  - You will have the satisfaction of contributing back to the
    community.

In the common case, the process is fairly simple:

1.  Organize your changes into one or more self-contained, documented
    patches with appropriate commit messages.
2.  Test your patches using the regression tests, as well as any other
    specific tests required to exercise your code.
3.  Post your patch(es) on [our Gerrit
    server](https://gem5-review.googlesource.com).
4.  Wait for reviews. Reviewers may ask you to modify your patch, or may
    engage you in some discussion.
5.  If necessary, update your patch based on initial reviews and wait
    for re-reviews.
6.  Once you've resolved any outstanding reviewer concerns and received
    a few 'ship it\!' reviews, someone with commit access should
    volunteer to commit your code. If they don't, please make an
    explicit request on the gem5-dev mailing list. If you submit enough
    high-quality patches that people find it annoying to keep committing
    your patches for you, you will likely be given commit access
    yourself :).

The sections below provide more detail on these steps.

### Creating/Submitting Patches to Gerrit

Please see the [contributing
documentation](https://gem5.googlesource.com/public/gem5/+/master/CONTRIBUTING.md)
for gem5-specific contributing notes and [the gerrit
documentation](https://gerrit-review.googlesource.com/Documentation/index.html)
for gerrit-specific instructions.

### Creating Patches

See [Managing Local Changes with Mercurial
Queues](Managing_Local_Changes_with_Mercurial_Queues "wikilink") to
learn how to set up an use MQ to manage your code changes. The rest of
this section will assume that you are using MQ to manage the patches you
wish to contribute.

  - When you've got something that you want to commit, think about what
    it does and who it might affect. Is it self contained? Can it be
    broken up into more logical segments? gem5 is a big project and
    there are a lot of people working on it. No one person knows
    everything, so we rely on the revision history and on comments to
    understand what is going on in the code. It is very important that
    you make change sets self contained. It is far easier to understand
    and review a series of ten 200 line changesets and what they do
    compared to a single 2000 change set that does a whole lot of stuff.
    This will also help you as a developer. If you keep your changesets
    small and self contained and you read them regularly, you will find
    your own bugs before you commit them.

<!-- end list -->

  - Please don't forget to put commit messages on your patches. The
    `-e`, `-l`, or `-m` options to `hg qref` will allow you to do this
    (run `hg help qref` for details).

<!-- end list -->

  - We want to be able to identify everyone who makes a change to the
    repository, so we need your name to show up in a consistent manner
    in the repository. Please add this to your `$HOME/.hgrc` file before
    you start making changesets (`hg commit` or `hg qfinish`)

<!-- end list -->

    [ui]
    username=Nathan Binkert <nate@binkert.org>

  - Please try to follow the [Coding Style](Coding_Style "wikilink").
    They make search and replace much more effective. It also makes it
    easier for developers to follow code if it all has a similar look.
    I'm sure that there are things that you hate about the style. I've
    never worked on a project where everyone liked the style, unless
    they've all worked on the project for a long time and just gotten
    used to it.

### Commit Messages

A canonical commit message consists of three parts:

  - A short summary line describing the change. This line starts with
    one or more keywords separated by commas followed by a colon and a
    description of the change. This line should be no more than 65
    characters long since version control systems usually add a prefix
    that causes line-wrapping for longer lines.
  - (Optional, but highly recommended) A detailed description. This
    describes what you have done and why. If the change isn't obvious,
    you might want to motivate why it is needed. Lines need to be
    wrapped to 75 characters or less.
  - (Optional) Tags describing patch metadata. You are highly
    recommended to use tags to acknowledge reviewers for their work.

For example:

    scons, arch: Fixed the way the build system handled ISAs

    Note the keyword and colon on the first line, and that it's < 65 chars.
    Please keep your detailed commit comments to 75 columns or less.  These
    comments will likely have to be wrapped manually, but that's not so bad
    a thing to have to do.

    Reported-by: Dedicated gem5 User <email>
    Signed-off-by: Original Developer <email>
    Reviewed-by: Grey Beard <email>
    [Committer: Rebased patch onto latest gem5]
    Signed-off-by: Committer <email>

The keyword should be one or more of the following separated by commas:

  - Architecture name in lower case (e.g., **arm** or **x86**): Anything
    that is target-architecture specific.
  - **base**
  - **ext**
  - **stats**
  - **sim**
  - **syscall_emul**
  - **config**:
  - **mem**: Classic memory system. Ruby uses its own keyword.
  - **ruby**: Ruby memory models.
  - **cpu**: CPU-model specific (except for kvm)
  - **kvm**: KVM-specific. Changes to host architecture specific
    components should include an architecture keyword (e.g., **arm** or
    **x86**) as well.
  - **gpu-compute**
  - **energy**
  - **dev**
  - **arch**: General architecture support (src/arch/)
  - **scons**: Build-system related. Trivial changes as a side effect of
    doing something unrelated (e.g., adding a source file to a
    SConscript) don't require this.
  - **tests**
  - **style**: Changes to the style checkers of style fixes.
  - **misc**

Tags are an optional mechanism to store additional metadata about a
patch and acknowledge people who reported a bug or reviewed that patch.
Tags are generally appended to the end of the commit message in the
order they happen. We currently use the following tags:

  - **Signed-off-by:** Added by the author and the submitter (if
    different). This tag is a statement saying that you believe the
    patch to be correct and have the right to submit the patch according
    to the license in the affected files. Similarly, if you commit
    someone else's patch, this tells the rest of the world that you have
    have the right to forward it to the main repository. If you need to
    make *any changes at all* to submit the change, these should be
    described within hard brackets just before your Signed-off-by tag.
  - **Reviewed-by:** Used to acknowledge patch reviewers. It's generally
    considered good form to add these. Some code review systems add them
    automatically.
  - **Reported-by:** Used to acknowledge someone for finding and
    reporting a bug.
  - **Reviewed-on:** Link to the review request corresponding to this
    patch. Some review systems add these automatically.
  - **Change-Id:** Used by Gerrit to track changes across rebases.
  - **Tested-by:** Used to acknowledge people who tested a patch.
    Sometimes added automatically by review systems that integrate with
    CI systems.

### Testing Patches

  - Before you circulate your patch for review, ask yourself: Have I
    compiled this code for all ISAs? Have I run the regression tests
    that are relevant? (You should run at least the quick regressions
    and the long too if you think you're going to change any results.)
    You should avoid embarrassing yourself, annoying reviewers, or worst
    of all breaking the tree with trivial things that can be caught by
    these two steps.

### Posting Patches

  - Now that you've made small, self contained changesets, you should
    seek feedback from people. We have our own [Reviewboard
    server](http://reviews.m5sim.org) for this process. You can use the
    [Reviewboard
    Extension](https://www.mercurial-scm.org/wiki/ReviewboardExtension)
    which provides the `hg postreview` command to send out your
    changesets for review. Some Tips:
      - If you want to post a review for the changeset at the tip, use
        "`hg postreview -o`"
      - If you want to update a review for the changeset at the tip, use
        "`hg postreview -o -u -e `<number>", where <number> is the
        review number on reviewboard.
      - Everything is easier if you are using Mercurial Queues as you
        can use "`hg qpush`" and "`hg qpop`" to get your diff to the tip
        and then use the postreview commands above.

If your patch touches Ruby code, please get a review from Brad or Nilay
before committing. If your patch touches ARM code please get a review
from Ali or Andreas before committing.

### Responding to Reviews

  - Your reviews may request some changes; if so, make those and (unless
    the changes are trivial) re-post the modified patch for review,
    updating the existing review as described above.

### Committing Patches

  - Once the reviewers are satisfied with your change, make sure your
    change is based on the up-to-date head of the tree. If you're using
    mq, this is pretty trivial: `hg qpop -a; hg pull -u; hg qpush`
    (assuming the tree hasn't changed too much out from under you)
  - If you made any changes in the last two steps, recompile and re-run
    the quick regression tests. Even if your changes were totally
    trivial, you should at least recompile and run the "hello world"
    test to make sure you haven't done something dumb (speaking from
    experience).
  - If you're a new developer and don't have commit access, you'll have
    to get someone with commit access to push the patch to the
    repository on your behalf. If you do have commit access, follow
    these steps:
      - Now that you're ready to commit, you can `hg qfinish` the
        relevant patches to turn them into changesets.
      - Ok, so you're ready to push your changesets to the repository.
        One last step. Run `hg outgoing`. Are you about to push what you
        think you are? Do the commit messages all make sense? Ok, go
        ahead and `hg push`.

If your patch has been on reviewboard for a while without getting any
reviews (or re-revires after you've posted changes), please email the
gem5-dev list. If you have commit access, it is fair to give warning via
email that you intend to commit the changes at some future date (e.g., a
week out from the date of the email) if you do not hear any objections.
Please do not simply commit a patch without giving warning on the
gem5-dev list.

### Committing Patches for other contributors

1.  Download patch from reviewboard
2.  Import the patch (hg qimport)
3.  Push the patch (hg qpush)
4.  Fix any style errors, etc.
5.  Copy description from reviewboard (hg qref -e OR vi
    .hg/patches/<patch>)
6.  Update the patch user with reviewboard info for the person who wrote
    the patch (hg qref -u “Name <email>”).
7.  Put your ID as “Signed-off-by: Name <email>”
8.  Check that it looks right (hg log -l1 -v)
9.  Test the patch(es) (something like scons
    build/ARM/tests/opt/quick/se/). Do what’s appropriate, the submitter
    should have already tested\!
10. MAKE SURE NO OTHER PATCHES HAVE BEEN COMMITTED BEFORE DOING THE
    FOLLOWING\!
11. Commit the patches locally (hg qfinish -a)
12. Make sure the commit messages still look right (hg log)
13. Push the patches (hg push <ssh://hg@repo.gem5.org/gem5>)
14. Ask the submitters to close the reviews on reviewboard
