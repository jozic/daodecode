---
layout: post
title: "Override a list property for Typesafe Config"
date: 2014-02-04 19:39
comments: true
categories: [typesafe-config, scala, java]
---
Today I naively was trying to override a list property for a unit test with the following code
``` scala
System.setProperty("list.property", """[ "val1", "val2"]""")
```

The tested code is reading the list property using [typesafe config](https://github.com/typesafehub/config) library

``` scala
val listProperty = config.getListString.asScala.toSet
```

That was throwing
`com.typesafe.config.ConfigException$WrongType: system properties: list.property has type STRING rather than LIST`

so I've googled a [solution](https://github.com/typesafehub/config/issues/69) for overriding list properties from
command line, which is same for overriding in code

Here is how it looks now

``` scala
System.setProperty("list.property.0", "val1")
System.setProperty("list.property.1", "val2")
```






