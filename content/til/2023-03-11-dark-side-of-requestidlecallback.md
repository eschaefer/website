---
title: 'The Dark Side of requestIdleCallback'
tags: [javascript]
categories: blog

comments: false
---

The `requestIdleCallback()` method in the DOM spec allows us to improve the performance of web applications by allowing us to de-prioritize some functions which could be run at a later time. But as with any tool available to optimize code execution, it comes with a dark side.

Here's an example of how I used it recently:

I'm working on an Electron desktop app. It has two pieces which communicate with each other:

- a system tray menu
- a web window with an interface.

There is an [IPC message channel](https://www.electronjs.org/docs/latest/tutorial/ipc) set up from the web window to the tray menu.

The web window is responsible for the heavy lifting, and all the network stuff. Within Electron it is a sandboxed Chromium browser, which has a WebSocket subscription open, and gets new messages at irregular intervals. Basically whenever my calendar is updated.

I wanted to be super clever with handling those messages. The new data would not need to appear instantly in the tray menu. A short delay would be fine, if the main thread was busy doing other things. So, I decided to emit those updates in `requestIdleCallback()`.

This worked fine for a while... until it didn't. I started getting reports of the tray menu not updating for a long time. Or not at all. So what on earth could be going on here? Surely the web window wasn't totally occupied 100% of the time right? Was my mental model flawed about `requestIdleCallback()`?

So, I started some [digging into the spec](https://w3c.github.io/requestidlecallback/#start-an-idle-period-algorithm). Something really stood out to me there:

> if the Document's visibility state is "hidden" then the user agent can throttle idle period generation [...]

To me, this reads clearly as "if the document is not visible, a `requestIdleCallback()` might not run until it's visible again". So, I decided to test this...

If you're viewing this article on **Safari**, then the following examples will not work. We can shim the behavior, but the shim will not accurately reproduce the intended "idle" behavior as [defined by the W3C spec for an idle period](https://w3c.github.io/requestidlecallback/#dfn-idle-period). Therefore I'd recommend viewing this article on Chrome.

### What happens with `requestIdleCallback()`s on an interval in a hidden document?

Minimize this browser for about 30 seconds and then come back. What do the log timings in the console of the codesandbox look like? 👇👇

_⚠️ Won't work on Safari_

<!-- insert embed -->
<iframe src="https://codesandbox.io/embed/blissful-bohr-p6cvr2?expanddevtools=1&fontsize=14&hidenavigation=1&module=%2Fsrc%2Findex.js&theme=dark"
style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
title="blissful-bohr-p6cvr2"
allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>

If you did this on Chrome, you'll see the timestamps from the logs within `requestIdleCallback()` all flood in at the same timestamp. There was backpressure of all the functions passed to `requestIdleCallback()` all queued to execute at the same time when the document became `visible` again.

This could be disastrous in some cases. Imagine that you have some analytics stuff or data syncing that makes network calls inside a function passed to `requestIdleCallback()`. You could potentially DDoS your service if these calls are being made on some kind of interval, and they're quietly being queued-up to fire when the document becomes visible again. This would also unleash a crippling effect on the CPU and memory resources of the browser as well, depending on what's happening in the callback.

### Solution? Use the `timeout` option

As I was browsing the [MDN docs](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestIdleCallback) about `requestIdleCallback()` I noticed it accepts a config object with a `timeout` option. They also explicitly recommend:

> A timeout option is strongly recommended for required work, as otherwise it's possible multiple seconds will elapse before the callback is fired.

So how does this affect the test from earlier? Try it again... minimize this browser for about 30 seconds and then come back. What do the log timings in the console of this codesandbox look like? 👇👇

_⚠️ Won't work on Safari_

<!-- insert embed -->
<iframe src="https://codesandbox.io/embed/wizardly-bardeen-gkt2gr?expanddevtools=1&fontsize=14&hidenavigation=1&module=%2Fsrc%2Findex.js&theme=dark"
    style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
    title="wizardly-bardeen-gkt2gr"
    allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
    sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>

Now each callback is guaranteed to execute after 2 seconds if an idle period was not observed. Meaning, even if the document is hidden, the callback will be guaranteed to execute after 2 seconds.

### Closing thoughts

This quirk happened to affect Chrome, but the `requestIdleCallback()` spec is not implemented the same way across all browsers. Firefox does not appear to queue up the callbacks in the same way while the document is hidden (try it out on the examples above).
