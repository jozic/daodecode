---
layout: post
title: "Root Application Context in Play 2.1"
date: 2013-02-23 17:34
comments: true
categories: [scala, play]
---

So [Play 2.1](http://www.playframework.com/) is officialy out and one of the new features it brings is the ability
to configure root application context. It have been in [2.1 branch](https://github.com/playframework/Play20/commit/da6bbc4)
 for a long-long time and I don't see this mentioned in
[highlights section](http://www.playframework.com/documentation/2.1.0/Highlights), but nevertheless it's there and it works.
To assign a root context to your play app just open your `application.conf` file and add the following line
```
application.context=/my_root_context
```
Make sure you have it starting with slash (`/`)
otherwise play will throw a configuration error at you saying "Invalid application context".

