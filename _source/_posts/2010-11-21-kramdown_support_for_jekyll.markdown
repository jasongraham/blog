---
layout: post
title: kramdown support for Jekyll
comments: true
time: '10:24'
updated: '2010-12-9'
categories: [blog, code, coderay, jekyll, kramdown]
---


**Update Dec. 9th 2010**: Added some instructions on kramdown and Jekyll, (optionally with [Coderay][]).  [Jump down][] to it.

**Update Nov. 11th 2010**: My changes have been merged into the mainline branch, which can be found on [Github][Jekyll].

The last several months I've been fiddling around with different [Markdown][] converters used with [Jekyll][Jekyll readme].  I like [Maruku's][maruku] enhanced feature set (footnotes are nice), but was dissapointed with the execution speed.  I briefly played around with [Rdiscount][], which is *much* faster, but I missed the extra features of Maruku.

[Markdown]:http://daringfireball.net/projects/markdown/
[Jekyll]:https://github.com/mojombo/jekyll
[Jekyll readme]:https://github.com/mojombo/jekyll#readme
[maruku]:http://maruku.rubyforge.org/maruku.html
[Rdiscount]:https://github.com/rtomayko/rdiscount#readme
[Coderay]:http://coderay.rubychan.de/
[Jump Down]:#using-kramdown-and-jekyll

I then discovered [kramdown][].  It seems like a happy middle between Maruku and Rdiscount [performance-wise][performance], while containing a feature set even larger than that of Maruku.  Unfortunately, it isn't yet supported by Jekyll, though it had been a [long requested][] feature.

[kramdown]:http://kramdown.rubyforge.org/
[performance]:http://kramdown.rubyforge.org/tests.html
[long requested]:https://github.com/mojombo/jekyll/issuesearch?state=open&q=kramdown#issue/175

I waited for a couple months, and then decided to fork Jekyll and add the functionality myself.  At the time, I was unsure how to support [options][] passed to kramdown's HTML parser, so I left that part off to work on later.

[options]:http://kramdown.rubyforge.org/converter/html.html#options

Another coder apparently had the same idea as me at almost exactly the same time, and created [his own fork][] with the same functionality, but also didn't add the options passing.

[his own fork]:https://github.com/digitalsanctum/jekyll/tree/add_kramdown_support

With the start of Thanksgiving break, I've finally had some time to tack on the options functionality.  It goes into the [site configuration YAML file][config] just like Maruku options or Rdiscount extensions would.

[config]:https://github.com/mojombo/jekyll/wiki/Configuration

Hopefully, the fork will be pulled into the main branch, though I may need to clean up the options passing portion, it admittedly feels a bit hackish.  I'm a n00b ruby coder, so my excuse is that I don't know any better.  Fixes or suggestions are welcome though.

### Using kramdown and Jekyll ###

First, be sure you are using my branch of Jekyll (or v0.8.0 or higher in the mainline).

To use kramdown with Jekyll, you will need the following in your `_config.yml` file.

{% codeblock %}
...
markdown: kramdown
...
{% endcodeblock %}

See the Jekyll wiki [configuration page][config] for more of the options.

#### Adding Coderay ####

If you are wanting to additionally use Coderay for codeblocking, then your `_config.yml` file must include the following.

{% codeblock %}
...
markdown: kramdown
kramdown:
  use_coderay: true
...
{% endcodeblock %}

You may use a kramdown [block attribute][] within your markdown files to specify how you want Coderay to handle the codeblocking for that block.

{% codeblock %}
---
layout: default
title: Silly Test Post
---

This is a paragraph element before a codeblocked block of C code.

    int main(void) {
        printf("Hello world!");
        return 0;
    }
{:lang="c"}

Another paragraph element.

{% endcodeblock %}

[block attribute]:http://kramdown.rubyforge.org/syntax.html#block-ials
