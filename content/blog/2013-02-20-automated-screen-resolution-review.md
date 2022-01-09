---
title: "Automate responsive screen captures with Review"
description: "Awesome node.js package that uses PhantomJS for easy screengrabs."
tags: [automation, node.js, front-end]
categories: blog
published: true
comments: false
---

Review is an awesome node.js package that uses PhantomJS to grab screenshots of all your websites in different screen resolutions.

It's best said on the [review](https://github.com/juliangruber/review) Github page:

> Updating large and possibly responsively designed sites can be a hassle. You never know whether your change breakes anything on the other end of your sitemap, or in a certain resolution, except if have a look at every individual page...in every resolution you care about.

Simply install review from the node.js package manager and feed it a JSON string of different websites and screen resolutions that you'd like to see them in. Highly recommended for anyone who needs to keep tabs on a responsive design, etc.

Install with the node package manager:

    $ npm install -g review

Lots of detailed usage examples [here](https://github.com/juliangruber/review).

![Taken from the review Github page](/images/posts/review-screenshot.png)
