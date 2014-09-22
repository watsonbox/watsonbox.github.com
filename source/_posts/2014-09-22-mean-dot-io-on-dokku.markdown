---
layout: post
title: "Mean.io on Dokku"
date: 2014-09-22 12:32:41 +0200
comments: true
categories: [Docker, Dokku, Node.js, Angular, MongoDB, Express]
---

Here's how I set up the [Mean.io](http://mean.io/) stack for deployment with [Dokku](https://github.com/progrium/dokku).

* Set up MongoDB on the Dokku host and create a database. I used Jeff Utter's single container [plugin](https://github.com/jeffutter/dokku-mongodb-plugin) for this.

```bash
$ dokku mongodb:start
$ dokku mongodb:create <app> <database>
$ dokku mongodb:link <app> <database>
```

* Modify `config/env/production.js` to use ENV var for MongoDB URL:

```js config/env/production.js
'use strict';

module.exports = {
  db: process.env.MONGO_URL,

// [REST OF FILE ELIDED]
```

* Configure the application to run in the production environment:

```bash
$ dokku config:set <app> NODE_ENV=production
```

* Set the unsafe-perm npm configuration option to allow npm to operate under the root account. More information [here](https://github.com/progrium/dokku/wiki/Troubleshooting). Create `.npmrc` as follows:

```bash .npmrc
unsafe-perm = true
```

* Configure a [buildpack which supports Grunt](https://github.com/mbuchetics/heroku-buildpack-nodejs-grunt) in `.env`:

```bash .env
BUILDPACK_URL=https://github.com/mbuchetics/heroku-buildpack-nodejs-grunt.git
```

* Add Heroku build task without environment to `Gruntfile.js` since Dokku doesn't read config during build steps:

```js Gruntfile.js
  // [REST OF FILE ELIDED]

  // For Heroku users only.
  // Docs: https://github.com/linnovate/mean/wiki/Deploying-on-Heroku
  grunt.registerTask('heroku:production', ['cssmin', 'uglify']);

  // Dokku does not set ENV vars during build steps:
  grunt.registerTask('heroku:', ['cssmin', 'uglify']);
};
```