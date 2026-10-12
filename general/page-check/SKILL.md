---
name: page-check
description: Load a URL in a real headless browser and report what actually happened - console errors, failed requests, rendered text - instead of trusting a status code. Use to verify your own web work or to check whether a third-party site is real. Triggers on - does it render, is the site up, check the page, did my change work.
---

# Checking a page for real

An HTTP status is not evidence that a page works. Three failures this catches,
all of which return 200:

- **A client-rendered app that never mounts.** 200, and a body of almost nothing.
- **A soft 404.** A site serving its not-found page at every path, 200 each time.
  The title is often the giveaway.
- **A page whose content arrives only after JavaScript runs.** `curl` sees an
  empty shell and you conclude the site has no pricing, when it has three tiers.

And one that returns an error while being perfectly healthy:

- **A bot challenge.** A 403 with a title like "Just a moment..." is a firewall,
  not a dead site. Do not reject a product on this. Verify it somewhere the
  firewall does not reach: the package registry, the source repository, the
  changelog.

## The order to check things in

**Read the text before the pixels.** A screenshot costs a great deal and answers
"does this look right", which is narrower than it seems.

1. The **accessibility tree or rendered text**: headings, links, visible copy.
2. **Console errors and failed requests.** These catch most breakage for almost
   nothing. An uncaught exception is nearly always a real bug. A 404 on a script
   chunk usually means a stale build is being served.
3. **A direct question to the DOM** when you want one specific fact.
4. A **screenshot**, only when the question is genuinely visual.

## Wait for content, not for a delay

A slow client-rendered app and a broken one are indistinguishable unless you
wait for something specific to appear. Wait for a selector, not a timeout.

## Never assert on text you put there yourself

A check for "error" matches the word in the prompt you just typed into the app
as readily as a real failure. The test passes while proving nothing. Match only
strings that can come from the system under test.

## Interpreting what you get

- Empty title and near-zero body length: the app did not mount. Read the console
  first, not the DOM.
- Failing requests to optional local services are frequently normal. Check
  whether the dependency is meant to exist before reporting it.
- A redirect landing on a different host is worth noticing, especially when
  judging whether a product still belongs to the company that submitted it.
