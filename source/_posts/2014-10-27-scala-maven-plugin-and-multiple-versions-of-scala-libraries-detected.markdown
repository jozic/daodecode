---
layout: post
title: "scala-maven-plugin and Multiple versions of scala libraries detected"
date: 2014-10-27 17:15
comments: true
categories: [scala, maven]
---

Have you ever seen messages like these running [scala-maven-plugin](https://github.com/davidB/scala-maven-plugin)?
```
[WARNING]  Expected all dependencies to require Scala version: 2.10.3
[WARNING]  mycoolcompany:mycoolproject:1.5.25 requires scala version: 2.10.3
[WARNING]  me.lessis:odelay-core_2.10:0.1.0 requires scala version: 2.10.4
[WARNING] Multiple versions of scala libraries detected!
```

Starting from scala 2.10 all changes in [bugfix/patch](http://semver.org/) version should be backward compatible,
so these warnings don't really have the point in this case.
But they are still very important in case when, let's say, you somehow end up with scala 2.9 and 2.11 libraries.
It happens that since version 3.1.6 you can fix this using `scalaCompatVersion` configuration

``` xml
			<plugin>
				<groupId>net.alchim31.maven</groupId>
				<artifactId>scala-maven-plugin</artifactId>
				<version>3.1.6</version>
				<configuration>
					<scalaCompatVersion>${scala.binary.version}</scalaCompatVersion>
					<scalaVersion>${scala.version}</scalaVersion>
				</configuration>
				<!-- other settings-->
            </plugin>
```

Where in my case I have `scala.binary.version` property defined as

``` xml
<scala.binary.version>2.10</scala.binary.version>
```
No more meaningless warnings!

[Related discussions on google groups](https://groups.google.com/forum/#!topic/maven-and-scala/XSkXrSS8WuU)
