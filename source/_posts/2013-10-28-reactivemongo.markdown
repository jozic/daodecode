---
layout: post
title: "ReactiveMongo, macros and  NoSuchElementException: None.get"
date: 2013-10-27 20:40
comments: true
categories: [scala, mongo, reactivemongo]
---
[ReactiveMongo](http://reactivemongo.org/) employs [scala macros](http://scalamacros.org/)
 to generate readers(deserializers) and writers(serializers)
for scala case classes. They work pretty good, but sometimes you can get a
`java.lang.NoSuchElementException` with message `None.get` :(

Let's say you have
``` scala
case class Person(fisrtName: String, lastName: String, age: Int)

object readers {
  implicit val personReader = Macros.reader[Person]
}

```

and somewhere in your code you use it
``` scala
import readers._
// ...
def findByFirstName(fName: String) =
  personCollection.find(BSONDocument("firstName" -> fName)).one[Person]
```

if your data is correct you are good, but let's say you have some corrupted documents, for example `Joe` doesn't have last name
```
> db.person.find().pretty()
{
	"_id" : ObjectId("526ecf6f7e04ab5f2d1a12ba"),
	"firstName" : "Eugene",
	"lastName" : "Platonov",
	"age" : 27
}
{
	"_id" : ObjectId("526ecf817e04ab5f2d1a12bb"),
	"firstName" : "Joe",
	"age" : 23
}
```

`findByFirstName("Eugene")` will return Future of Success, but `findByFirstName("Joe")`
will return you Future of Failure with ugly `java.util.NoSuchElementException: None.get` which
points to the line where your macro-generated reader is defined.

Here is another, even more interesting, but much harder to spot problem:

If you insert your person doc via mongo shell

```
 db.person.insert({firstName: "Eugene", lastName: "Platonov", age: 27})
```
the type of `age` field will be ... right, [Double](http://docs.mongodb.org/manual/core/shell-types/).
Despite that mongo shell will show you `"age" : 27` when you search for that document.
And again reactivemongo will throw a `NoSuchElementException` exception at you. Sigh.

To avoid it insert integers as `NumberInt`s

```
db.person.insert({firstName: "Eugene", lastName: "Platonov", age: NumberInt(27)})
```

I think reactive mongo should be more explicit about what went wrong and hopefully
this will be [fixed soon](https://github.com/ReactiveMongo/ReactiveMongo/pull/131), but if not, you know where the dog lies buried.




