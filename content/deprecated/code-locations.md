Full system code locations are as follows:

  - `/sim/system` - Architecture and OS independent code
  - `/arch/`<arch>`/system.(cc|hh)` - Architecture dependent, OS
    independent code
  - `/arch/`<arch>`/`<os>`/system.(cc|hh) -` - Architecture dependent,
    OS dependent code
  - `/kern/`<os>`/*` - Architecture independent, OS dependent code

Very little of the code that was in `/kern/`<os> was actually
architecture independent. Even things that at first glance appeared to
be, actually patched architecture dependent parts of the kernel. Since
there wasn't much shared code, I didn't feel any any large amount of
effort should be put into sharing litterally 3 lines so unless we find a
place where there is a lot of code, os dependent architecture
independent code should go in it's proper directory in a namespace for
that OS and simply be called by the source architecture and os dependend
files.