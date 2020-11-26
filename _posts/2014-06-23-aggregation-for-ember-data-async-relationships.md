---
layout: post
title: "Aggregation for Ember Data Async Relationships"
date: 2014-06-23 17:12:21 +0200
permalink: /blog/2014/06/23/aggregation-for-ember-data-async-relationships/
tags: [Ember.js]
---

Ember.js has [computed properties](http://emberjs.com/guides/object-model/computed-properties-and-aggregate-data/) for aggregating data which are automatically re-calculated when the source data set changes. But what if we want to display aggregate data from an async relationship in a list view? We might have the following:

```coffeescript
App.Invoice = DS.Model.extend
  lineItems: DS.hasMany('lineItem', { async: true })

  lineItemAmounts: Ember.computed.mapBy('lineItems', 'amount')
  amount: Ember.computed.sum('lineItemAmounts')
```

Each `App.Invoice` has a bunch of `App.LineItem`s, and calculates its own amount by summing up their individual amounts. However, each time an invoice is rendered, all of the line items will be fetched from the server in order to make this calculation.

What we can do instead is send the aggregate figure from the server along with the invoice. It might be calculated on the fly (with a `group by` query, for example), or maintained along with the invoice. Here's an approach that does exactly this, but will switch to using the real ember data objects once they have been loaded:

```coffeescript
App.Invoice = DS.Model.extend
  subtotal: DS.attr('number')
  lineItems: DS.hasMany('lineItem', { async: true })

  lineItemAmounts: Ember.computed.mapBy('lineItems', 'amount')
  lineItemAmountsSum: Ember.computed.sum('lineItemAmounts')

  # Uses subtotal until async lineItems become available
  amount: (->
    if @cacheFor('lineItems')
      @get('lineItemAmountsSum')
    else
      @get('subtotal')
  ).property('lineItems.@each.amount')
```

In this case `subtotal` is the aggregated amount from the server.
