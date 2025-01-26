---
title: 'Console logging can lead to memory leaks'
tags: [javascript, til]

comments: false
---

It's pretty typical to use `console.log` to debug problems in a UI. That is if you want to travel through time ([and not space](https://twitter.com/sophiebits/status/1481833292810924036)). However, if you log an object or array, this can quickly lead to memory leaks, especially in React apps where components can potentially re-render many times.

When the object or array is logged into the developer tools, the Javascript engine garbage collector **cannot clean up the values being logged**. This really took me for a wild ride until I realized what was happening.
