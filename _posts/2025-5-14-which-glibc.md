---
layout: article
title: Finding which glibc we are using.
key: which-glibc
tags: C C++ GNU
---

<!-- more -->

1.  Making sure which `gcc` executable we are using.
2.  `gcc -print-file-name=libc.so` to show the glibc's path.

    ```console
    # gcc-10 -print-file-name=libc.so
    /usr/lib/gcc/x86_64-linux-gnu/10/../../../x86_64-linux-gnu/libc.so
    ```

3.  The `libc.so` is almost certainly a linker script, `cat` it to find out the actual path of the shared library.

    ```text
    /* GNU ld script
       Use the shared library, but some functions are only in
       the static library, so try that secondarily.  */
    OUTPUT_FORMAT(elf64-x86-64)
    GROUP ( /lib/x86_64-linux-gnu/libc.so.6 /usr/lib/x86_64-linux-gnu/libc_nonshared.a  AS_NEEDED ( /lib/x86_64-linux-gnu/ld-linux-x86-64.so.2 ) )
    ```

4.  Execute the shared library file and check the version it prints.

    ```text
    GNU C Library (Ubuntu GLIBC 2.31-0ubuntu9.17) stable release version 2.31.
    Copyright (C) 2020 Free Software Foundation, Inc.
    This is free software; see the source for copying conditions.
    There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
    PARTICULAR PURPOSE.
    Compiled by GNU CC version 9.4.0.
    libc ABIs: UNIQUE IFUNC ABSOLUTE
    For bug reporting instructions, please see:
    <https://bugs.launchpad.net/ubuntu/+source/glibc/+bugs>.
    ```
