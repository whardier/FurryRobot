Why Furry Robot?
================

Github suggested it.  I rather like their project naming random suggestion 
feature.

Why?
====

Because it was fun

How?
====

Use IPTables to redirect all traffic leaving your network for a destination 
subnet to go to localhost on a local port.

Inetd answers the request and starts up furryrobot, a python program, which then 
attempts to get the source address and port from inetd and then uses that 
information against the list of connection tracking ports.

The connection tracking table is scanned up to 10 times with a small wait 
between while it populates.  Once a match is found the original destination 
address and port are fed to an SSH process which takes over stdin/stdout of the 
python script.  Tada.. "You're in."

How do I.. do what exactly?
===========================

Nothing except for trusting a key needs to be done on the server side.  Once 
you've shared a passwordless key with a remote server that has a version of 
OpenSSH that supports the -W flag (your client must support the -W flag as well) 
then you're ready to get started.

I use OpenBSD-inetd.. here's the line in my config:

    # <service_name> <sock_type> <proto> <flags> <user> <server_path> <args>
    65535   stream  tcp nowait  root /usr/local/bin/furryrobot furryrobot /root/.ssh/id_dsa remoteuser@remotetunnelserver

That's right.  I'm running it as root.  If I don't run it as root I won't have 
access to the connection tracking tables!

Now for the IPTables line:

    itables -t nat -A OUTPUT -d 10.0.0.0/8 -p tcp -m tcp -j REDIRECT --to-ports 65535

Now when I try to reach 10.0.0.0/8 on ANY port it will be redirected to the 
local machine at port 65535 where inetd is listening.

You're basically done at this point.  Try to telnet to somewhere behind 
`remotetunnelserver` on a port that's listening.. for instance:

    spencersr@bigboote$ telnet 10.1.0.68 22
    Trying 10.1.0.68...
    Connected to 10.1.0.68.
    Escape character is '^]'.
    SSH-2.0-OpenSSH_5.5p1 Debian-6

Good day to you sir.
