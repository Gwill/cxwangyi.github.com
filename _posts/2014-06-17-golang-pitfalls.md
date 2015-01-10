# Pitfalls in Go"

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

## Keep methods of an RPC type clear

When you create an RPC server, you might want to monitor the RPC
variable using `expvar`.  A straightforward solution is to define a
`String()` method for the RPC type:

    type RPCSerive struct {
      state State
    }
    func (s *RPCService) String() string { return fmt.Sprintf(...) }

    s := new(RPCService)
    rpc.Register(s)
    expvar.Publish("service", s)

However, `rpc.Register` would warn, saying that `String` does not
match the signature requirements of RPC methods.

The solution is not to define `RPCService.String`; instead, define
`State.String()` and publish `s.state` instead of `s`.
