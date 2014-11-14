---
layout: post
title: A slightly more paranoid than necessary Chromium browser setup
comments: true
description: Browsing safely with privacy and thwarting malware
categories: [chromium, howto, privacy, security]
---

{% img right http://upload.wikimedia.org/wikipedia/commons/5/5f/Chromium_11_Logo.svg 246 246 %}

Google's new web browser, [Chrome][], is a very fast browser.  My wife especially likes it, for this reason, as well as because [I really lock down Firefox][] in a way that sometimes can make it hard to use sites without some configuration.  Being somewhat of a privacy and anti-tracking fanatic, I asked myself how to keep our online privacy reasonably safe while she's online, while not causing the majority of sites to fail for her while she's watching videos, house hunting, or shopping.

[I really lock down Firefox]:/2009/12/14/howto_secure_firefox/

This page describes how I set up our computers (running [Ubuntu][]) to accomplish this.

#### Advantages of this setup ####
1. No history of anything persists after you close the browser window.
2. Compatibility with most websites using flash ([Youtube][], [Hulu][], . . . )

#### Disadvantages of this setup ####
1. No web history or bookmarks.
2. No automatic logins or remembered passwords.
3. No Ad blocking or JavaScript blocking.
4. No extensions.
5. Some sites that stream with flash, such as [Pandora][] and [NPR][] require local flash storage (depends, see below).
6. Google periodically updates their [Safe Browsing Filter][] so that a warning will pop up if you go to site with either known malware or scams, and this will doesn't allow these updates.  Be extra vigilent.
7. This does nothing to hide what website you're browsing from your ISP, the government,...  For that, you need something like [Tor][].

[Safe Browsing Filter]:http://dev.chromium.org/developers/design-documents/safebrowsing
[Tor]:https://www.torproject.org/

So there are serious disadvantages to securing the browser this way, and it is not perfect.  Read on for details.

### Install Chromium ###

[Chromium][] is the open-source software from which Google's [Chrome][] is made.  Google runs the Chromium project, and the two are for all purposes the same.  Chromium only lacks the branding.

You can choose between the [development version of Chromium from the PPA](https://launchpad.net/~chromium-daily/+archive/ppa), or the [stable version](https://launchpad.net/~chromium-daily/+archive/stable), which can be installed on Ubuntu Karmic or later with the following commands.

{% codeblock lang:bash %}
sudo add-apt-reposititory ppa:chromium-daily/ppa
#use ppa:chromium-daily/stable for the stable version instead

sudo apt-get update && sudo apt-get install chomium-browser
{% endcodeblock %}

### Kill all Flash Cookies ###

A [flash cookie][], or Local Shared Object, is a file a website stores on your computer, outside of the control of your browser settings.  It is different from a regular [cookie][].  They are associated with adobe flash, which is used by many websites.  Unfortunately, they are also [used to store tracking information](http://www.wired.com/epicenter/2009/08/you-deleted-your-cookies-think-again/), as well as back up data from regular cookies stored by your browser.

In most Linux distributions, Adobe Flash settings are stored in `~/.adobe` and the cookies themselves in `~/.macromedia` folders.  I have these simlinked to `/dev/null` (effectively a [black hole][]) so that anything trying to write to these folders doesn't get an error message, but nothing ever gets written to disk.

{% codeblock lang:bash %}
rm -rf ~/.adobe ~/.macromedia
ln -s /dev/null ~/.adobe
ln -s /dev/null ~/.macromedia
{% endcodeblock %}

You can alternatively set up a cron job to delete the contents of `~/.adobe` and
`~/.macromedia` so that you can get the benefits of being able to use sites
which require flash cookies, while simultaneously deleting them periodically
(and not having to trust a third party extension to delete them for you).  To
set up a [cron][] job to delete the contents of these folders every 5 minutes,
add the following line to your crontab, which you can access via `crontab -e`.
Replace `USER` with your username.

{% codeblock lang:bash %}
# m     h dom mon dow command
  */5   *  *   *   *  /bin/rm -rf /home/USER/.adobe/* /home/USER/.macromedia/*
{% endcodeblock %}

[cron]:http://en.wikipedia.org/wiki/Cron

### Open Chromium and Set Default Settings ###

Now we get to setup the default settings within Chromium.  Open the browser, but don't go anywhere.  We're going to make the settings directory read-only in a bit so that nothing can change the settings or write in new ones.  This will also stop Chromium from logging your browsing history or installing extensions.

0. If you wish to make Chromium the default browser for your system, you might as well do so now, since it asks you. Otherwise, say no.
1. In the upper right corner click on the wrench icon and pick "Options" from the menu.
2. Choose your starting behavior and homepage on the "Basics" tab.
3. In the personal tab,
	* Never Save Passwords.
	* Never save text from forms.
4. In the under the hood tab,
	* Disable "Show suggestions for navigation errors".  Google doesn't need to know what you're typing into the address bar.
	* Disable "Use a suggestion service . . . ".  Same as above.
	* Set cookie settings to Only Sites you Visit.  Will block a few third party tracking cookies.
	* Change the download settings to your preference.

### Set Browser to Default to Incognito Mode ###

Chromium comes with Incognito mode for "private" browsing, and won't remember any history or cookies while in this mode after you close the browser.  You may ask, if it does this already, why are we doing the other steps?  The steps we're using go farther than incognito mode is capable.  For example, when we're done, going to the history tab in Chromium will always show nothing.  I still have incognito mode enabled for some redundancy in protection.

To enable incognito mode, edit (as root) the file `/etc/chromium-browser/default` to appear like below.

{% codeblock lang:bash %}
# Default settings for chromium-browser. This file is sourced by /bin/sh from
# /usr/bin/chromium-browser

# Options to pass to chromium-browser
CHROMIUM_FLAGS="-incognito"
{% endcodeblock %}

### Make Chromium Profile Read-Only ###

This will make it so that the settings cannot be changed from within Chromium.  It will also freeze your browser history with nothing in it.

{% codeblock lang:bash %}
chmod a-w -R ~/.config/chromium/Default/*

# Note that on some installs, the folder 'Default'
# may not exist. In this case, use
# chmod a-w -R ~/.config/chromium/*
{% endcodeblock %}

Note that Chromium currently uses this folder to create sockets/symlinks to
temporary files, and will not run without allowing these to be made.  This is
why we leave the folder itself writable.

If you choose to do this and are sufficiently paranoid, whenever you update
Chromium, you should delete your profile, and then start over with a fresh
profile, and repeat all the setup above before once again making the profile
Read-Only.

### Disable Chromium's Disk Cache ###

This is optional.  All web browsers will [cache pages][] to improve the perceived speed of web browsing.  Removing the caching ability will make it harder for your browsing history to be determined.

Bear in mind this only prevents Chromium from using your hard drive as cache, and your RAM will still be used (and as best I can tell, no option to disable cacheing in RAM exists).

[cache pages]:http://en.wikipedia.org/wiki/Web_cache

{% codeblock lang:bash %}
rm -rf ~/.cache/chromium
ln -s /dev/null ~/.cache/chromium
{% endcodeblock %}

[Youtube]:http://www.youtube.com
[Hulu]:http://www.hulu.com
[Pandora]:http://www.pandora.com
[NPR]:http://npr.org
[Chrome]:http://www.google.com/chrome
[flash cookie]:http://en.wikipedia.org/wiki/Local_Shared_Object
[cookie]:http://en.wikipedia.org/wiki/HTTP_cookie
[Chromium]:http://www.chromium.org/Home
[Ubuntu]:http://www.ubuntu.com
[black hole]:http://en.wikipedia.org/wiki//dev/null
