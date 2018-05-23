---
title: "Asking for help"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 70
---

Many of the people on the gem5-users mailing list are happy to help when
someone has a problem or something doesn't work. However, please keep in
mind that it's not our job to help. We all have other commitments, so
before we spend time helping someone we like to see that they have put
some effort into solving the problem themselves.

1.  Before posting a question to the list, please check if the question
    is already answered. The wiki provides answers to the most common
    questions (check both [Documentation](Documentation "wikilink") and
    the [Frequently Asked
    Questions](Frequently_Asked_Questions "wikilink")), and the [mailing
    list archive at
    gmane.org](http://dir.gmane.org/gmane.comp.emulators.m5.users) is
    easily searchable.
2.  If your problem is with your configuration script, look at the
    config.ini output. This is the final configuration that's getting
    built in textual form. Make sure this reflects the configuration you
    think you're building.
3.  Make sure you're running with gem5.opt or gem5.debug and not
    gem5.fast. The gem5.fast binary compiles out assertion checking for
    speed, so a problem that causes a crash or mysterious error on
    m5.fast may result in a more informative assertion failure with
    gem5.opt or gem5.debug.
4.  Try running your code on the latest version from the [gem5
    repository](http://repo.gem5.org/gem5), if you're not doing that
    already. Your problem may have been fixed since you last updated
    your local version.
5.  Take a few minutes to look at the source code and see if you can
    identify your problem. If you're running into a specific error
    message, the file and line number where the error message is printed
    should be displayed along with the error message, so start there.
    Even if you don't get a specific error message, look at the module
    that's giving you trouble and see if you can figure it out. Use
    cscope or another tool to find where relevant functions/variables
    are used. gem5 is not like other open-source software packages where
    end users would never be expected to look at the source code when
    problems occur. gem5 is as much of a framework as an application. We
    don't (and you shouldn't) expect to have it do everything you want
    without touching the source code.
6.  If it seems appropriate, enable some debug flags (--debug-flags=Foo)
    and see if the resulting information helps. Your examination of the
    source in step 3 should show you which trace flags might be relevant
    (they're the first argument to the DPRINTF calls). If your problem
    is occurring on the C++ side, don't be afraid to run under gdb to
    see what's really happening. See [Debugging](Debugging "wikilink")
    for more details.
7.  If you still need help, use the information you've gathered in steps
    1-4 to ask the most specific and informative question possible on
    the [gem5-users](http://www.gem5.org/mailman/listinfo/gem5-users)
    list. Include the command line you used, specific error messages,
    program outputs, stack traces, relevant trace snippets, etc. (Don't
    post huge traces in their entirety though... just the relevant
    bits.) If you've written your own scripts, try and find the shortest
    script (or the minimum modification to one of the supplied example
    scripts) that exhibits the same problem and post that. If you found
    something on the wiki but it didn't quite apply or didn't work,
    mention that so we can update the wiki appropriately. If you have a
    theory about what the problem might be, please let us know, but
    include enough basic information so others can decide whether your
    theory is correct.
8.  If you have solved a problem that you reported on the list and the
    answer may be of general interest, post it to the list as a
    follow-up to the original thread so others can benefit from the
    solution.

**Finally, please don't e-mail or call any list member directly unless
explicitly invited to do so. If we haven't responded we are either busy,
don't know the answer or some combination of the two. Pestering will not
get your question answered faster, and it may get it never answered at
all.**

