---
layout: post
title: Living in Arch, i3wm, Vim etc
---

Starting as a basic reminder of the special hacks.  

[TOC]

#### Chinese input method

1.  Install `fcitx` and `fcitx-sunpinyin`, finally the `fcitx-im` front-end group.
2.  [Set environment variables and start up commands](https://wiki.archlinux.org/index.php/Fcitx#Desktop_Environment).
3.  Add `fcitx -d -r --enable fcitx-sunpinyin`.
4.  Utilize the `fcitx-configtool` to get the rest working.


#### Copying from Vim to Clipboard / Primary / Selection or whatever it is.

1.  Install `xclip`.
2.  Go back to Vim, and enter `:!xclip -f %` to yank the text in current file
    to Clipboard / Primary / Selection or whatever.
3.  Go to where you'd like to paste, and hit `middle mouse button`

Alternatively, in terminal, type
```shell
cat file-to-yank | xclip
xclip -o                    # check if yanked correctly
xclip -o | xclip -sel clip  # send from PRIMARY to CLIPBOARD
```
and `Ctrl-V` should work now.  

Interestingly though, if you have installed `fcitx-sunpinyin`, a regular
`"+yy` followed by a `Ctrl-;` would get you a pop-up list of yanked texts.  


#### Secondary display notes

```shell
xrandr --output HDMI2 --auto        # if it is not turned on by default
xrandr --output HDMI2 --dpi 165     # NOTE: the dpi change is applied GLOBALLY!
xrandr --output HDMI2 --right-of eDP1 --rotate left     # the programmer's setup
```


#### Installing fonts

```shell
cp times*<TAB> ~/.fonts/            # copy font files to user local font dir
                                    # the wildcard makes sure all variations
                                    #     are copied
fc-cache -fv ~/.fonts/              # update font cache
```

Changes should take effect after a restart of the running program.  

