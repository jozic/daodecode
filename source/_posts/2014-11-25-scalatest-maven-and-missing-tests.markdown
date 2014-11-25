---
layout: post
title: "Scalatest, Maven and Missing Tests"
date: 2014-11-25 14:49
comments: true
categories: [scala, maven, scalatest]
---

As you may know [ScalaTest](http://www.scalatest.org) provides a [JUnitRunner](http://www.scalatest.org/user_guide/using_junit_runner) and you can perfectly run your scala tests as if they are regular junit tests. In particular, you can run them from maven, using surefire plugin. Why would you do this? In case you have a mixed Java/Scala project with tests written using junit as well as scalatest.
There is a small issue though, which can make you think you are going crazy! Maven may ignore some tests and skip them. How come?

With scalatest you can write much more descriptive and readable tests like

``` scala
  "withFrequency" should "calculate frequency of each element in the source" in {
    Iterable("a", "b", "c", "a", "b", "d").withFrequency should be(Map("a" -> 2, "b" -> 2, "c" -> 1, "d" -> 1))
  }
```
But if at least one of the tests has parenthesis in test name, then something goes wrong and surefire (scalatest junitrunner???) just pretends the whole test class doesn't exist.

`Causes maven to ignore whole test class`
```scala
  "withFrequency" should "calculate frequency (of each element in the source)" in {
    Iterable("a", "b", "c", "a", "b", "d").withFrequency should be(Map("a" -> 2, "b" -> 2, "c" -> 1, "d" -> 1))
  }
```

The solution is obvious - avoid parenthesis. If you really want them, you can easily use square brackets instead.

`Workaround`
```scala
  "withFrequency" should "calculate frequency [of each element in the source]" in {
    Iterable("a", "b", "c", "a", "b", "d").withFrequency should be(Map("a" -> 2, "b" -> 2, "c" -> 1, "d" -> 1))
  }
```
