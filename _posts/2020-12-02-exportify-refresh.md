---
layout: post
title: "Exportify Refresh"
description:
  An overview of new Exportify features, fixed bugs and improved robustness, as well as a complete overhaul of the React dev stack
date: 2020-12-02 18:00:00 +0100
tags: [Exportify, Spotify, Javascript, React]
---

Way back in 2015 I released the first version of [Exportify](https://github.com/watsonbox/exportify), a small web application for exporting / backing up Spotify playlists to CSV format for safekeeping (click [here](https://watsonbox.github.io/exportify/) to go straight to the app if you'd like to try it).

It's fair to say I've rather neglected this project over the past few years, so I decided to spend a good chunk of time this November on adding new features, fixing bugs and improving robustness, as well as a complete overhaul of the [React](https://reactjs.org/) dev stack.

<a href="https://github.com/watsonbox/exportify" target="_blank"><img src="https://github.com/watsonbox/exportify/raw/3b95ee506b032b84078311c44ebbc79a165bf223/screenshot.png"/></a>

<!--more-->

## New Features

Although the technical stack refresh came first in order of work, it's probably the new features that'll interest users the most.

It's important for me to say that in many cases these features came from [the suggestions](https://github.com/watsonbox/exportify/issues?q=is%3Aissue+is%3Aclosed) [or contributions](https://github.com/watsonbox/exportify/pulls?q=is%3Apr+is%3Aclosed) of others, so I did my best to work through old pull requests in order, and credit people wherever I could. Thanks to all who contributed 🙏. The main updates:

* A new [playlist search facility](https://github.com/watsonbox/exportify/pull/82)
* [Artist and audio features](https://github.com/watsonbox/exportify/pull/80) as configurable extra data in exports
* Possible to [change user](https://github.com/watsonbox/exportify/pull/84) (log out)
* [Liked songs can be exported](https://github.com/watsonbox/exportify/commit/bc5253c7ad0687eaaaad4cb761d9a836dbcbed83) as a playlist
* [Complete revamp of rate limiting](https://github.com/watsonbox/exportify/pull/75) meaning that in most cases it's not something users will need to worry about
* Various miscellaneous bugs fixed

There's loads more detail on each of those items and more in [the milestone](https://github.com/watsonbox/exportify/milestone/1?closed=1) over on Github, or of course in the [project `README`](https://github.com/watsonbox/exportify). Hopefully these additions will improve the usefulness of the tool for the majority of use cases.

Next let's take a look at what's changed under the hood.


## Development Stack Overhaul

When I came to revisit the tech stack for this project, I wasn't inspired at all to build on top of what I'd put together back in 2015. Okay, there was an automated test suite but coverage wasn't particularly great and it would fail intermittently. There was no dependency management framework, and no development/production environment separation. The libraries I'd used were out of date and the React/JS world had moved on impressively since then.

Starting with [Create React App](https://github.com/facebook/create-react-app) I ended up with the following tools/libs:

* [React 17](https://reactjs.org/)

    A JavaScript library for building user interfaces

* [Bootstrap 4](https://getbootstrap.com/)

    Styling and UI components

* [Font Awesome 5](https://fontawesome.com/)

    Vector icon set and toolkit

* [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)

    Light-weight solution for testing React DOM nodes

* [Mock Service Workers (MSW)](https://mswjs.io/)

    Network-level request mocking (more of my own thoughts [here](https://watsonbox.github.io/posts/2020/11/30/discovering-msw.html))

It really was day and night to see how things evolved as I added in new components and upgraded others. In particular working with MSW was a much improved testing experience for a web request-heavy app. You can check out [my other blog post about my experience in setting it up](https://watsonbox.github.io/posts/2020/11/30/discovering-msw.html).

The majority of this new stack was introduced as part of [this PR](https://github.com/watsonbox/exportify/pull/72), so feel free to take a look if you'd like to dig into the code.

### Development Confidence

_One_ of the reasons I haven't updated this project more over the years was: I was kind of worried about breaking things 😬. As I see it, my recent changes now give me and contributors three lines of defense:

1. **Writing code**

    Moving to [Typescript](https://www.typescriptlang.org/) for type checking, in their own words, "saves you time catching errors and providing fixes before you run code".

2. **Deploying code**

    The more comprehensive automated test suite using [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/), [Jest](https://jestjs.io/), and [MSW](https://mswjs.io/) should give a lot more confidence that things aren't breaking. Snapshot testing is now also present as a useful guard against regressions.

3. **Debugging live issues**
  
    Those two measures clearly don't catch everything. For the rest there's live error monitoring with [Bugsnag](https://www.bugsnag.com/). I've [configured it](https://github.com/watsonbox/exportify/blob/7dbccd31f1d8595fd900374847d40542a6f39ae9/src/index.tsx#L9-L21) to make sure that no sensitive data leaks into the error reports, only the details of the exception itself. This also helps to answer questions like "what percentage of distinct users experience an issue?".


## Future Development

A small note about my own personal vision when it comes to the feature set of this tool.

While I'm always open to considering feature requests or code submissions, my own view is that they should remain focused on _getting data out of the Spotify Web API_ rather than on presenting it or importing it into other applications.

It's surely the case that there are plenty of interesting things that can be done with this data once it's been exported, though. Let me know what you come up with! 👇
