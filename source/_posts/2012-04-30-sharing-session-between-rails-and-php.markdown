---
layout: post
title: "Sharing session between Rails and PHP"
date: 2012-05-01 13:07
comments: true
categories: [Ruby, Rails, PHP]
---

_Update: Here's a [more elegant solution](/blog/2012/07/24/rails-and-php-session-sharing-number-2/) which uses Dalli instead of memcache-client._

Sometimes Rails and PHP applications must live side by side in harmony, which can mean sharing session data between them. Here's one way this can be achieved with very little modification on the PHP side. First make sure your PHP application is using [memcached](http://memcached.org/) for session storage:

``` ini php.ini
session.save_handler = memcache
session.save_path = "tcp://localhost:11211"
```

If you're interested in seeing how the PHP data gets serialized, take the value of the `PHPSESSID` cookie and perform a `GET` request in a memcached telnet session:

``` bash
$ telnet localhost 11211
Trying ::1...
Connected to localhost.
Escape character is '^]'.
get a29ajb8go3bkm6r6k2qu4hv3d5
VALUE a29ajb8go3bkm6r6k2qu4hv3d5 0 298
state|s:32:"hgc8cf6e737b4f28ca9872ghe059577b";access_token|s:113:"AAACldPr8ZAokBAFmih7MCYhTlrOSmz7Ro3wJrZCVLeZCkrpQGhSL5hy4dRjYXikOjaBWbt2GJkjcpQj6MpJIopZBU3vURpVfJZBHKZAb7MyQZDZD";user_id|i:100000123456789;LAST_FACEBOOK_IDENTITY_CHECK|s:10:"1334154070";LAST_FACEBOOK_OAUTH_CHECK|s:10:"1334154070";
END
```

There are a few Ruby libraries out there which can speak the language of PHP serialization. I've chosen Thomas Hurst's [php-serialize](https://github.com/watsonbox/php-serialize). Note that I've linked to my fork which also supports `.` characters in array keys. Add that to your Gemfile, and then include the following middleware in your Rails application.

{% gist 2367946 %}

Now you can use this session store in `session_store.rb`:

```
YourApp::Application.config.session_store :php_session_store, :expire_after => 10.minutes
```

That's it, your applications should both now be able to seamlessly access the same session data.