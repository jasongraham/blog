---
layout: post
title: Wordpress and Child Themes&#58; wish I'd found them sooner
comments: true
time: '10:30'
categories: [code, themes, wordpress]
---

Yesterday, [Wordpress 3.0][] was released.  Out of curiosity I went to check out what new features they had, when I discovered something totally unrelated to the new release: [child themes][].

[Wordpress 3.0]:http://wordpress.org/development/2010/06/thelonious/
[child themes]:http://codex.wordpress.org/Child_Themes

My wife and I still use Wordpress for our jointly-written, family oriented blog, since the [Jekyll][] framework I prefer for blogging is command line intensive.  One of my biggest pet-peeves regarding Wordpress themes was that any edits you made to customize your theme would be _completely_ wiped out whenever the theme was updated.  So between my predilection for tinkering and the difficulty of arriving at something both my wife and I agree on aesthetically, it's aggravating to have my careful changes forgotten each update.

[Jekyll]:http://github.com/mojombo/jekyll

So [child themes][], to me, are a wonderful idea.  Essentially, they are glorified [diffs][] that are automatically patched into the other theme, allowing you to overwrite or add CSS or php function without needing to change the original code.  Most importantly, they allow you to update the original theme without removing the changes made by the child.

[diffs]:http://en.wikipedia.org/wiki/Diff

So I made my own child theme based on the new default for Wordpress, [Twenty Ten][].  The _very_ simple changes essentially amount to the following:

+ color changes
+ Increase the font size in the navigation bar and title.
+ Separate posts and comments with border "bubbles"
+ Increase the width of the content area
+ In the sidebar, create a bit more separation between the widget titles and their content.

[Twenty Ten]:http://en.blog.wordpress.com/2010/04/26/new-theme-twenty-ten/

So if you have any need for Wordpress, I highly recommend the [tutorials][] included at the bottom of the previously linked page for [child themes][].  If I think of any other excuses to warp the theme, maybe I'll get to play with adding some php functions.

[tutorials]:http://codex.wordpress.org/Child_Themes#Resources
