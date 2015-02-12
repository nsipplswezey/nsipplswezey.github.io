---
layout: post
title:  "Testing an iFrame in Jekyll"
date:   2015-02-10 19:15:37
categories: meta posts
---

If a great technical blog post is [short, reproducible and visual](http://nsipplswezey.github.io/meta/posts/2015/02/09/great-technical-posts.html) then what tools do we have to introduce reproducibility and visibility into our blogging?

Let's explore iFrames. An iFrame is an html element, which creates an inline frame used to embed another document within the current HTML document.

I think of them simply as a web-view inside a web-view.

How do we use an iFrame in a technical blog post?

If you're writing your blog post using markdown, then it turns out to be as easy as embedding the iFrame tag with a line return above and below it.

Check out this spectacular [MDN tutorial](https://developer.mozilla.org/en-US/docs/Web/Guide/Events/Touch_events) on using custom touch event listeners and canvas elements to enable touch interactions with mobile web views.

This example only works on a mobile device.

<iframe src="https://mdn.mozillademos.org/en-US/docs/Web/Guide/Events/Touch_events$samples/Example?revision=716919" height="800" width="600" marginwidth="0" marginheight="0" scrolling="no"></iframe>

Or how about Mike Bostock's D3 Box-Pplot animation of data from the [Michelson-Morley experiment](http://en.wikipedia.org/wiki/Michelson%E2%80%93Morley_experiment).

<iframe src="http://bl.ocks.org/mbostock/raw/4061502/0a200ddf998aa75dfdb1ff32e16b680a15e5cb01/" height="500" width="600" marginwidth="0" marginheight="0" scrolling="no"></iframe>

Great! Now we have a nice contained environment to demonstrate code visually!
