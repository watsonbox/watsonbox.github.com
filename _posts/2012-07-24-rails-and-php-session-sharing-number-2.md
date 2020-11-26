---
layout: post
title: "Rails & PHP Session Sharing #2"
date: 2012-07-24 16:37
permalink: /blog/2012/07/24/rails-and-php-session-sharing-number-2/
tags: [Ruby, Ruby on Rails, PHP]
---

Back in May I [posted](/blog/2012/05/01/sharing-session-between-rails-and-php/) on how I set up shared sessions between Rails and a legacy PHP app. Since then I've upgraded memcache-client to Mike Perham's replacement, [Dalli](https://github.com/mperham/dalli/). I took this opportunity to tidy up the PHP session store as well. Here is what I have now:

{% gist 3170415 %}

This is cleaner than what I had before and should be much more resistant to changes in Dalli, too.
