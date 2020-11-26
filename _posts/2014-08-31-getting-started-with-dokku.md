---
layout: post
title: "Getting Started with Dokku"
date: 2014-08-31 14:35:02 +0200
permalink: /blog/2014/08/31/getting-started-with-dokku/
tags: [Docker, Dokku]
---

*UPDATE: Some recent changes to [ohardy/dokku-mariadb](https://github.com/ohardy/dokku-mariadb) have resolved the problems with losing DB links on reboot.*

I decided to jump in and set up my own mini-Heroku with [Docker](https://www.docker.com/) and [Dokku](https://github.com/progrium/dokku) as seems to be the fashion at the moment. Here are my thoughts and a few of the issues I hit along the way. Using Dokku v0.2.3.

I [installed Dokku on Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-use-the-digitalocean-dokku-application). It doesn't have to be Digital Ocean but they certainly make it easy. Then [added some swap space](https://www.digitalocean.com/community/tutorials/how-to-add-swap-on-ubuntu-14-04), too. Save some typing by setting up an alias:

```bash
$ alias dokku='ssh -t dokku@yourvps.tld
```

<!--more-->

## Configuration Variables

Dokku allows applications to be configured using enviornment variables as described in the twelve-factor app:

```bash
$ dokku config:set acme company_name="Acme Corporation"
```

That won't work though, because Dokku's command line app [doesn't support spaces in config variables](https://github.com/progrium/dokku/issues/482). You can get around this by setting the configuration directly in `/home/dokku/acme/ENV`:

```bash
export company_name="Acme Corporation"
```

They won't be correctly reported by `dokku config acme`, but they'll work.


## Configuration Backup

You can backup Dokku's configuration (including application configuration) with:

```bash
$ dokku backup:export [FILE]
```

This will place the backup in `/home/dokku/[FILE]`.


## Multiple Domains and Basic Auth

I installed a couple of plugins which worked well out of the box:

[https://github.com/matto1990/dokku-secure-apps](https://github.com/matto1990/dokku-secure-apps)<br/>Secures an individual app with HTTP Basic authentication. Dead simple to use and useful when deploying apps for testing or private use only.

[https://github.com/neam/dokku-custom-domains](https://github.com/neam/dokku-custom-domains)<br/>Configure additional custom domains for your dokku app.


## Persistent Storage

Dokku has plenty of [plugins for different data stores](https://github.com/progrium/dokku/wiki/Plugins#datastores). I decided to work with [ohardy/dokku-mariadb](https://github.com/ohardy/dokku-mariadb) since on a VPS with farily limited memory, I preferred to only run one container and a single instance of MariaDB.

The main problem I faced here was keeping database connections working after a reboot. On reboot, docker containers are restarted and will receive a new private IP. This means that the `DATABASE_URL` used by the apps, e.g. `mysql://[PRIVATE_IP]:3306/[database]` will change and break database connections. Despite some [discussion](https://github.com/Kloadut/dokku-pg-plugin/issues/12) and [attempted fixes](https://github.com/Kloadut/dokku-md-plugin/commit/de26c1e10f1e30c059ae827fff65b4a132ccc2ed), the multiple container dokku MariaDB plugin ([kloadut/dokku-md-plugin](https://github.com/Kloadut/dokku-md-plugin)) uses a dynamic port for each container, which also changes on reboot, so suffers from the same problem.

I fixed this issue in [watsonbox/dokku-mariadb:reboot-persist-connections](https://github.com/watsonbox/dokku-mariadb/commit/ab724e99f49968695da3a0ce09fe7582bf735328) by patching ohardy/dokku-mariadb to use a fixed IP address. It also uses a fixed port (3306), since only one container/instance is created.

Worth noting that ohardy/dokku-mariadb stores persistent DB data on the host in `/home/dokku/.o_mariadb` (kloadut/dokku-md-plugin in `/home/dokku/.mariadb`).
