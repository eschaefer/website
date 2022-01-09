---
title: "Building an audiovisual installation using PD"
tags: [creative code, PureData]
categories: blog
published: true
comments: false
---

![PD screenshot](/images/posts/pd-patch.png)

Creative code time. I started the putting together a PureData patch this week, based on a installation proposal [my friend Nick](https://soundcloud.com/sftstps/) and I collaborated on. The concept is to take video motion data from a room, and make the aggregate change in motion speed (people moving) directly influence a full-surround sound environment. No motion would result in a low drone, and quick motion would turn the environment into a responsive dance freakout technorave. It'll be rad.

Currently the patch just takes motion detection data and translates the values to influence a few oscillators. Time to switch those out for a looping audio file...

I'm making the code [freely available on Github](https://github.com/eschaefer/field), too.
