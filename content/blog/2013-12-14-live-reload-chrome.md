---
title: "The quickest way to set-up live reload in Chrome"
tags: [front end, tools]
categories: blog
published: true
comments: true
---

Front-end development just got a little easier. Ever wanted to have like four browser windows next to each other, and have them automatically refresh when you've made an update to the project's javascript, CSS, or whatever?

I read a bunch of walkthroughs on setting up [guard](https://github.com/guard/guard) and [guard-livereload](https://github.com/guard/guard-livereload) through the terminal, but hopefully I can save you from having to do that as well.

Setting up LiveReload through Sublime Text 2's package manager is a much easier option.

So, to get directly to the point:

1. Open Submlime Text 2 and install the LiveReload package through Package Control.
2. Re-open Sublime Text 2 and open up whatever project directory you're working from.
3. Install the Chrome browser extension called [LiveReload](https://chrome.google.com/webstore/detail/livereload/jnihajbhpnppcggbcgedagnkighmdlei) (or the [same extension for Firefox](https://addons.mozilla.org/en-US/firefox/addon/livereload/)). Once it's installed, toggle the extension.
4. Navigate to your dev project in Chrome, and behold it automatically updating when you save changes in ST2!
