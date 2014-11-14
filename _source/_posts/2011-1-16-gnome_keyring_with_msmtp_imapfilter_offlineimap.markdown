---
layout: post
title: Using Gnome Keyring with msmtp, offlineimap, and imapfilter via cron
time: '10:40'
comments: true
updated: '2012-1-28'
categories: [cron, email, gnome-keyring, imapfilter, msmtp, offlineimap, passwords]
---

**Update Jan 28th, 2012**: Simplified [cron](#cron) portion.  
**Update Jan 27th, 2011**: Added portion relavent to cron.  It's down at the
[bottom of the page][].
{: .update}

[bottom of the page]:#cron

In my [previous post][], I walked through setting up using [mutt][], [msmtp][], [offlineimap][], [imapfilter][], and [archivemail][] to work together.  As I wrote the post, I was reminded of how bothered I felt about having passwords for my email accounts in my configuration files.  For a single user computer, this isn't a huge problem, but I would still prefer keep my passwords out of plain text.

[previous post]:/2011/01/10/email_with_mutt_offlineimap_imapfilter_msmtp_archivemail/
[mutt]:http://www.mutt.org/
[msmtp]:http://msmtp.sourceforge.net/
[offlineimap]:https://github.com/jgoerzen/offlineimap/wiki
[imapfilter]:http://imapfilter.hellug.gr/
[archivemail]:http://archivemail.sourceforge.net/

Some searching online lead me to decide to use some pre-existing integration done using [gnome keyring][], which lets your store passwords in an encrypted form.

[gnome keyring]:http://en.wikipedia.org/wiki/GNOME_Keyring

A gnome-keyring patch for msmtp has [already been integrated][msmtp-gnome] into the `msmtp-gnome` package in the Ubuntu repositories, and a nice [script][msmtp-keyring-script] to add the appropriate data to the keyring exists.  Another great post I found explains how to [integrate offlineimap with gnome-keyring][offlineimap-gnome-keyring] with a python helper script.  It defines a couple functions that can pull the username and password directly within the configuration file.  I've updated my own configuration files and tutorial in the previous post.

[msmtp-gnome]:http://simple-and-basic.com/2008/10/using-msmtp-with-the-gnome-keyring.html
[msmtp-keyring-script]:https://github.com/gaizka/misc-scripts/raw/master/msmtp/msmtp-gnome-tool.py
[offlineimap-gnome-keyring]:http://www.clasohm.com/blog/one-entry?entry_id=90957

I didn't find any examples of using imapfilter with gnome-keyring, but I was reminded after seeing the [example extended settings file][imapfilter-ext-config] that since the configuration files are actually Lua scripts, we can use a slightly modified version of the python helper script used with offlineimap, and then simply pipe the input into the settings file to clean up.

[imapfilter-ext-config]:http://imapfilter.hellug.gr/sample.extend.lua.txt

{% codeblock imapfilter.py lang:python %}
#!/usr/bin/python
"""
File: imapfilter.py

Used to grab IMAP password information from
Gnome-Keyring.

Usage:
   ./imapfilter.py SERVER_NAME INFO

   where INFO is either "user" or "password", depending
   on the information you want.
"""

import sys
import gtk
import gnomekeyring as gkey

class Keyring(object):
    def __init__(self, name, server, protocol):
        self._name = name
        self._server = server
        self._protocol = protocol
        self._keyring = gkey.get_default_keyring_sync()

    def has_credentials(self):
        try:
            attrs = {"server": self._server, "protocol": self._protocol}
            items = gkey.find_items_sync(gkey.ITEM_NETWORK_PASSWORD, attrs)
            return len(items) > 0
        except gkey.DeniedError:
            return False

    def get_credentials(self):
        attrs = {"server": self._server, "protocol": self._protocol}
        items = gkey.find_items_sync(gkey.ITEM_NETWORK_PASSWORD, attrs)
        return (items[0].attributes["user"], items[0].secret)

    def set_credentials(self, (user, pw)):
        attrs = {
                "user": user,
                "server": self._server,
                "protocol": self._protocol,
            }
        gkey.item_create_sync(gkey.get_default_keyring_sync(),
                gkey.ITEM_NETWORK_PASSWORD, self._name, attrs, pw, True)

def main(server, info):
    keyring = Keyring("offlineimap " + server, server, "imap")
    (username, password) = keyring.get_credentials()

    if info == "user":
        print(username)
    elif info == "password":
        print(password)
    else: # neither user nor pass
        sys.exit(-2)

if __name__ == '__main__':
    server = sys.argv[1]
    info = sys.argv[2]
    main(server, info)
{% endcodeblock %}

{% codeblock lang:lua %}
-------------------------------------
-- Partial Imapfilter configuration
-- example.lua
-------------------------------------

-- Function strips trailing newlines
-- piped back from imapfilter.py
function trim(s)
    return (s:gsub("^%s*(.-)%s*$", "%1"))
end

-- extract username and password from gnome-keyring
status, myuser = pipe_from('/path/to/imapfilter.py mail.example.com user')
myuser = trim(myuser)

status, mypass = pipe_from('/path/to/imapfilter.py mail.example.com password')
mypass = trim(mypass)

ACCOUNT = IMAP {
        server = 'mail.example.com',
        username = myuser,
        password = mypass,
}

{% endcodeblock %}

Finally, the trick to allowing cron to access the gnome-keyring is found in
supplying it with the right environmental variable (notably,
`DBUS_SESSION_BUS_ADDRESS`) so that offlineimap can talk with your already
authenticated session of gnome-keyring.  My original script to do this was
really kludgey, but I found a [simpler version][] of the script from an
individual who has a very similar setup.
{: #cron}

[simpler version]:http://dev.gentoo.org/~tomka/mail.html

Take the `export_x_info.sh` script below, and make it run on startup.  Note
that is must start after DBus.

{% codeblock export_x_info.sh lang:bash %}
#!/bin/bash
# file: export_x_info.sh
# Export the dbus session address on startup so it can be used by cron
touch $HOME/.Xdbus
chmod 600 $HOME/.Xdbus
env | grep DBUS_SESSION_BUS_ADDRESS > $HOME/.Xdbus
echo 'export DBUS_SESSION_BUS_ADDRESS' >> $HOME/.Xdbus
{% endcodeblock %}

If the script is called correctly, you will see a file `~/.Xdbus` created,
that looks like the following:

{% codeblock lang:bash %}
DBUS_SESSION_BUS_ADDRESS=unix:abstract=/tmp/dbus-some-long-complicated-name
export DBUS_SESSION_BUS_ADDRESS
{% endcodeblock %}

Then, you can use cron to run the following:

{% codeblock pullmail.sh lang:bash %}
#!/bin/bash
# file: pullmail.sh
source $HOME/.Xdbus; /usr/bin/offlineimap
{% endcodeblock %}

Congradulations!  You now have gnome-keyring managing your passwords for you,
and your mail is being fetched automatically.
