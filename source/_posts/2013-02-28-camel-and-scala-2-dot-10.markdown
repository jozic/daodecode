---
layout: post
title: "Camel and Scala 2.10"
date: 2013-02-28 23:33
comments: true
categories: [scala, camel]
---

[Camel](http://camel.apache.org/) and [Scala](http://www.scala-lang.org/) is a good combination,
but if you like to be on the bleeding edge and almost done converting your scala app to [2.10](http://www.scala-lang.org/node/27499)
you can find that camel still doesn't have a stable release supporting scala 2.10.
So SNAPSHOT will do the trick in this case, but you know... it's a SNAPSHOT :)  
All you need is apache snapshots repo and camel-scala 2.11-SNAPSHOT

``` scala
resolvers += "Apache Snapshots" at "http://repository.apache.org/snapshots/"

libraryDependencies += "org.apache.camel" % "camel-scala" % "2.11-SNAPSHOT"
```
Add those to your sbt build and you are all set.