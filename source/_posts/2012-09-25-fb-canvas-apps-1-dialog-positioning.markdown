---
layout: post
title: "FB Canvas Apps #1: Dialog Positioning"
date: 2012-09-25 16:33
comments: true
categories: [Facebook, jQuery, CoffeeScript, ColorBox]
---

Having worked on several Facebook applications, I've built up a few tricks to solve some common problems. This one shows how to get a dialog (jQuery UI, in this case) to appear centered in the browser rather than centered in the canvas iFrame.

The problem is caused by Facebook resizing the canvas iFrame to accommodate the entire page. When a dialog is displayed inside the canvas application, it is displayed vertically centered inside that iFrame, which can cause the current browser scroll position to jump up or down accordingly.

The fix is to call Facebook's [`FB.Canvas.getPageInfo`](http://developers.facebook.com/docs/reference/javascript/FB.Canvas.getPageInfo/) JS API method, to position the dialog relative to the user's current browser scroll position.

Here is an implementation which uses [CoffeeScript](http://coffeescript.org/) and extends the [jQuery UI dialog widget](http://jqueryui.com/demos/dialog/).

``` coffeescript Facebook Initialization
$ ->
  # Async Facebook initialisation
  window.fbAsyncInit = ->
    FB.init({ appId: 'APP_ID', status: true, cookie: true, xfbml: true })
    FB.Canvas.setAutoGrow()

    # Allows other components to hook into this event with $(document).on('fb:initialized', -> ...)
    $(document).trigger('fb:initialized')
```


``` coffeescript The Dialog Widget
CanvasDialog =
  # Some example default dialog options
  options: {
    width: 600,
    modal: true,
    autoOpen: true
  }

  _init: ->
    if @_inCanvas
      if this._fbLoaded()
        this._showDialogInCanvas()
      else
        $(document).on 'fb:initialized', => this._showDialogInCanvas()
    else
      this._showDialog()

  # Displays the dialog positioned correctly inside the FB canvas iFrame
  _showDialogInCanvas: ->
    FB.Canvas.getPageInfo (info) =>
      @options.position = ['center', info.scrollTop + 50]
      this._showDialog()

  # Calls the jQuery UI dialog's original init method
  _showDialog: -> $.ui.dialog.prototype._init.apply(this, arguments)

  # True if dialog is being loading inside the FB iFrame (any iFrame for that matter)
  _inCanvas: top != self

  # True if FB javascript API has been loaded
  _fbLoaded: -> typeof FB != 'undefined'


$.widget "ui.canvasDialog", $.ui.dialog, CanvasDialog
```

``` coffeescript Example Usage
$('#canvasDialog').canvasDialog()
```

### Notes

* By triggering an event when Facebook is initialized, the `CanvasDialog` is able to wait for the Facebook JS API to become available if it has not yet been loaded.

* In development it can be useful to work outside the Facebook environment. The `CanvasDialog` detects this and shows the dialog without attempting to call the Facebook API.

* In the past I've often used the excellent and well maintained [ColorBox](http://www.jacklmoore.com/colorbox) library for dialogs. I note that it too has been [updated](https://github.com/jackmoore/colorbox/pull/93) with similar functionalilty.