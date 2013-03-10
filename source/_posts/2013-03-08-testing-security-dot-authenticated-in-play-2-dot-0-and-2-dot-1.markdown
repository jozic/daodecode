---
layout: post
title: "Testing Security.Authenticated in Play 2.0 and 2.1"
date: 2013-03-08 21:00
comments: true
categories: [scala, play]
---

You have a [Play](http://www.playframework.com/) application and let's say you want to add some simple authentication logic to restrict access to your actions.
You want all your users to provide a ticket (e.g [cas proxy ticket](https://wiki.jasig.org/display/CAS/Proxy+CAS+Walkthrough))
in a request header, so you can use the ticket to authenticate them.

Play's [Secured.Authenticated](http://www.playframework.com/documentation/api/2.1.0/scala/index.html#play.api.mvc.Security$)
to the rescue!

You can come up with something like this in Play 2.1:

``` scala
trait Secured {

  val logger = Logger("secured")

  final def fail(reason: String) = {
    logger.debug(s"Access attempt failed: $reason")
    Unauthorized("must be authenticated")
  }

  final def secured[A](action: Action[A]) =
    Security.Authenticated(
      req => req.headers.get("authTicket"),
      _ => fail("no ticket found")) {
      ticket => Action(action.parser) {
        request => withTicket(ticket) {
          action(request)
        }
      }
    }

  private def withTicket(ticket: String)(produceResult: => Result): Result =
    Async {
      isValid(ticket) map {
        valid => if (valid) produceResult else fail(s"provided ticket $ticket is invalid")
      }
    }

  def isValid(ticket: String): Future[Boolean]
}
```
In Play 2.0 it looks almost same except that `isValid` returns _old_ play's `Promise` instead of scala's `Future`.

And you can use it like this:

``` scala
    def securedAction = secured {
      Action {
        request => Ok("Am I protected?")
      }
    }
```

All good, but how do we know it works? Test it!

Let's create a fake controller which has both secured and non-secured actions. And we'll use it in our tests, or rather [specs](http://etorreborre.github.com/specs2/).

``` scala
  object FakeController extends Controller with Secured {
    def securedAction = secured {
      Action {
        request => Ok("Am I protected?")
      }
    }

    def nonSecuredAction = Action {
      request => Ok("I don't care")
    }

    def isValid(ticket: String) = Promise.pure(ticket == "valid")
  }
```

To test that `authTicket` param is required we can have something like this:

``` scala
    "return UNAUTHORIZED if `authTicket` param is not provided" in {
      running(app) {
        val result = FakeController.securedAction process FakeRequest()
        status(result) must_== UNAUTHORIZED
        contentAsString(result) must_== "must be authenticated"
      }
    }
```

Here `app` is a [FakeApplication](http://www.playframework.com/documentation/api/2.1.0/scala/index.html#play.api.test.FakeApplication)
 and `process` is our method (added through implicits) which returns a [Result](http://www.playframework.com/documentation/api/2.1.0/scala/index.html#play.api.mvc.Result)
  given a [Request](http://www.playframework.com/documentation/api/2.1.0/scala/index.html#play.api.mvc.Request).

Please note in Play 2.0 `status` and `contentAsString` don't know how to handle [AsyncResults](http://www.playframework.com/documentation/api/2.1.0/scala/index.html#play.api.mvc.AsyncResult),
so with this implementation of `withTicket` you need to do some trible dance to make it work.

In Play 2.0 `Authenticated` returnes `Action[(Action[A], A)]` so our `process` looks like this:

``` scala
  implicit def action2actionExecutor[A](wrapped: Action[(Action[A], A)]): ActionExecutor[A]
  = new ActionExecutor[A](wrapped)

  class ActionExecutor[A](wrapped: Action[(Action[A], A)]) {
    def process(request: Request[A]): Result = wrapped.parser(request).run.await.get match {
      case Left(errorResult) => errorResult
      case Right((innerAction, _)) => innerAction(request)
    }
  }
```

in Play 2.1 we get just [EssentialAction]() (and we can use [implicit class]()) so it looks much simple:

``` scala
  implicit class ActionExecutor(action: EssentialAction) {
    def process[A](request: Request[A]): Result =
      concurrent.Await.result(action(request).run, Duration(1, "sec"))
  }
```

To test what happens if `authTicket` is provided but is not valid we can do the following:

``` scala
    def requestWithAuthTicket(ticket: String = "invalid") =
      FakeRequest().withHeaders("authTicket" -> ticket)

    "return UNAUTHORIZED if `authTicket` param is not valid" in {
      running(app) {
        val result = FakeController.securedAction process requestWithAuthTicket()
        status(result) must_== UNAUTHORIZED
        contentAsString(result) must_== "must be authenticated"
      }
    }
```
To make sure that it allows access when ticket is valid let's add this

``` scala
    "return whatever it returns if `authTicket` param is valid" in {
      running(app) {
        val result = FakeController.securedAction process requestWithAuthTicket(ticket = "valid")
        status(result) must_== OK
        contentAsString(result) must_== "Am I protected?"
      }
    }
```

Full source code can be found on [github](https://github.com/jozic/play-security-authenticated-tests) in branches play20 and play20 correspondently.