---
layout: post
title: Router Died
comments: true
time: '9:30'
categories: [blog, router, server]
---

{% img right http://community.spiceworks.com/images/products/0002/3622/wrt54gl.jpg 150 'RIP' %}

Yesterday my wife and I mourned the passing of Bobby, or as he was better known to the world, [Linksys WRT54G][].  He has been my faithful router and friend for nearly 5 years now, and for the entire time that I've known him, his only thoughts were to serve...packets.  Indeed, he will be missed.

On an unrelated note, I bought a new router yesterday.  Rodney (I enjoy using router names that are easy to yell at), is a [Linksys WRT160n][], and has bravely stepped up to take Bobby's place.  I had planned to make my next router wireless N with 1Gb wired transfer, but decided to hold off on that, seeing as routers which actually implement the [recently finalized][], full [wireless N][] specs aren't on the market yet, and I really have no need for 1Gb connections.  Maybe after Rodney bites the dust, or I build a media center PC I'll get one.

{% img left http://www.noadapterneeded.com/images/linksys-wireless-router.jpg 150 150 'Rodney' %}

One important selection criteria for routers, at least for me, is the ability to flash [dd-wrt][], an alternative, Linux based, open-source firmware.  It turns out that many cheap-ish routers can support many more features than the built in firmware allows, so for me, this is a cheaper way to get a [fully featured][] device.  They have a nice [list of supported devices][], and well written instructions on how to upgrade and configure afterwards.

I *strongly* recommend doing some searching and reading around their wiki before starting: I nearly bricked poor Rodney yesterday because I was following some generic instructions rather than the hardware specific ones.

[Linksys WRT54G]:http://www.linksys.com/servlet/Satellite?c=L_Product_C2&childpagename=US%2FLayout&cid=1149562300349&pagename=Linksys%2FCommon%2FVisitorWrapper

[Linksys WRT160n]:http://www.linksysbycisco.com/US/en/products/WRT160N

[recently finalized]:http://www.engadget.com/2009/09/12/its-official-802-11n-standard-finalized-after-a-mere-seven-yea/

[wireless N]:http://en.wikipedia.org/wiki/IEEE_802.11n-2009

[dd-wrt]:http://dd-wrt.com/site/content/about

[fully featured]:http://www.dd-wrt.com/wiki/index.php/What_is_DD-WRT%3F#File_Versions

[list of supported devices]:http://dd-wrt.com/site/support/router-database
