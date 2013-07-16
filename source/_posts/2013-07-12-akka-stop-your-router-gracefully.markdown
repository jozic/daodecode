---
layout: post
title: "Akka: Stop Your Router Gracefully"
date: 2013-07-12 23:34
comments: true
categories: [scala, akka, router]
---
Let's say you want to find an answer to [Life, the Universe and Everything](http://en.wikipedia.org/wiki/Answer_to_Life,_the_Universe,_and_Everything)
 and you know it will take some time,
 so you want to do it in parralel while doing something else (DSE).  
You say to yourself "okay, I'll fire an actor and send him a message or bunch of messages, then DSE and collect my results from the actor after i'm done"

So you've got it coded
``` scala
  val answerToLifeTheUniverseAndEverything = new AtomicInteger(0)

  val actor = system.actorOf(Props(new Actor {
    def receive = {
      case i: Int => {
        answerToLifeTheUniverseAndEverything.getAndIncrement()
      }
      case _ =>
    }
  }))
```

Now you want make it work and get results at the end. You need to make sure to access the result after all work is done.
From [akka documentation](http://akka.io/docs/) you found out that [`akka.pattern.GracefulStopSupport`](http://doc.akka.io/api/akka/2.2.0/index.html#akka.pattern.GracefulStopSupport)
seems a good choice to do it.

```
  for (i <- 1 to 42) actor ! i

  // DO SOMETHING ELSE

  import akka.pattern.gracefulStop

  Await.result(gracefulStop(actor, timeout), timeout)

  println(s"We found the answer, it's ${answerToLifeTheUniverseAndEverything.get}")

```
as a result you see
```
We found the answer, it's 42
```
Nice!  
But then you think "I can do it not only in parallel to my DSE, but compute it in many independent actors, so it will be done even faster".
Using a router actor seems an easy way to do it.

```
  val answerToLifeTheUniverseAndEverything = new AtomicInteger(0)

  val router = system.actorOf(Props(new Actor {
    def receive = {
      case i: Int => {
        answerToLifeTheUniverseAndEverything.getAndIncrement()
      }
      case _ =>
    }
  }).withRouter(RoundRobinRouter(nrOfInstances = 10)))


  for (i <- 1 to 42) router ! i

  // DO SOMETHING ELSE

  import akka.pattern.gracefulStop

  Await.result(gracefulStop(router, timeout), timeout)

  println(s"We found the answer, it's ${answerToLifeTheUniverseAndEverything.get}")
```

Router sends messages to routees in round-robin fashion and routees do the work parallel to each other.
At the end you again see
```
We found the answer, it's 42
```

Super nice! You're done! You can go and grab a milk shake. Or do you?    
Computing answer to Life, the Universe and Everything is a complex task, so it really should take some time and effort to calculate it.  
Let's add ```Thread.sleep(1000)``` pretending that our actor(or rather many actors) do something time consuming.
```
    def receive = {
      case i: Int => {
        Thread.sleep(1000)
        answerToLifeTheUniverseAndEverything.getAndIncrement()
      }
      case _ =>
    }
```
And then while you enjoing your shake all of a sudden you see
```
We found the answer, it's 10
```
What the hell?  
The thing is `gracefullStop` sends a `PoisonPill` to an actor, which is a router in our case.
Once an actor gets to a `PoisonPill` message it decides to die and as a good parent, he can't allow his children stay and suffer in this cruel world, so he kills them too.  
"But `PoisonPill` will be the last message my router gets, right? So it dies after all other messages are processed." you say.  
Right, but processing a message from a router's point of view is just to send it to one of its routees.  
But then it dies when they still in progress and because the number of messages (42) is greater than number of routees (10) they all have more than one message to process.
So they do, until their *Daddy* comes and gets them to the *Land of Peace*. Their unprocessed messages are lost.
Thus the answer to Life, the Universe and Everything you get is 10. It can be different, but it usually will be around a number of routees.

So what is the solution?

The answer before akka 2.2 is don't use `gracefulStop` with routers! (Of course you can take a look at the source code and make your own version of `gracefulStop`)
Since 2.2 you can specify stopMessage which is a `PoisonPill` by default. So the line will look like

```
  Await.result(gracefulStop(router, timeout, Broadcast(PoisonPill)), timeout)
```
And now the answer is correct again :)
```
We found the answer, it's 42
```
