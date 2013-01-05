---
layout: post
title: "Set Connect Timeout"
description: ""
category: networking
tags: []
---
{% include JB/setup %}

# Set Connection Timeout #

When we talk about timeout, we might refer to connection timeout,
sending timeout or receiving timeout.  This post is about setting the
connection timeout with Linux and BSD (Mac OS X).

## Set Timeout in Programs ##

In header file `sys/socket.h`, there is an API:

    int setsockopt(int socket, int level, int option_name, const void *option_value, socklen_t option_len);

You can set the parameter `option_name` by one of the following

    SO_SNDTIMEO     set timeout value for output
    SO_RCVTIMEO     set timeout value for input

to specify sending or recieving timeouts.

The following Linux kernel code snippet shows that the connection
timeout can also be set by using `SO_SNDTIMEO`.

    timeo = sock_sndtimeo(sk, flags & O_NONBLOCK);

    if ((1 << sk->sk_state) & (TCPF_SYN_SENT | TCPF_SYN_RECV)) {
       /* Error code is set above */
       if (!timeo || !inet_wait_for_connect(sk, timeo))
           goto out;

       err = sock_intr_errno(timeo);
       if (signal_pending(current))
       goto out;
    }

## Overriding the Default Linux Connection Timeout ##

According to [this post](http://www.sekuda.com/overriding_the_default_linux_kernel_20_second_tcp_socket_connect_timeout), OS kernels have limits on the number of retries and timeout.  To change the OS defaults in a running Linux kernel, you can use the `/proc` interface:

    # cat /proc/sys/net/ipv4/tcp_syn_retries
    5
    # echo 6 > /proc/sys/net/ipv4/tcp_syn_retries
