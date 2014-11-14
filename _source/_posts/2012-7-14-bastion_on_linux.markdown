---
layout: post
title: Bastion on Linux
time: '15:00'
comments: true
categories: [Arch, Bastion, Linux, games]
---

I just finished playing through the main story line of [Bastion][].  It was an
amazing game, especially the music.

[Bastion]:http://en.wikipedia.org/wiki/Bastion_(video_game)

Getting the best performance out of it on my [Arch][] Linux box was, however,
a bit difficult, so with this short post I want to let others avoid a bit of
frustration and enjoy the game to its fullest.

[Arch]:http://www.archlinux.org

+ First, make absolutely sure you have compositing[^1] turned off.  With it
  on, the video would stutter horribly (1 or less FPS) at some random point in
  the game.  Switching to a different virtual desktop would temporarily fix
  the issue, but about 1/3 of the time this would crash the game.

  How do you know if you have compositing turned on?  If your desktop uses any
  animation or transparency, odds are you do.

+ Ensure you have the latest *stable* drivers for your video card.

+ If the game crawls in scenes with rain or snow, turn it off in the game's
  source files.  On my machine, the game was installed to
  `/opt/games/Bastion`.  From that directory, run the following search and
  replace command as root (or via sudo):

  {% codeblock lang:bash %}
  sed -i 's/MaxFlyers=".*/MaxFlyers="0"/g' Content/Game/FlyerSettings.xml
  {% endcodeblock %}

  {% comment %}
  *Fixing syntax codeblock in vim*
  {% endcomment %}

If you haven't had a chance to play this great game yet, give it a try.

Other (Linux specific) tweaks applicable for gaming:

+ Arch Wiki [Gaming](https://wiki.archlinux.org/index.php/Gaming)
+ Windows API implementation for Linux: [Wine](http://appdb.winehq.org/)

[^1]: To turn this off, search on Google for "compositing" along with your
    Linux distribution and / or window manager.  For me (Arch Linux, xmonad),
    I made sure that `xcompmgr` wasn't called in my `.xinitrc`.

