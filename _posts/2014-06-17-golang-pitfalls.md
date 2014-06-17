---
layout: post
title: "Pitfalls in Go"
description: ""
category:
tags: []
---
{% include JB/setup %}

## `break` in `select` or `switch` and in `for`

In the follow code snippet

    for ... {
      select {
        case <-ch:
          break
      }
    }

The `break` jumps out from `select` only, but not that far to get out
of `for`.

## `os/exec.Cmd.Run()` cannot be called repeatedly to restart the process

If you want to monitor the execution of a process and restart it when
it failed.  You can use `os/exec.Cmd.Run()`.  However, if you want to
restart the process, you cannot reuse the existing `os/exec.Cmd`
variable and call `Run()` again.  If you do so, `Run()` returns an
error like "already started".  Instead, you need to create a new
`os/exec.Cmd` object and call its `Run()`.
