---
title: "SPEComp"
date: 2018-05-13T18:51:37-04:00
draft: false
---
## How to run SpecOMP on M5

1.  build cross-compiler
    1.  download crosstool-0.43.tar.gz from
        <http://kegel.com/crosstool/#download>
        (http://kegel.com/crosstool/crosstool-0.43.tar.gz)
    2.  untar it
    3.  edit demo-alpha.sh
        1.  RESULT_TOP="your_alpha_compiler"
        2.  GCC_LANGUAGE="c,c++" --\> GCC_LANGUAGE="c,c++,fortran"
        3.  eval \`cat ...dat ...dat\` sh all.sh --\> eval \`cat
            alpha.dat gcc-4.2.4-glibc-2.3.6-tls.dat\` sh all.sh --notest
    4.  set gcc to gcc-3.4 instead of gcc-4.3 (when I use gcc-4.3 to
        build the cross-compiler, there is a segmentation fault)
    5.  run demo-alpha.sh
    6.  there may be some errors (a header file (.h) missing " or some
        things like that). You can modify the files directly and fix
        these errors.
    7.  after you modify the files, you should change demo-alpha.sh
        again
        1.  eval \`cat alpha.dat gcc-4.2.4-glibc-2.3.6-tls.dat\` sh
            all.sh --notest --\> eval \`cat alpha.dat
            gcc-4.2.4-glibc-2.3.6-tls.dat\` sh all.sh --nounpack -notest
              - \--nounpack flag avoids downloading the source again to
                overwrite you modification
    8.  get a new cross-compiler for alpha supports openmp
2.  compile SpecOmp
      - because runspec may not work in M5, I compile the source
        directly
    <!-- end list -->
    1.  install specomp by following its document
    2.  change directory to benchspec
    3.  edit
            Makefile.defaults
        1.  LIBS=-Lyour_alpha_compiler/gcc-4.2.4-glibc-2.3.6/alpha-unknown-linux-gnu/lib
        2.  CC=your_alpha_compiler/gcc-4.2.4-glibc-2.3.6/alpha-unknown-linux-gnu/bin/alpha-unknown-linux-gnu-gcc
        3.  CFLAGS=-Iyour_alpha_compiler/gcc-4.2.4-glibc-2.3.6/alpha-unknown-linux-gnu/include
            $(EXTRA_CFLAGS) $(PORTABILITY) $(CPORTABILITY) -fopenmp
            -O3
        4.  CXX=your_alpha_compiler/gcc-4.2.4-glibc-2.3.6/alpha-unknown-linux-gnu/bin/alpha-unknown-linux-gnu-gcc
        5.  FC=your_alpha_compiler/gcc-4.2.4-glibc-2.3.6/alpha-unknown-linux-gnu/bin/alpha-unknown-linux-gnu-gfortran
        6.  FFLAGS=-Iyour_alpha_compiler/gcc-4.2.4-glibc-2.3.6/alpha-unknown-linux-gnu/include
            $(EXTRA_FFLAGS) $(PORTABILITY) $(FPORTABILITY) -fopenmp
            -O3
        7.  F77FLAGS=-Iyour_alpha_compiler/gcc-4.2.4-glibc-2.3.6/alpha-unknown-linux-gnu/include
            $(EXTRA_FFLAGS) $(PORTABILITY) $(FPORTABILITY) -fopenmp -O3
        8.  LD=$(CC) $(CFLAGS)
            -Lyour_alpha_compiler/gcc-4.2.4-glibc-2.3.6/alpha-unknown-linux-gnu/lib
            -fopenmp -O3
    4.  change directory to OMPM2001/3xx.xxxx/src (e.g., 320.equake/src)
    5.  make and get the binary for alpha
    6.  the list which can be compiled without error: wupwise, swim,
        applu, equake, apsi, gafort (however, there is a segmentation
        fault during execution), art, ammp
3.  put files into images
    1.  copy all files and directories from
        your_alpha_compiler/gcc-4.2.4-glibc-2.3.6/alpha-unknown-linux-gnu/alpha-unknown-linux-gnu/lib
        to /lib of the linux image for M5 (e.g., linux-latest.img)
    2.  copy the executables and input files of SpecOmp to another image
        and set it as the third disk image of M5
4.  run specomp
    1.  run M5
    2.  mount the image containing the files of specomp
    3.  execute specomp executable with correspondent commands.

I am not very sure that I remember everything. However, the operation
flow may help others who want to do the same thing. If there is any
missing steps or mistakes, please correct me.
