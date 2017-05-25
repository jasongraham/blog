---
layout: post
title: "How to use systemd timers"
date: 2013-03-06 19:21
comments: true
categories: [Arch, Linux, systemd, cron, howto]
---

I was setting up some scripts to run backups recently, and decided I would try
to set them up to use [systemd timers][] rather than the more familiar to me
[cron jobs][].

[systemd timers]:https://fedoraproject.org/wiki/User:Johannbg/QA/Systemd/Systemd.timer
[cron jobs]:https://en.wikipedia.org/wiki/Cron

As I went about trying to set them up, I had the hardest time, since it seems
like the required information is spread around in various places.  I wanted to
record what I did so firstly, I can remember, but also so that others don't
have to go searching as far and wide as I did.

There are additional options associated with the each step I mention below, but
this is the bare minimum to get started.  Look at the man pages for
`systemd.service`, `systemd.timer`, and `systemd.target` for all that you can
do with them.

* make me a table of contents
{:toc}

## Running a Single Script ##

Let's say you have a script `/usr/local/bin/myscript` that you want to run
every hour.

### Service File ###

First, create a service file, and put it wherever it goes on your Linux distribution (on Arch, it is either `/etc/systemd/system/` or `/usr/lib/systemd/system`).

{% codeblock myscript.service lang:bash %}
[Unit]
Description=MyScript

[Service]
ExecStart=/usr/local/bin/myscript
{% endcodeblock %}

{% comment %}
[][] fix syntax highlighting in editor
{% endcomment %}

### Timer File ###

Next, create a timer file, and put it also in the same directory as the service file above.

{% codeblock myscript.timer lang:bash %}
[Unit]
Description=Runs myscript every hour

[Timer]
# Time to wait after booting before we run first time
OnBootSec=10min
# Time between running each consecutive time
OnUnitActiveSec=1h
Unit=myscript.service

[Install]
WantedBy=multi-user.target
{% endcodeblock %}

{% comment %}
[][] fix syntax highlighting in editor
{% endcomment %}

### Enable / Start ###

Rather than starting / enabling the service file, you use the timer.

{% codeblock lang:bash %}
# Start timer, as root
systemctl start myscript.timer
# Enable timer to start at boot
systemctl enable myscript.timer
{% endcodeblock %}

## Running Multiple Scripts on the Same Timer ##

Now let's say there a bunch of scripts you want to run all at the same time.
In this case, you will want make a couple changes on the above formula.

### Service Files ###

Create the service files to run your scripts as I [showed previously](#service-file),
but include the following section at the end of each service file.

{% codeblock lang:bash %}
[Install]
WantedBy=mytimer.target
{% endcodeblock %}

{% comment %}
[][] fix syntax highlighting in editor
{% endcomment %}

If there is any ordering dependency in your service files, be sure you specify
it with the `After=something.service` and/or `Before=whatever.service`
parameters within the `Description` section.

Alternatively (and perhaps more simply), create a wrapper script that runs the
appropriate commands in the correct order, and use the wrapper in your service
file.

### Timer File ###

You only need a single timer file.  Create `mytimer.timer`, as I 
[outlined above](#timer-file-1).

### Target File ###

You can create the target that all these scripts depend upon.

{% codeblock mytimer.target lang:bash %}
[Unit]
Description=Mytimer
# Lots more stuff could go here, but it's situational.
# Look at systemd.unit man page.
{% endcodeblock %}

{% comment %}
[][] fix syntax highlighting in editor
{% endcomment %}

### Enable / Start ###

You need to enable each of the service files, as well as the timer.

{% codeblock lang:bash %}
systemctl enable script1.service
systemctl enable script2.service
...
systemctl enable mytimer.timer
systemctl start mytimer.service
{% endcodeblock %}

Good luck.
