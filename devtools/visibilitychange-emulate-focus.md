---
title: "`visibilitychange` event not fired when switching tabs"
date: 2024-07-15
tags:
  - devtools
  - javascript
comments: true
description: >-
  The visibilitychange event does not fire when you switch between tabs if you
  have DevTools open with the "Emulate a focused page" option enabled.
---

[Wagtail's concurrent editing notifications feature][wagtail-cen] relies on the
`visibilitychange` event. [According to MDN][mdn-visibilitychange],

> This event fires with a `visibilityState` of `hidden` when a user navigates to
> a new page, switches tabs, closes the tab, minimizes or closes the browser,
> or, on mobile, switches from the browser to a different app.

During development, for some reason I couldn't get the event to fire when I
switched between tabs while DevTools was open.

It works in incognito mode; weirdly, I couldn't get it to work in normal mode,
even after disabling all extensions.

The issue could be replicated using a barebones `.html` file with an event
listener in an inline `<script>` tag, so it wasn't anything to do with the code.

A few hours of debugging led me to the cause: I had the
**"Emulate a focused page"** option enabled.

If you didn't know – the option allows you to ensure the page stays focused as
you interact with DevTools. This is useful for debugging elements that disappear
when focus is lost, such as popups. You can find it in the "Rendering" tab in
Chrome's DevTools (you might need to click the + button to show the tab).

It turns out that the option also prevents the `visibilitychange` event from
being fired (or perhaps `document.visibilityState` from being changed) when you
switch to a different tab.

I was working on a notification popup, so I had the option enabled. After
disabling the option, the event was dispatched as expected.

## TL;DR

The `visibilitychange` event does not fire when you switch between tabs if you
have DevTools open with the "Emulate a focused page" option enabled.

[wagtail-cen]: https://guide.wagtail.org/en-latest/releases/new-in-wagtail-6-2/#concurrent-editing-notifications
[mdn-visibilitychange]: https://developer.mozilla.org/en-US/docs/Web/API/Document/visibilitychange_event
