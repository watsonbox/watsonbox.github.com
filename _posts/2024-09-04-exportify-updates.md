---
layout: post
title: "Exportify 2024 Updates"
description:
  Dark mode, search enhancements and exports, internationalization, and all dependencies updated!
date: 2024-09-04 18:00:00 +0200
tags: [Exportify, Spotify, Javascript, React]
---

[Last time](https://watsonbox.github.io/posts/2020/12/02/exportify-refresh.html) I made a series of updates the focus was mainly on the technical stack, bugs and robustness. This time I've put more effort into delivering all the features you've requested since then, plus some of my own! Hopefully you'll find that these changes make [Exportify](http://exportify.app) a more useable and mature tool.

## 🌓 Dark mode

The new [Color modes support](https://getbootstrap.com/docs/5.3/customize/color-modes/) in Bootstrap 5.3 made this much easier, so I thought, why not! I'm pretty pleased with the result:

![Image]({{"/assets/images/Exportify Dark Mode.png" | relative_url }})

## 🗺 Internationalization

This one was [requested](https://github.com/watsonbox/exportify/issues/102) some years ago, and I admit at first I was hesitant because of how it might slow down development. In the end, though, I was curious about the implementation, so I decided to build it after all. At this point the feature set is pretty stable, so I hope that the benefit it brings to users will easily outweigh the impact on development time.

<!--more-->

![Image]({{"/assets/images/Exportify Internationalization.png" | relative_url }})

A few notes on this feature:

- CSV file headings have also been translated.
- In future I might add features which fall back to English, relying on [community contributions](https://github.com/watsonbox/exportify/pulls) to bring languages up to date 🙏.
- The translations I've added are AI generated so will almost certainly need tweaks, but hopefully serve as a good starting point.
- The 8 initially supported languages are roughly the most used ones [according to Spotify](https://github.com/watsonbox/exportify/pull/179).

They are: English, French (Français), Spanish (Español), Italian (Italiano), German (Deutsch), Portuguese (Português), Swedish (Svenska), and Dutch (Nederlands).

## 🔍 Search enhancements

This feature has two main components:

1. It's now possible to click "Export All" to export the results of any search, with no upper limit.
2. There is a new advanced syntax for searching by owner, public and collaborative status.

![Image]({{"/assets/images/Exportify Advanced Search.png" | relative_url }})

## Many other smaller updates

As on previous major updates, I also took this opportunity to refresh almost all of the project technical dependencies, so it's now running on:

- [React 18.3](https://react.dev/) - JavaScript library for building user interfaces
- [Bootstrap 5.3](https://getbootstrap.com/) - Styling and UI components
- [Font Awesome 6.6](https://fontawesome.com/) - Vector icon set and toolkit
- [TypeScript 5.5](https://www.typescriptlang.org/) - JavaScript with syntax for types

There are many other tweaks too, including:

- Improved mobile usability
- Better loading indicators
- A full migration to TypeScript
- Exporting playlists with the same name
- New CSV fields
- Several bug fixes

If you'd like to dig deeper into the details of all the small updates as well as those mentioned above, you can find them on the Github [August 2024 Refresh milestone](https://github.com/watsonbox/exportify/milestone/2?closed=1).

## Exportifies?

I must confess to being a little disappointed that Exportify has now fragmented into two versions, and that the existence of `exportify.net` serves to position the fork as the "canonical" version. I would have much preferred for community efforts on improving it to be concentrated in one place, and for my own continued efforts and new features to remain discoverable. That’s open source though, and of course two versions are better than none 🙂.

Exportify remains a labour of love for me: an interesting side-project which also helps me to keep my front-end development skills up to date. While marketing isn't my focus, I hope despite that it will continue to be useful for those who could benefit from it.

With future discoverability in mind, I reluctantly registered [http://exportify.app](http://exportify.app) (the old link will still work).

**And with that, I hope you enjoy these new features!**
