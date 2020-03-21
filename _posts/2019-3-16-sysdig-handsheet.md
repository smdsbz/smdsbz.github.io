---
layout: article
title: Sysdig Cheatsheet
key: sysdig-cheatsheet
tags: UnixTools
---

Ready-to-use commands with `sysdig`.  

<!-- more -->

> __Note__  
> `sysdig` only works upon supported kernel, probably fail to insert mod on official preview channel kernel or a custom
> one.  

__Show user, command name and arguments for every program launched by a real user (i.e. from bash)__  

```console
$ sysdig -p "%user.name) %proc.name %proc.args" evt.type=execve and evt.arg.ptid=bash
```

> __Note__  
> Apply more rules to filter, otherwise will list `system()` calls in program as well.  

Listing user commands (with a fully-customized tmux, listing date, running `tmux-contimuum`):  

```text
smdsbz) dirname /home/smdsbz/.tmux/plugins/tmux-continuum/scripts/continuum_save.sh
smdsbz) check_tmux_vers bash /home/smdsbz/.tmux/plugins/tmux-continuum/scripts/check_tmux_version.sh 1.9
smdsbz) check_tmux_vers bash /home/smdsbz/.tmux/plugins/tmux-continuum/scripts/check_tmux_version.sh 1.9
smdsbz) check_tmux_vers bash /home/smdsbz/.tmux/plugins/tmux-continuum/scripts/check_tmux_version.sh 1.9
smdsbz) check_tmux_vers bash /home/smdsbz/.tmux/plugins/tmux-continuum/scripts/check_tmux_version.sh 1.9
smdsbz) check_tmux_vers bash /home/smdsbz/.tmux/plugins/tmux-continuum/scripts/check_tmux_version.sh 1.9
smdsbz) check_tmux_vers bash /home/smdsbz/.tmux/plugins/tmux-continuum/scripts/check_tmux_version.sh 1.9
smdsbz) check_tmux_vers bash /home/smdsbz/.tmux/plugins/tmux-continuum/scripts/check_tmux_version.sh 1.9
smdsbz) bash /home/smdsbz/.tmux/plugins/tmux-continuum/scripts/check_tmux_version.sh 1.9
smdsbz) tr -dC [:digit:]
smdsbz) tmux -V
smdsbz) tr -dC [:digit:]
smdsbz) tmux show-option -gqv @continuum-save-interval
smdsbz) tmux show-option -gqv @continuum-save-last-timestamp
smdsbz) tmux show-option -gqv @continuum-save-interval
smdsbz) date +%s
smdsbz) dirname /home/smdsbz/.tmux/plugins/tmux-continuum/scripts/continuum_save.sh
smdsbz) check_tmux_vers bash /home/smdsbz/.tmux/plugins/tmux-continuum/scripts/check_tmux_version.sh 1.9
smdsbz) check_tmux_vers bash /home/smdsbz/.tmux/plugins/tmux-continuum/scripts/check_tmux_version.sh 1.9
smdsbz) check_tmux_vers bash /home/smdsbz/.tmux/plugins/tmux-continuum/scripts/check_tmux_version.sh 1.9
smdsbz) check_tmux_vers bash /home/smdsbz/.tmux/plugins/tmux-continuum/scripts/check_tmux_version.sh 1.9
smdsbz) check_tmux_vers bash /home/smdsbz/.tmux/plugins/tmux-continuum/scripts/check_tmux_version.sh 1.9
smdsbz) check_tmux_vers bash /home/smdsbz/.tmux/plugins/tmux-continuum/scripts/check_tmux_version.sh 1.9
smdsbz) check_tmux_vers bash /home/smdsbz/.tmux/plugins/tmux-continuum/scripts/check_tmux_version.sh 1.9
smdsbz) bash /home/smdsbz/.tmux/plugins/tmux-continuum/scripts/check_tmux_version.sh 1.9
smdsbz) tr -dC [:digit:]
smdsbz) tmux -V
smdsbz) tr -dC [:digit:]
smdsbz) tmux show-option -gqv @continuum-save-interval
smdsbz) tmux show-option -gqv @continuum-save-last-timestamp
smdsbz) tmux show-option -gqv @continuum-save-interval
smdsbz) date +%s
```

See also  

- `proc.cmdline`, `proc.exeline`
    - Prints full command line.
- `fd.name`
    - Possible using filter `fd.name contains /dev/pts` to get `pts` of a shell / terminal emulator, achieving peeking
        into their console output (with `script` or else).

