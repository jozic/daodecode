---
layout: post
title: "Sbt Plugins: No Default Package Please"
date: 2014-02-18 16:46
comments: true
categories: [sbt, scala, java]
---

Usually people don't put sbt builds in a package (I'm talking about scala files here), especially for simple and small projects.
And they easily can use all the sbt plugins.   
But there are other users who have bigger and more complex builds.
So they put their builds in packages and then they discover they can use only plugins which themselves defined in packages. Bummer!


Scala, as well as Java, does allow classes in [default or unnamed](http://docs.oracle.com/javase/specs/jls/se7/html/jls-7.html#jls-7.4.2) package, but those classes can be accessed within default package only.
There is literally [no way](http://docs.oracle.com/javase/specs/jls/se7/html/jls-7.html#jls-7.5) to import a class from default package.
Thus when someone creates an sbt plugin and puts it in default package, (s)he makes owners of packaged builds
 feel almost physical pain when they try to use the plugin.


So if you are an author of an sbt plugin, please consider to put it into a package. Doing otherwise violates
[best practices](https://github.com/sbt/sbt/blob/0.13/src/sphinx/Extending/Plugins-Best-Practices.rst#dont-use-default-package)
 and makes God to kill a kitten.
