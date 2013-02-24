---
layout: post
title: SSH Proxy for Firefox and Chromium
comments: true
time: '15:10'
description: Bash script to open SSH tunnel for proxy
categories: [chromium, code, firefox, howto, privacy, ssh]
---

{% img left http://www.webmastersbydesign.com/wp-content/uploads/2008/07/encryption.jpg %}

When my wife and I were on our [anniversary vacation on Whidbey][], I was sad that the wireless internet access provided to us was unencrypted.  I suppose that I should be glad that any existed at all, since Whittney and I were able to use Netflix as we were there.

[anniversary vacation on Whidbey]:/2010/06/10/whidbey_vacation

I wanted at the time to be able to set up a [ssh tunnel][] to my home server, running the website that you're reading.  The tunnel itself is encrypted, so it would be equivalent security-wise to any browsing that I would do from home.  Something like this would be greatly helpful for using the web at the airport or your local coffee shop.

[ssh tunnel]:http://en.wikipedia.org/wiki/Tunneling_protocol#Secure_Shell_tunneling

<!-- more -->

Anyway, I had a chance to finally set up a script to allow me to easily open and close a tunnel.  Strictly speaking, this is as easy as using the `ssh -D user@server` command, but I wanted to script it somehow.

### The Script ###

{% codeblock lang:bash %}
#!/bin/bash
#
# A simple script to set up or stop a SSH tunnel
# to my home server.
#
# Usage: 
#
#	proxy start
#		Sets up the tunnel and saves the PID in /tmp/ssh_tunnel_id.txt
#
#	proxy stop
#		Reads the process id from /tmp/ssh_tunnel_id.txt and kills that process.

PORT=8080 # The local port to be used for forwarding.  Change if desired.

REMOTE=homeserver # Your server. Mine is aliased in ~/.ssh/config as homeserver, but
		  # this would typically be something like user@server or similar.

PIDFILE=/tmp/ssh_tunnel_pid.txt # will just hold the numeric process id of the tunnel


if [ $# -gt 0 ] ; then # if we pass an arguement (ie, start or stop)

  if [ $1 = start ] ; then
    # make sure there is not an open tunnel already
    if [ -f $PIDFILE ] ; then
      echo "A tunnel is already opened.  No action has been taken."
    else
      # Open the tunnel. We background this ourselves because
      # I don't need a password since I am using passkey pairs, and
      # I can then get at the PID using the '$!' in bash, which doesn't
      # work when passing ssh the -f to background itself.
      ssh -ND localhost:$PORT $REMOTE &

      # save the PID of the tunnel to the PID file
      echo $! > $PIDFILE

      echo "Tunnel has been opened on localhost:$PORT"
    fi
  elif [ $1 = stop ] ; then
    if [ -f $PIDFILE ] ; then # a tunnel is opened, so shut it
      kill `cat $PIDFILE`
      rm $PIDFILE
      echo "Tunnel has been closed."
    else # no PID file exists, must not have a tunnel opened.
      echo "Cannot find a PID file $PIDFILE.  Are you sure a tunnel is open?"
    fi

  else # neither start nor stop is passed
    echo "Use \"$0 start\" to start a proxy, or \"$0 stop\" to stop the proxy."
  fi

else # no arguements were passed
  echo "Use \"$0 start\" to start a proxy, or \"$0 stop\" to stop the proxy."
fi
{% endcodeblock %}

### Firefox Configuration ###

I used the [FoxyProxy Basic][] addon for Firefox to easily switch between proxied or no-proxy modes.  Use a socks version 5 connection type when setting up the proxy, to localhost and port 8080 (or whatever you want, just change the script).

[FoxyProxy Basic]:https://addons.mozilla.org/en-US/firefox/addon/15023/

If you don't want to use an addon, you can also enable the proxy manually through the `about:config` screen by following [this helpful tutorial][].

[this helpful tutorial]:http://wiki.freaks-unidos.net/weblogs/azul/firefox-ssh-tunnel

### Chromium Configuration ###

Here, as with Firefox, you can enable the proxy manually or with addons to help manage the proxy switching for you.  The manual configuration is under `Wrench Button` &rarr; `Options` &rarr; `Under the Hood` &rarr; `Change Proxy Settings`.  Set up localhost and port 8080 (or change the port to whatever you want, changing the script as well) using a Socks version 5 connection (leave the others blank) and you're golden.

Many extensions exist to help manage proxy settings in the extensions library.  As a side note which I find unsurprising, many are maintained by authors with Chinese looking names.
