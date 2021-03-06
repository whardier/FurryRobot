Why Furry Robot?
================

Github suggested it.  I rather like their project naming random suggestion 
feature.

Why?
====

Because it was fun.  Also because I was annoyed.  I often combine the two.

This program was thought up "on the pot" so to say.  I simply wanted to get 
behind our firewall without mucking with SOCKS, HTTP, or special connection 
tunneling support programs for each individual TCP application.

I also wanted to show off the -W flag OpenSSH has introduced into my world.

I use the -W flag rather than setting up local/remote port forwarding because 
its fast, it doesn't have to juggle multiple tunneled sockets, and it's simple.

So simple a few lines of python could utilize it for a project like this.

Requirements!
=============

- Linux! (More operating systems soon!)
- IPTables! (ipvw coming soon!)
- Python! (or pypy!)

How?
====

Use IPTables to redirect all traffic leaving your network for a destination 
subnet to go to localhost on a local port.

Inetd answers the request and starts up furryrobot, a python program, which then 
attempts to get the source address and port from inetd and then uses that 
information against the list of connection tracking ports.

The TCP state connection table is read in and the source address and port are 
used to match the connection line. Once a match is found the original 
destination address and port are fed to an SSH process which takes over 
stdin/stdout of the terminal.  Tada... you're in.

How do I.. do what exactly?
===========================

Nothing except for trusting a key needs to be done on the server side.  Once 
you've shared a passwordless key with a remote server that has a version of 
OpenSSH that supports the -W flag (your client must support the -W flag as well) 
then you're ready to get started.

I use OpenBSD-inetd.. here's the line in my config:

    # <service_name> <sock_type> <proto> <flags> <user> <server_path> <args>
    65535   stream  tcp nowait  spencersr /usr/local/bin/furryrobot furryrobot --inetd --ssh-identity-file /home/spencersr/.ssh/id_dsa_furryrobot --ssh-host remoteuser@remotetunnelserver

Now for the IPTables line:

    itables -t nat -A OUTPUT -d 10.0.0.0/8 -o lo -p tcp -m tcp -j REDIRECT --to-ports 65535

And finally the route command:

    ip route add 10.0.0.0/8 dev lo metric 100

Now when I try to reach 10.0.0.0/8 on ANY port it will be redirected to the local interface, intercepted by iptables, and then 
local machine at port 65535 where inetd is listening.

The route line is very important.  It removes the need to know what your current default route is and allows local networks to have preference.

You're basically done at this point.  Try to telnet to somewhere behind 
`remotetunnelserver` on a port that's listening.. for instance:

    spencersr@bigboote$ telnet 10.1.0.68 22
    Trying 10.1.0.68...
    Connected to 10.1.0.68.
    Escape character is '^]'.
    SSH-2.0-OpenSSH_5.5p1 Debian-6

Good day to you sir.
