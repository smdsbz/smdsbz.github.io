---
layout: article
title: LLDB Blitzkrieg
tags: UnixTools
---

Simple demonstration of `lldb` capabilities.

```
$ clang++ -Wall -std=c++1z -g ...
$ lldb a.out
(lldb) breakpoint set --file filename --line 80
(lldb) run
(lldb) s        # thread step-in
(lldb) n        # thread step-over
(lldb) f        # thread step-out
(lldb) expr (int) variable_name
# MAGIC: will print `variable_name''s value for you!
(lldb) exit
$ _
```
