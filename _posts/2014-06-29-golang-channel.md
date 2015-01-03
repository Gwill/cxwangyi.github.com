# Channels in Go

## Reference Type

A channel variable is in fact a C pointer pointing to a blocking queue
implementation in C.  So, it is cheap to pass a channel as function
parameters.

## Access

Channel variables can be defined for read-and-write (bidirectional),
read-only (outbound) and write-only (inbound).

    chan   int  // a channel can be written into and read from
    chan<- int  // integers can be written into the channel
    <-chan int  // integers can be read out from the channel

## Creation

Channels are created using `make`:

    c := make(chan int)      // unbuffered channel of integers
    c := make(chan int, 100) // buffered channel of integers

The buffer size is the size of the blocking queue.

## Cap and Len

The builtin function `cap`, when applied with a channel, returns the
buffer capacity.

The builtin function `len`, when applied with a channel, returns the
number of elements queued in the channel buffer.

`cap` and `len` return 0 for `nil` channels.

## Read and Write

Writing to a full (or unbuffered) channel blocks, until a reader reads
something thus make room in the buffer.

Reading to a full (or unbuffered) channel blocks, until a writers
writes something that can be read.

## Close a Channel

A sender can close a bidirectional or outbound channel using `close`,
after send the last value.

After this, any receive from the channel will succeed without
blocking, returning the zero value of the channel element.  The form:

    x, ok := <-c

will also set `ok` to `false` for a closed channel.

## Use with For-Range

The following `for` loop reads values sent on channel `c` until the
`c` is closed.

    for e := range c {
       // process e
    }

## Use for Comminication

Communication is the primary design purpose of channels.  There are
many use cases on the Internet.

## Use for Notification

Channel can also be used for communication of notifications.

    c := make(chan int)
    go func() {
      list.Sort()
      c <- 1 // notify task completion
    }
    <-c  // wait unitl list.Sort() is completed.

Above example can be extended to wait for a group of goroutines.
