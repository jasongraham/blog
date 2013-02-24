---
layout: post
title: Generating Gravatar Favicons for Jekyll
comments: true
time: '15:05'
categories: [blog, favicon, gravatar, jekyll]
---

Today I decided that my previous [favicon][] used on this site was incredibly boring.  I like the way that [Wordpress.com][] handles making their favicons using their [Gravatar][] service, and I decided that I could be cool and use that too!

[favicon]:http://en.wikipedia.org/wiki/Favicon
[Wordpress.com]:http://wordpress.com
[Gravatar]:http://gravatar.com

Since [Jekyll][] is a static blog generator, I have to generate the favicon when I update the site, rather than on page loads.  I wrote a script that pulls my Gravatar and converts it to the appropriate format.  It depends on the [netpbm][] packages, available in most Linux distributions.

[Jekyll]:http://github.com/mojombo/jekyll
[netpbm]:http://netpbm.sourceforge.net/

{% codeblock lang:bash %}
#!/bin/bash
#
# This script downloads my gravatar image and uses it to create the
# favicon for the website.

# Replace 'HASH' with the value specific to your email address.
# Also, I pull the biggest version of the image available (256x256),
# but this probably isn't necessary.
wget http://www.gravatar.com/avatar/HASH?s=256 -O /tmp/favicon.jpg

convert /tmp/favicon.jpg ./favicon.png

# below is adapted from
# http://www.brennan.id.au/13-Apache_Web_Server.html#favicon
pngtopnm -mix ./favicon.png > tmp_logo.pnm

rm -f favicon.ico

pnmscale -xsize=32 -ysize=32 ./tmp_logo.pnm > tmp_logo32.ppm
pnmscale -xsize=16 -ysize=16 ./tmp_logo.pnm > tmp_logo16.ppm

pnmquant 256 tmp_logo32.ppm > tmp_logo32x32.ppm
pnmquant 256 tmp_logo16.ppm > tmp_logo16x16.ppm

ppmtowinicon -output ./favicon.ico tmp_logo16x16.ppm tmp_logo32x32.ppm

rm -f tmp_logo* favicon.png

{% endcodeblock %}

From there, I can include this in my [deploy script][] which updates the contents on the site.

[deploy script]:http://code.the-graham.com/blog/tree/deploy.sh
