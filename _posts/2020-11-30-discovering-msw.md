---
layout: post
title: "Discovering MSW"
date: 2020-11-30 19:42:58 +0100
tags: [Javascript, React, Exportify]
---

Recently, as part of a [development stack refresh](https://github.com/watsonbox/exportify/pull/72) for [Exportify](https://github.com/watsonbox/exportify), I found myself digging around for the best approach to mocking HTTP requests in a JS test suite.

I needed something that would play nicely with [Jest](https://jestjs.io/) and React, allow me to mock requests at the transport layer so my tests could be de-coupled from the HTTP request library I chose to use, as well as of course providing a convenient DSL for writing the tests themselves.

<!--more-->

Previously I'd written my tests using [CasperJS](https://www.casperjs.org/), and [Mockjax](https://github.com/jakerella/jquery-mockjax) for the request mocking. I'd found the test runs to be a little unreliable, plus CasperJS is no longer actively maintained and Mockjax only actually mocks the jQuery request API, which I also planned to replace during the refresh.

## Enter MSW

I came across [MSW](https://mswjs.io/) (Mock Service Worker), which seemed to be the answer to my problems. It comes by default with [React Testing Library](https://testing-library.com/docs/react-testing-library/intro), so I didn't have to look very far 🙂.

This post isn't a tutorial on how to use MSW: see the [documentation](https://mswjs.io/docs/) for that. Rather, it's a collection of short summaries of what worked well (and didn't!) for me.

## Setting up the Suite

At this point, if you're curious and you'd like to skip ahead, you could check out [`PlaylistTable.test.jsx`](https://github.com/watsonbox/exportify/blob/master/src/components/PlaylistTable.test.jsx) for a complete example of how I set up the tests for the main React component in Exportify.

The general approach there is:

1. Set up the server which handles mocking

    ```jsx
    import { setupServer } from "msw/node"
    import { handlers } from "../mocks/handlers"

    const server = setupServer(...handlers)

    server.listen()
    ```

2. Restore Jest mocks and MSW handlers after each test. This ensures that any contextual changes made for any specific test will be reset.

    ```jsx
    afterEach(() => {
      jest.restoreAllMocks()
      server.resetHandlers()
    })
    ```

3. Create `mocks/handlers.jsx` with some request handlers, for example:

    ```jsx
    export const handlers = [
      rest.get('https://api.spotify.com/v1/me', (req, res, ctx) => {
        return res(ctx.json(
          {
            "display_name" : "watsonbox",
            "external_urls" : {
              "spotify" : "https://open.spotify.com/user/watsonbox"
            },
            "followers" : {
              "href" : null,
              "total" : 6
            },
            "href" : "https://api.spotify.com/v1/users/watsonbox",
            "id" : "watsonbox",
            "images" : [ ],
            "type" : "user",
            "uri" : "spotify:user:watsonbox"
          }
        ))
      })
    ]
    ```

## Catching Unmocked Requests

Most of my experience with mocking requests in test suites these days is using [WebMock](https://github.com/bblimke/webmock) with Ruby on the back end. More often than not, I want to ensure that no "live" requests are made. That is to say, I'd like like an unmocked request to be a test failure. In fact, I often use this as a process for writing the mocks themselves - running the test suite and mocking out one request at a time, validating that they make sense as I go.

As luck would have it, [this feature](https://github.com/mswjs/msw/issues/191) was [merged](https://github.com/mswjs/msw/pull/257) in July this year, and can be switched on by updating the call to `server.listen`:

```jsx
server.listen({
  onUnhandledRequest: 'warn'
})
```

Okay, this will warn rather than raise an error, but [there are other options available](https://mswjs.io/docs/api/setup-server/listen#onunhandledrequest). If the example handler above were omitted, the output would be something like the following, and as expected it helps with writing the mocks themselves:

```bash
console.warn
  [MSW] Warning: captured a GET https://api.spotify.com/v1/me request without a corresponding request handler.

    If you wish to intercept this request, consider creating a request handler for it:

    rest.get('https://api.spotify.com/v1/me', (req, res, ctx) => {
      return res(ctx.text('body'))
    })
```

Great!

## Contextual Mocks

After testing the "happy path" behavior, I'll typically want to set up a more exceptional response from the API. This is often the case when fixing bugs with a test to guard against regressions.

A real example of this is needing to test a bug which only occurs when a Spotify playlist contains an item with a `null` track. The [full commit is here](https://github.com/watsonbox/exportify/commit/032ec7f246308a8acb74de2f70ba706141ad9fda) but the approach was to use a *[runtime request handler](https://mswjs.io/docs/api/setup-server/use)* ([released in May](https://github.com/mswjs/msw/releases/tag/v0.18.0) 😌) to handle a request to `https://api.spotify.com/v1/me` differently for a single spec, as follows:

```jsx
// At the top level
import { handlers, nullTrackHandlers } from "../mocks/handlers"

// New test
test("playlist with null track skips null track", async () => {
  server.use(...nullTrackHandlers)

  // The test body
})
```

In fact, this is the reason for the `resetHandlers()` call in the `afterEach` callback above. It ensures that the runtime handler is removed after each test.

Finally the mock itself must be added to `handlers.jsx`:

```jsx
export const nullTrackHandlers = [
  rest.get('https://api.spotify.com/v1/playlists/4XOGDpHMrVoH33uJEwHWU5/tracks?offset=0&limit=10', (req, res, ctx) => {
    return res(ctx.json(
      {
        "href" : "https://api.spotify.com/v1/playlists/4XOGDpHMrVoH33uJEwHWU5/tracks?offset=0&limit=100",
        "items" : [
          {
            "added_at" : "2020-11-08T22:12:50Z",
            "added_by" : {
              "external_urls" : {
                "spotify" : "https://open.spotify.com/user/"
              },
              "href" : "https://api.spotify.com/v1/users/",
              "id" : "",
              "type" : "user",
              "uri" : "spotify:user:"
            },
            "is_local" : false,
            "primary_color" : null,
            "track" : null, // The exceptional case
            "video_thumbnail" : {
              "url" : null
            }
          }
        ]
      }
    ))
  })
]
```

## Asserting Request Counts

The next thing I came up against was wanting to know how many times a given API endpoint had been called. Setting up mocked responses and checking the results was already great, but how could I be sure that duplicate requests weren't accidentally being fired resulting in the same behavior but with a performance overhead and exasperating rate limiting problems?

Well, I couldn't find anything out of the box so decided to use a pattern relying on [standard Jest mock functions](https://jestjs.io/docs/en/mock-functions) which can be used with any [expectation matchers](https://jestjs.io/docs/en/expect).

First add a mock to the handlers file and call it in each mock body:

```jsx
export const handlerCalled = jest.fn()

export const handlers = [
  rest.get('https://api.spotify.com/v1/me', (req, res, ctx) => {
    handlerCalled(req.url.toString())

    ...
```

This can then be imported and used in the tests:

```jsx
import { handlerCalled, handlers } from "../mocks/handlers"

expect(handlerCalled.toHaveBeenCalledTimes(1))
```

Actually, in my case I wanted to be sure that and API call had been called once *for a specific URL*, so to avoid setting up a "call mock" for each mocked request, I typically ended up setting expectations this way:

```jsx
expect(handlerCalled.mock.calls).toEqual([ // Ensure API call order and no duplicates
  [ 'https://api.spotify.com/v1/me' ],
  [ 'https://api.spotify.com/v1/users/watsonbox/playlists' ],
  [ 'https://api.spotify.com/v1/users/watsonbox/tracks' ],
  [ 'https://api.spotify.com/v1/me/tracks?offset=0&limit=20' ]
])
```

## Matching Requests

Overall, my experience using Jest and MSW has been great, and I feel like I'm hitting the project at the right time since some of the features I've needed haven't been available for long!

Probably the only niggle I had came in the restrictions when setting up the matching of mocked requests. As far as I can tell, setting up a mock like this:

```jsx
rest.get('https://api.spotify.com/v1/playlists/4XOGDpHMrVoH33uJEwHWU5/tracks?offset=0&limit=10', (req, res, ctx) => {
```

Is equivalent to this (without query params):

```jsx
rest.get('https://api.spotify.com/v1/playlists/4XOGDpHMrVoH33uJEwHWU5/tracks', (req, res, ctx) => {
```

In addition to certain parameters, I'd also love to be able to set up mocks which only match certain headers. That way I could set up all the mocks to match the correct Bearer token, and get warnings if that was for some reason not the case, plus perhaps set up some specific 401 responses for testing that behavior.
