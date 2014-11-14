---
layout: post
title: How to Secure Firefox
comments: true
description: Securing Firefox to Protect your Privacy and Thwart Malware
categories: [firefox, howto, privacy, security]
---

{% img right https://people.mozilla.com/~faaborg/files/shiretoko/firefoxIcon/firefox-512-noshadow.png 228 228 %}

After leaving a post describing [how to set up Chromium Securely](/2009/12/14/secure_chromium_setup/), I figured that I would explain how I have Firefox setup in the same way.  This setup meets my own needs, so you may want to modify what I've done to better suit you.  I'll try to explain the pros and cons of each step along the way.

This post will deal with securing Firefox.  I'll leave another one in the future explaining what I do to speed it up.

Also, I use [Arch Linux][] as my operating system, but most of this guide is OS agnostic.

#### Advantages ####
1. Browsing information (cookies, cache, history) is deleted on browser exit.
2. Compatibility with most websites is maintained (provided you know how to adjust extension settings when there are problems).

#### Disadvantages ####
1. Initially, tweaking extensions for compatibility with your commonly used websites will take some time.
2. Some sites that stream with flash, such as [Pandora][] and [NPR][] require local flash storage.

Firefox comes installed by default on my Linux distribution of choice, so I leave you to figure out how to install it if you need to.

### Kill all Flash Cookies ###

A [flash cookie][], or Local Shared Object, is a file a website stores on your computer, outside of the control of your browser settings.  It is different from a regular [cookie][].  They are associated with adobe flash, which is used by many websites.  Unfortunately, they are also [used to store tracking information](http://www.wired.com/epicenter/2009/08/you-deleted-your-cookies-think-again/), as well as back up data from regular cookies stored by your browser.

* #### Symlinking to /dev/null ####

	In most Linux distributions, Adobe Flash settings are stored in `~/.adobe` and the cookies themselves in `~/.macromedia` folders.  I have these simlinked to `/dev/null` (effectively a [black hole][]) so that anything trying to write to these folders doesn't get an error message, but nothing ever gets written to disk.

{% codeblock lang:bash %}
rm -rf ~/.adobe ~/.macromedia
ln -s /dev/null ~/.adobe
ln -s /dev/null ~/.macromedia
{% endcodeblock %}

Every so often, I do listen to [NPR][] or another site that requires Flash cookies for streaming media storage.  When that happens, I simply delete the simlinks to use them, and then repeat the command set above when I'm done.


* #### With extensions ####

	For those of you using Windows, or those not wanting to mess with the command line to watch a video, the Firefox extension [BetterPrivacy][] allows you to a way to control Flash Cookies by deleting them on browser exit, at fixed intervals, or even if they haven't been changed for a given amount of time.  I recommend it over nothing.

* #### Deleting Flash Cookies via Cron ####

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

### Installing Privacy Extensions ###

Speaking of extensions, this is as good a place as any to start installing them.

Click on `Tools > Addons`, and under the Get Add-ons tab, search for and install the following:

1. [Adblock Plus](http://adblockplus.org/en/).  Not only will this remove the majority of adds of all kinds from pages, but it has the capacity to filter out known malware sites or tracking servers through a variety of [subscription lists](http://adblockplus.org/en/subscriptions).
2. [NoScript](http://noscript.net/).  Now we start getting into extension which can give you a headache if you're not careful.  NoScript will block a website from executing Java, JavaScript and Flash.  We do this because these will slow down web browsing, and they can also be used as methods to attack a browser to break into your computer or steal personal information.  Using this extension, you can selectively allow a site to execute some scripts, while not allowing others.  The disadvantage of this extension is that after you install it, as you travel to sites you know and trust, you will need to tell it to remember to allow certain scripts to run on this site.  If you would find this too much of a hassle, I this addon probably isn't for you.
3. [RequestPolicy](http://www.requestpolicy.com/).  Same disclaimer as with NoScript.  This extension compliments NoScript by requiring you to give permission now for a site to execute a script from _another_ site.  For example, many sites use [Google Analytics](http://www.google.com/analytics/) to see who comes to their sites, what they do there, where the come from on the web, what browser they're using . . . lots of information.  Sites do this by making you execute a script run by Google.  This is a somewhat tame example of what is called cross site scripting, but this could be used nefariously as well.  Besides wanting a bit more privacy while online, these scripts can be used to attack your computer / obtain your personal information.  The fewer of them that are allowed to run, the better.  You would be surprised how many scripts have nothing to do with the functionality of a website, and are simply watching where you go, what you click, and trying to serve you adds.
4. [User Agent Switcher](http://chrispederick.com/work/user-agent-switcher/).  This extension attempts to fool a website into believing that you're not using Firefox, but are actually some other browser.  I mostly use this in an attempt to spoof any malware specifically targetting Firefox to leave me alone, but it can also be used to make sites think you're something like the Google Search Bot program, or something equally exotic.

After you install these addons, close and restart your browser.  You will now be confronted with setting the defaults for all of these.  I find the documentation they provide to be pretty good to getting started, so I will let the extensions speak for themselves.

### Set Default Browser Settings

Now we can set some settings within Firefox itself that will enhance privacy settings.

Click on `Edit > Preferences` (Tools rather than Edit if you're in Windows), and head to the Privacy tab.  From here, you will want to do several things.

0. Uncheck the box saying "Accept third-party cookies". These are never needed, and are typically tracking related.
1. The easiest way to ensure privacy would be to tell Firefox to Never Remember History in the first drop box.  Unfortunately, this won't allow you to save any passwords in Firefox, so I don't use this.
2. Instead, I click on Settings on the right side, and have Firefox delete everything except saved passwords when it closes.

Next, go to the Security tab.  If you want to have Firefox remember passwords for certain sites, be sure to use a Master Password to protect (and encrypt) those stored passwords.  Its much easier to remember one password than all of them.  Conversely, it may be more secure to not store any of them at all.

### Disable Disk Cache by Editting about:config

Excepting the powerful extensions, my favorite feature of Firefox is that they expose the configuration settings if you want them.  They can be accessed by typing [about:config](about:config) into your address bar.  Frequently guides to [speed up Firefox][] will ask you to change settings in them, and are good ways to increase your browsing speed.  I suggest trying one to see if this helps you.

[speed up Firefox]:http://www.blogsdna.com/6522/how-to-speed-up-your-firefox-browser.htm

Though this, I disable disk caching and offline state saving so that Firefox doesn't spin up the hard drive on my laptop.  The keys you want to change are `browser.cache.disk.enable` and `browser.cache.offline.enable`, which I set to `false` (double-clicking on the key will change it).

---
Additional resources on more advanced topics such as ad and malware filtering proxies, Apparmor Profiles, and TOR proxies are left to the reader.

[Youtube]:http://www.youtube.com
[Firefox]:http://www.mozilla.com/firefox/
[Hulu]:http://www.hulu.com
[Pandora]:http://www.pandora.com
[NPR]:http://npr.org
[flash cookie]:http://en.wikipedia.org/wiki/Local_Shared_Object
[cookie]:http://en.wikipedia.org/wiki/HTTP_cookie
[Arch Linux]:https://www.archlinux.org
[black hole]:http://en.wikipedia.org/wiki//dev/null
[BetterPrivacy]:https://addons.mozilla.org/en-US/firefox/addon/6623
