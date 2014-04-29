---
layout: post
title: "Call Scala curried functions from Java"
date: 2014-04-29 13:40
comments: true
categories: [scala, java]
---

For some reason I thought scala curried functions is one of the features you will have troubles using from Java. But it happens that you can perfectly call a curried Scala function

``` scala
class A {
	def curried(i: Int)(s: String) = i + s.toInt
}
```
from Java

``` java
new A().curried(5, "7")

```