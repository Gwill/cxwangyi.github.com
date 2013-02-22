---
layout: post
title: "Use Blade with Mac OS X"
description: ""
category:
tags: []
---
{% include JB/setup %}

[typhoon-blade](http://code.google.com/p/typhoon-blade) is a
convenient software build tool designed for Linux.  However, in order
to use it under Mac OS X, you need to do some hacks.

## The `readlink` Tool

Mac OS X uses BSD user tools, which might have different command line
flags compared with their counterparts with Linux.

After download and unpack `blade` into an arbitrary directory, say
`~/blade`, you need to edit `~/blade/blade` and change the command
line flag of all `readlink` invocations into `-n`.

After doing so, you cannot invoke blade by `~/blade/blade` directly;
instead, you need to create a symbolic link, say, `~/bin/blade -->
~/blade/blade`, and invoke that symbolic link.

## Relax the Compiler ##

In order to force developers using Google C++ code style, blade would
require the C++ compiler to do strict syntax checking.  The following
flags appear in `~/blade/src/blade/blade_platform.py`:

                "-Werror=char-subscripts",
                "-Werror=comments",
                "-Werror=conversion-null",
                "-Werror=empty-body",
                "-Werror=endif-labels",
                "-Werror=format",
                "-Werror=format-nonliteral",
                "-Werror=missing-include-dirs",
                "-Werror=non-virtual-dtor",
                "-Werror=overflow",
                "-Werror=overloaded-virtual",
                "-Werror=parentheses",
                "-Werror=reorder",
                "-Werror=return-type",
                "-Werror=sequence-point",
                "-Werror=sign-compare",
                "-Werror=switch",
                "-Werror=type-limits",
                "-Werror=uninitialized",
                "-Werror=unused-label",
                "-Werror=unused-result",
                "-Werror=unused-value",
                "-Werror=unused-variable",
                "-Werror=write-strings"

However, these checks would be too strict that even the standard C++
library coming with XCode 4.6 does not pass the check.

I removed some of them, in particular, `-Werror=empty-body` and
`-Werror=missing-include-dirs`, to make it work with Mac OS X.

*IMPORTANT:* After doing above changes, you need to repack the Python
 code to make it work:

     cd ~/blade
     ./dist_blade
