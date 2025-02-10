---
layout: post
title: "Personal Slack Data Export"
description:
  A guide to exporting personal Slack data for archival.
date: 2025-02-09 22:00:00 +0100
tags: [Slack]
---

The best tool I found for the job is [`slackdump`](https://github.com/rusq/slackdump). The following steps apply to MacOS only, though of course you may find some work on other platforms.

## Setup

The easiest way to get started is [with Homebrew](https://github.com/rusq/slackdump?tab=readme-ov-file#installation-and-quickstart):

```shell
brew install slackdump
```

To set up authentication, load the Slack web app, and follow the instructions [here](https://github.com/rusq/slackdump/blob/master/doc/login-manual.rst) to extract the token and cookie, which should then be added to a file called `secrets.txt` in the folder where `slackdump` will be run:

```
SLACK_TOKEN=xoxc-<...elided...>
COOKIE=xoxd-<...elided...>
```

<!--more-->

## Channel Listing & Splitting

Unfortunately, `slackdump` doesn't provide any runtime estimation while it's exporting, and exports (particularly with uploaded files) can get very large. Slack rate limiting means they can also take rather a long time.

Let's split the export up into chunks of different types of data.

```shell
slackdump list channels
```

This can take a few minutes, but eventually spits out a file like `channels-XYZ.txt` with a line for each type of chat you have access to.

First let's get rid of archived channels. We won't bother with those.

```shell
sed '/  arch  /d' channels-XYZ.txt > channels-live.txt
```

Now we'll split it up into DMs, groups chats, public and channels:

```shell
awk '$1 ~ /^D/ {print $1}' channels-live.txt > channels-dms.txt
awk '$3 ~ /^Group:/ {print $1}' channels-live.txt > channels-groups.txt
awk '$3 ~ /^#/ {print $1}' channels-live.txt > channels-public.txt
awk '$3 ~ /^🔒/ {print $1}' channels-live.txt > channels-private.txt
```

Make sure they're all there before moving on:

```shell
➜ wc -l channels-dms.txt channels-groups.txt channels-public.txt channels-private.txt
     209 channels-dms.txt
     424 channels-groups.txt
    1459 channels-public.txt
      53 channels-private.txt
    2145 total
➜ wc -l channels-live.txt
    2146 channels-live.txt
```

Should you wish to triage any of these first, do it in two stages like this:

```shell
awk '$3 ~ /^🔒/' channels-live.txt > channels-private-triage.txt
# Remove any channels/chats which are not required
awk '{print $1}' channels-private-triage.txt > channels-private.txt
```

## Exporting

Run an export for each list:

```shell
slackdump export -o dms.zip -avatars -files=true @channels-dms.txt # 26m45s
slackdump export -o groups.zip -avatars -files=true @channels-groups.txt # 7m35s
slackdump export -o private-channels.zip -files=false @channels-private.txt # 1h51m
slackdump export -o public-channels.zip -files=false @channels-public.txt
```

Note that I decided to only download user avatars for the DM export, and omit uploaded files for channels to save space.

I also provided my own runtimes here, which correspond to the number of channels in the `wc` output above (except public which I cut down significantly). Clearly the runtime also depends a lot on channel sizes, but it gives a vague idea of the order of magnitude.

Slack rate limiting definitely seems to be the limiting factor, allowing bursts of ~5s followed by 10s waits, once the export gets going.

### Monitoring Progress

As mentioned, it's not easy to track export progress. I noticed that when it starts, Slackdump shows its temporary directory, for example:

```
INFO temporary directory in use tmpdir=/var/folders/59/xwg_1t6d29l23h48lddxdlsr0000fn/T/slackdump-1846332288
```

Inside are a bunch of files, one for each channel (plus one or two more). Knowing how many files are in the input channel list, we can put together a makeshift progress indicator as follows:

```shell
cd /var/[your dir]
TOTAL=732; while :; do count=$(ls -1 2>/dev/null | grep -vE 'users.json.gz|workspace.json.gz' | wc -l); printf "\r[%-*s] %d/%d (%.1f%%)" 40 "$(printf '#%.0s' $(seq $((40 * count / TOTAL))))" "$count" "$TOTAL" "$((100 * count / TOTAL)).0"; [ "$count" -ge "$TOTAL" ] && break; sleep 1; done; echo
```

(Set `TOTAL` to the total number of channels in the input list).

```
[########################                ] 455/732 (62.0%)
```

## Viewing

As [the `README` describes](https://github.com/rusq/slackdump?tab=readme-ov-file#previewing-results), Slackdump contains its own built-in viewer:

```shell
slackdump view slack-dms.zip
```

Alternatively I've also had success with `slack-export-viewer`:

```shell
python3 -m venv .
source bin/activate
pip3 install slack-export-viewer
source bin/activate
slack-export-viewer -z ./slack-export.zip -p 8000
deactivate
```

**Note**: If you get an error like "codec can't encode character", it's [this](https://github.com/hfaran/slack-export-viewer/issues/184). Extract the zip and point it at the directory instead.
