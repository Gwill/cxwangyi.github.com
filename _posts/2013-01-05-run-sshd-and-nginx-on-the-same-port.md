# Run SSHD and Nginx On the Same Port

I have a computer (`bridge`) in the lab, which is highly restricted by the firewall - it can access the Internet, but only those services listening on port 80.

What I want is to use `bridge` as a proxy, so when I am working on the development workstation (`dev`) in the lab, I can access the Internet, including Github, Google and other services.

Due to the firewall restriction, `bridge` along cannot be a full-fledged proxy; but I am lucky to have a VPS (`free`) in the Internet. So what I really need is to set up a tunnel from bridge to free using dynamic forwarding:

    bridge$ ssh -D 7777 `free` -p 80

Note that since `bridge` can only access services listening on port 80, we must configure SSHD on `free` to listen on 80.

However, `free` is a Web server and the Nginx running on it already takes port 80.  So we need to multiplex port 80 for both Nginx and SSD on free.  After Googling around, I found the magical program SSLH (http://www.rutschle.net/tech/sslh.shtml).

SSLH is a light port-de-multiplexing server in 2000 lines of C code.  It listens on specified port, and forward connections to the port to other services by testing on the first data packet and judging the protocol of the connection.

Here follows how I use SSLH to multiplex port 80 on `free`.

First, I download and build SSLH. On `bridge`, issue the following commands:

    wget http://www.rutschle.net/tech/sslh-1.14.tar.gz
    tar xzf sslh-1.14.tar.gz
    cd sslh
    make

Building sshlh requires libconfig and libwrap. To install them on Ubuntu systems, use:

    sudo apt-get install libwrap0-dev libconfig8-dev

To make sure how to install them on Debian or other systems, please refer to the README file distributed with SSLH code.

Once successfully built, we would have two binary files: sslh-fork and sslh-select.  The former forwards by forking processes, and is thus slow; whereas the latter forwards by using select, which is much more efficient.  However, sslh-fork had been tested in many use cases and is guaranteed robust, whereas sslh-select requires more tests.  Since `free` runs a simple Web service, I just sslh-fork:

    sudo cp ./sslh-fork /usr/sbin

I also copied the default configuration file and start shell script:

    sudo cp ./scripts/etc.default.sslh /etc/default/sslh
    sudo cp ./scripts/etc.init.d.sslh /etc/init.d/sslh

Then edit /etc/default/sslh to be as follows:

    LISTEN=0.0.0.0:80
    HTTP=localhost:8080
    SSH=localhost:22
    SSL=localhost:443
    USER=nobody
    PID=/var/run/sslh.pid

The first line makes SSLH listens on port 80 of all network interfaces.  The second line specifies the server (Nginx) to which HTTP connections are forwarded.  The third line specifies the SSHD.

To enable above configuration, I also need to edit /etc/init.d/sslh, to ensure that the SSLH will be started by the following command line:

    $DAEMON --user ${USER} --pidfile ${PID} --listen ${LISTEN} --ssh ${SSH} --ssl ${SSL} --http ${HTTP}

Note that variables LISTEN, SSH, SSL, and HTTP are read from above configuration file.

Since the SSHD on `free` had already been configured to listen on port 22, I do nothing to it.  But I need to make Nginx listens on localhost:8080 (port 8080 on the loop back interface) instead of 0.0.0.0:80 (port 80 on all network interfaces).

My Nginx is installed in /usr/local/nginx and configuration files are in /usr/local/nginx/conf.  What I did is to ensure that all virtual servers defined in /usr/local/nginx/conf/nginx.conf and other config files included by nginx.conf listen on localhost:8080.  A fast command is:

    sudo sed -i'' 's/listen *80/listen localhost:8080/' *.conf

Then we restart Nginx, and make sure that no service is listening on port 80 by using the following command:

    netstat -tpln | grep 80

At this moment, it is correct if we cannot access the Web service served by Nginx from any computer other than localhost. (Because we edited the nginx.conf to bind all Nginx virtual servers to the loop back interface, localhost.)

Then we start the SSLH service:

    sudo /etc/init.d/sslh start

Everything is done!

If everything is ok, we should be able to connect to `free` via SSH and port 80:

    ssh free -p 80

We should also be able to access Nginx server on `free` from a Web browser with URL `http://free`.
