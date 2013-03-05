---
layout: post
title: "Use Google Libs with Blade on Mac OS X"
description: ""
category:
tags: []
---
{% include JB/setup %}

In this
[article](http://cxwangyi.github.com/2013/02/22/use-blade-with-mac-os-x/),
I explained how to use
[typhoon-blade](http://code.google.com/p/typhoon-blade/) on Mac OS X.

In addition to the build tool `blade`, the typhoon-blade project also
provides a set of handful libraries open sourced by Google, collectly
known as `google-libs`, prebuilt for 64-bit Linux.

I use these libraries in programming distributed systems, as they
would be linked statically by blade into my program, thus makes it
easy to deploy my programs on computer cluster running Linux, without
bothering me to deploy dependent dynamic libraries.

However, my primary development environment is Mac OS X.  So I
modified teh build rules of google-libs to make it work with Mac OS X.
Also, I need to change blade a little to make it adapt my changes to
google-libs.

Because after you unpack google-libs, the code goes into a directory
known as `//thirdparty`, in the rest of this post, I use the word
`google-libs` and `thirdparty` interchangablly.

## Fake `//thirdparty` on Mac OS X

The google-libs contains primarily the following three libraries:
`//thirdparty/glog`, `//thirdparty/gtest`, and `//thirdparty/gflags`.
What I did is to change BUILD files of these libraries and make them
alias to the system-wide libraries installed using Homebrew.

### Edit `//thirdparty/gtest/BUILD`

We would like to use the gtest installed by Homebrew, instead of the
one in corp SVN prebuilt for 64-bt Linux.  So edit
`//thirdparty/gtest/BUILD` and make the target
`//thirdparty/gtest:gtest` an alias of `#gtest`.

    cc_library(name = 'gtest',
               srcs = 'null.cc',
               deps = ['#gtest']
              )

    cc_library(name = 'gtest_main',
               srcs = 'null.cc',
               deps = ['#gtest_main']
              )

    # cc_test(
    #    name = 'gtest_test',
    #    srcs = 'gtest_test.cc'
    # )

Note that we need to add `srcs = 'null.cc'` into the `cc_libraray`;
otherwise, it would complains that the command `ar cr libgtest.a`
creates an empty archive.

### Edit `//thirdparty/glog/BUILD`

    cc_library(
        name = 'glog',
        srcs = 'null.cc',
        # deps = '//thirdparty/glog-0.3.2/src:glog'
        deps = ['#glog', '#gflags']
    )

    cc_test(
        name = 'glog_test',
        srcs = 'glog_test.cc',
        deps = ':glog'
    )

### Edit `//thirdparty/gflags/BUILD`

    cc_library(
        name = 'gflags',
        srcs = 'null.cc',
        # deps = '//thirdparty/gflags-2.0/src:gflags'
        deps = '#gflags'
    )

    cc_test(
        name = 'gflags_test',
        srcs = 'gflags_test.cpp',
        deps = ':gflags'
    )


## No `//thirdparty` in Include Path

By default, `blade` would try to find header files in `//thirdparty`.
So, even if we write `#include <glog/logging.h>` in our source code,
`blade` would find `//thirdparty/glog/logging.h`, instead of
`/usr/local/include/glog/logging.h`.

To change this, we need to modify `blade/blade.conf` as follows:

    cc_config(
        # extra_incs='/thirdparty'
        extra_incs='/usr/local/include'
    )


## Linker Flags for Mac OS X

The Apple `ld` coming with Mac OS X does not recognize the flag
`--no-undefined` as GNU `ld`, but it supports a flag, `--undefined
error`, which has exactly the same function.  So we need to modify
`blade` source code file `~/blade/src/blade/cc_targets.py` to change
the following line

    self._write_rule('%s.Append(LINKFLAGS=["-Xlinker", "--no-undefined"])'

by

    self._write_rule('%s.Append(LINKFLAGS=["--undefined error"])'


Remember to redistribute `blade` by running `~/blade/dist_blade`.


## Use System-wide `protoc`

In order to use the system-wide `protoc` instead of the one prebuilt
by blade authors for Linux, we need to edit `blade.conf` and change
the following

    proto_library_config(
        protoc='thirdparty/protobuf/bin/protoc',
        protobuf_libs=['thirdparty/protobuf:protobuf'],
        protobuf_path='thirdparty',
        protobuf_include_path = 'thirdparty',
        protobuf_php_path='thirdparty/Protobuf-PHP/library',
        protoc_php_plugin='thirdparty/Protobuf-PHP/protoc-gen-php.php'
    )

into

    proto_library_config(
        protoc='/usr/local/bin/protoc',
        protobuf_libs=['#protobuf'],
        protobuf_path='/usr/local/lib',
        protobuf_include_path = '/usr/local/include/google',
    )


## Use System-wide `libgtest_main.a`

We need to change `blade.conf` to tell blade to link the system-wide
`libgtest_main` instead of that in `//thirdparty`.

    gtest_main_libs=['#gtest_main']
