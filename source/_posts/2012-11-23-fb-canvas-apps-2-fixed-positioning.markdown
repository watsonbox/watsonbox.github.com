---
layout: post
title: "FB Canvas Apps #2: Fixed Positioning"
date: 2012-11-23 13:32
comments: true
categories: [Facebook, jQuery, CoffeeScript]
---

Recently I was asked to add a fixed position 'tab' to the left side of a Facebook canvas app, for example like the [Get Satisfaction](https://getsatisfaction.com/) 'Community' tab.

Normally this would be a simple case of using `position: fixed`, but not in this case because Facebook canvas apps live inside iFrames. Facebook resizes this iFrame to fit its content, meaning that the main browser scrollbar is used to scroll around the page rather than the iFrame's scrollbar. However, fixed position elements are fixed relative to the iFrame, not the main document. Since the iFrame never actually scrolls, the fixed element stays put.

The solution is to use a little javascript to scroll the fixed element relative to the Facebook scroll position at a fixed interval. This gives a slightly different user experience but is the best approach I've found so far.

```coffeescript CoffeeScript/jQuery: pseudoFixed Method and Invocation
$.fn.pseudoFixed = ->
  top = this.position().top

  window.setInterval(
    => FB.Canvas.getPageInfo((info) => this.animate({ top: info.scrollTop + top })),
    1000
  )

$('[data-behavior~=pseudo-fixed]').pseudoFixed()
```

```html Example HTML Tab Element
<div data-behavior="pseudo-fixed" style="position: fixed; left: 0; top: 150px">...</div>
```