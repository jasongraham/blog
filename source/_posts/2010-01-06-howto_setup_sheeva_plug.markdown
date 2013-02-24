---
layout: post
title: How to Setup a Sheeva Plug
comments: true
description: Fixing apt-get, fixing dns, adding users, setting up lighttpd and cgit
categories: [blog, code, howto, server, sheevaplug]
---

{% img right http://upload.wikimedia.org/wikipedia/commons/thumb/7/7b/SheevaPlug_with_external_drive_enclosure.jpg/300px-SheevaPlug_with_external_drive_enclosure.jpg %}

My Christmas present to myself was a new computer of sorts: a [Sheeva Plug][] computer from Marvel.  Its a tiny, cheap, wall-wart style computer, and much lower power than the previous dual-core machine I had running this server.

I wanted to document the steps that I used to get the sucker up and running, so I can remember them later.  For some of the steps, I am referring to the [Computing Plugs wiki][], itself running off of a Sheeva plug.

So here is documentation for myself on what I did, and for anyone else who finds it useful.

_EDIT: I have since reinstalled the OS with [debian][], using the [excellent tutorial][] by Martin Michlmayr.  The following tutorial assumes you have the default Ubuntu 9.04 that comes with the plug._

### Contents ###
+ This will be replaced with a table of contents
{:toc}

---

### Fix the DNS issue ###

The Sheeva Plug won't use the DNS servers set within my DHCP router without the following line being commented in the file `/etc/dhcp3/dhclient.conf`:

{% codeblock lang:bash %}
supersede domain-name-servers 127.0.0.1;
{% endcodeblock %}

---

### Get apt-get working ###

See [this article](http://computingplugs.com/index.php/Fixing_apt-get) for instructions and options of what to do.

From there, run `apt-get update && apt-get dist-upgrade` to update the software on the server.


---

### Add a non-root default user ###

I don't want to use root to log in or do much of anything directly.  I want to use the typical default Ubuntu `sudo` techniques.  Thus, I'll add myself another user, make and add him to the admin group, and disable the ability to log in as root remotely or via `su`.

{% codeblock lang:bash %}
# as root
useradd -d /home/USERNAME -m USERNAME -p PASSWORD --shell /bin/bash

groupadd -g ### --system admin
{% endcodeblock %}

The `-g ###` above is the number you want for your admin group.  If you don't care, then leave off that part.

Make the line is `/etc/group` with admin look as `admin:x:###:USERNAME`, where `###` is just the number that you chose, or that the system assigned.

Enter `visudo` to edit the sudoers file, and add:

{% codeblock lang:text %}
# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL
{% endcodeblock %}

Verify that you can now execute a command as sudo successfully, then nuke the root user.

{% codeblock lang:bash %}
# as the new user
sudo pwd # make sure sudo is working

# if it is, you can safely remove the ability to log in as root
sudo passwd -l root
{% endcodeblock %}

Finally, remove the ability to log in remotely as root.  In `/etc/ssh/sshd_config`, change

{% codeblock lang:bash %}
LoginGraceTime 20 #(length of time system in seconds server will wait for a response during login)
PermitRootLogin no
{% endcodeblock %}

Remotely logging in to the server can now only be done through the new user, and root commands and functionality are preserved through sudo.  For added security, [use ssh keys][] rather than passwords, and / or move sshd to a non-standard port.

---

### Install Additional Software ###

First, lets get some basic software that I consider useful.

+ `man-db`: Serious oversight by the creators to neglect the manual pages in the default install.
+ `vim-nox`: My text editor of choice is [vim][].
+ `git-core`: Needed for the code repository management through gitosis later.
+ `vnstat`: A bandwidth logging tool.  Also, a cron job to update the stats hourly should be added.
{% codeblock lang:bash %}
apt-get install man-db vim-nox git-core vnstat

# Start vnstat logging
# Same command should be put in a cron job hourly
vnstat -u -i eth0
{% endcodeblock %}
+ [Install gitosis][] to manage git.
+ After installing gitosis, deploy `git-daemon` to those made public through gitosis for easy cloning.  For Ubuntu 9.04 at least, mind that there is a bug in the `git-daemon-run` package, and to use [this fix][].

I run several services using the server[^1], including this webpage, and my [code repository][] through [Git][].  Some generic codeblocks:

[^1]: I used to run all these through the server, but have since started using
      [Linode][] for that purpose.

[Linode]:https://www.linode.com/

#### Webserver through lighttpd ####

[Lighttpd](http://www.lighttpd.net/) (pronounced "lighty") is a smaller, lighter webserver especially well suited for this plug with its smaller amount of memory (512M), and the fact that its mostly just static pages.

{% codeblock lang:bash %}
# all commands as root
apt-get install lighttpd

# cgi support needed by cgit later
lighttpd-enable-mod cgi

/etc/init.d/lighttpd force-reload
{% endcodeblock %}

#### Setup cgit ####

[Cgit][] is what displays my [code repository][].  The install is fairly well documented.  For help with the lighttpd configuration, I found [these posts][] linking to [this pastebin][] helpful.

There are two places you could put the git repository: as a subdomain or not (ie, www.example.com/git or git.example.com).

{% codeblock lang:lighttpd %}
# cgit setup as www.example.com/git
$HTTP["url"] =~ "^/git" {
  alias.url += (
    "/git/" => "/path/toward/cgit_cgi/",
  )
  cgi.assign = ( "/path/to/cgit.cgi" => "" )
}

url.rewrite-once = (
	"^/git/([^?/]+/[^?]*)?(?:\?(.*))?$"   => "/git/cgit.cgi?url=$1&$2",
)
{% endcodeblock %}

{% codeblock lang:lighttpd %}
# cgit setup as git.example.com
$HTTP["host"] =~ "git.example.com" {
  server.document-root = "/path/toward/cgit.cgi/"
  cgi.assign = ( "/path/to/cgit.cgi" => "" )
  url.rewrite-once = ( 
    "/([^?/]+/[^?]*)?(?:\?(.*))?$"   => "/cgit.cgi?url=$1&$2",
  )
}
{% endcodeblock %}

To make the repositories managed by gitosis readable by the webserver, I added the webserver user to the gitosis group (git in the [Install gitosis] tutorial).

I also finally made myself a setup to mirror a few repositories, and tested it with mirroring gitosis and cgit.  Creating the repository with `git clone --mirror` was easy, and I modified a cron job I found online (but can't find now, I'm disappointed I can't credit the source) to iterate through the directory where I put them and automatically update them.

{% codeblock lang:bash %}
MIRRORDIRS=/path/to/repo1.git/:/path/to/rep2.git/

# Use ':' as the internal field separator due to above construction
IFS=:

for gitdir in $MIRRORDIRS; do
    echo $gitdir
    pushd $gitdir
    git fetch
    popd
done
{% endcodeblock %}

Finally, viewing the actual files within the tree view of cgit can be enhanced by performing some syntax codeblocking through python-pygments, and then adding the following file (instructions in the header).  Again, I want to credit the site I found this on, but it was from back when I was using the old server, and I can't find it.  Author's email is at least there.

{% codeblock lang:python %}
#!/usr/bin/env python
#
# 10/15/2009 - seb@dbzteam.org
#
# Use Pygments (http://pygments.org/) to codeblock files managed with git 
# and web interfaced with cgit (http://hjemli.net/git/cgit/).
#
# Install:
#
# 1- Install python-pygments ( sudo apt-get install python-pygments )
# 2- Copy this script to /usr/local/bin/pygmentize_cgit.py (with exec rights)
# 3- Add this statement into the 'global settings' section of cgit 
#    configuration file /etc/cgitrc:
#      # Source code codeblocking
#      source-filter=/usr/local/bin/pygmentize_cgit.py
#
import sys
import pygments
import pygments.lexers
import pygments.formatters

def pygmentize(fn, in_stream=None, out_stream=None):
    # If not provided in_stream will be read from stdin and out_stream 
    # will be written to stdout.
    if in_stream is None:
        in_stream = sys.stdin
    if out_stream is None:
        out_stream = sys.stdout

    # Use pygments to codeblock in_stream.
    codeblock = True

    lexer = None
    try:
        lexer = pygments.lexers.get_lexer_for_filename(fn)
    except pygments.util.ClassNotFound:
        codeblock = False

    formatter = None
    try:
        formatter = pygments.formatters.get_formatter_by_name('html', 
                                                              noclasses=True, 
                                                              style='default')
    except pygments.util.ClassNotFound:
        codeblock = False

    if codeblock:
        pygments.codeblock(in_stream.read(), lexer, formatter, 
                           outfile=out_stream)
    else:
        # If sth went wrong or the file extension is no recognized only copy 
        # in_stream to out_stream without modifications.
        out_stream.write(in_stream.read())

if __name__ == '__main__':
    if len(sys.argv) != 2:
        sys.exit(1)
    pygmentize(sys.argv[1])
{% endcodeblock %}

[debian]:http://www.debian.org/
[excellent tutorial]:http://www.cyrius.com/debian/kirkwood/sheevaplug/
[Sheeva Plug]:http://www.marvell.com/products/embedded_processors/developer/kirkwood/sheevaplug.jsp
[Computing Plugs wiki]:http://computingplugs.com/index.php/Main_Page
[vim]:http://www.vim.org/
[code repository]:http://code.the-graham.com
[Git]:http://git-scm.com/
[use ssh keys]:https://help.ubuntu.com/community/SSH/OpenSSH/Keys
[Install gitosis]:http://scie.nti.st/2007/11/14/hosting-git-repositories-the-easy-and-secure-way
[Cgit]:http://hjemli.net/git/cgit/
[these posts]:http://redmine.lighttpd.net/boards/2/topics/2041
[this pastebin]:http://pastebin.com/f1305ed26
[this fix]:https://help.ubuntu.com/community/Git#Making%20available%20public%20cloning%20of%20the%20projects
