---
layout: post
title: What is it you do exactly? (Overview)
time: '21:20'
comments: true
categories: [DSP, communication, information, research, school]
---

I thought I'd start by pointing to some background material, for those interested, about the history of what is called "[Information Theory][]".  The founding father of the field is [Claude Shannon][], and his work is very important as to telling you what is possible when trying to move data from point A to point B.  As an aside, Shannon is actually a very interesting geek figure. Among other things, his masters thesis is the basis of modern digital logic circuits.  Definitely check out his Wikipedia page.

Unfortunately, Shannon and the rest of pure theory tells you nothing about how to accomplish these limits of possibility.  Thus, people have since been trying to figure out how to achieve this goal while simultaneously minimizing the power used to transmit information, and minimizing the probability of error as that data is sent.  Its a very complex problem, with lots of interesting subproblems as the exact circumstances of the transmission channel change.

Anyway, I have a classic example of a digital transmission system in the image below.  This could be thought of as a cell phone (with the whole part where the cell tower is involved conveniently removed) or perhaps a digital walkie-talky system.

There are many steps involved; each of the boxes is complicated enough to have a graduate level class dedicated to the 'basics' of its implementation.  But its easy enough to trace your way through the steps required.

<!-- more -->

{% img https://cacoo.com/diagrams/UBxf2qq5Hq4u2ZGR-0F230.png %}

A quick explanation:

* 	**Quantization**: Turning the [analog signal][] into a [digital signal][].

* 	**Source Coding**: This is data compression.  A good example would be a sending a fax.  If you split the page up into a bunch of tiny squares, and think of the black parts being '1's, and the white parts being '0's, then a typical document has a _lot_ more 0's than 1's.  So instead, the data is compressed (much like making zip folder of files on your computer) before being transmitted.  This makes the transmission as small as possible.

* 	**Channel Coding**: Any time you transmit data sometimes the message gets scrambled a bit in transit.  Think of driving along downtown in a big city.  As you move, the cell towers can loose sight of you for a little bit as you move past buildings.  If this break in transmission is short enough, the cell phone is smart enough to piece together the little bit of the signal that you missed because of this step.

	You can see this happen in the written language too.  For example:

	{% blockquote %}
	My comp_ter keys can be a bit sti_ky sometimes.
	{% endblockquote %}

	Your brain is able to put together the message based on the English dictionary and surrounding context.  In a nutshell, channel coding does more or less the same thing by providing a dictionary of sorts so the receiver can check what it hears against a list of codewords, and pick the best match.

* 	**Signal Modulation**: This is difficult to visualize.  This takes the codewords put together by the channel coding, and decides how to send them as signals through space.  For example, the your car radio can pick up FM ([frequency modulation][]) and AM ([amplitude modulation][]) channels, which are two such methods.

The rest of the chain involves undoing these steps in reverse order to get back to an approximation of the voice signal that went into the system in the first place.

[Information Theory]:http://en.wikipedia.org/wiki/Information_theory
[Claude Shannon]:http://en.wikipedia.org/wiki/Claude_E._Shannon
[analog signal]:http://en.wikipedia.org/wiki/Analog_signal
[digital signal]:http://en.wikipedia.org/wiki/Digital_signal
[bits]:http://en.wikipedia.org/wiki/Bit
[frequency modulation]:http://en.wikipedia.org/wiki/Frequency_modulation
[amplitude modulation]:http://en.wikipedia.org/wiki/Amplitude_modulation
[cacoo.com]:http://cacoo.com
