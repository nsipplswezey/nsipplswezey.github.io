---
layout: post
title:  "Repositioning a canvas with touch listeners"
date:   2015-02-12 19:15:37
categories: technical posts
---

In the previous blog post, I showed an iFrame with a canvas element in it, that listens for mobile touch events. It was pulled from a great [MDN tutorial](https://developer.mozilla.org/en-US/docs/Web/Guide/Events/Touch_events)

Let's try to reposition that canvas element so that instead of having it in the top left of the webview, we could have it positioned on the bottom right of our webview.

We can reposition any HTML element using CSS.


```html
<canvas id="canvas" width="600" height="600" style="border:solid black 1px; bottom:0; right:0; position:fixed; ">
```

Here's the simplest example, using inline css.

I'll come back to this blog post with a nice simple visual example.
When you see it, you'll realize that by repositioning the canvas element, we've actually completely broken its touch functionality.


Which raises the question:
How does this nice touchable canvas element actually work?
