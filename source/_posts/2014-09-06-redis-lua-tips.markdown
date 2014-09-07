---
layout: post
title: "Redis Lua Tips"
date: 2014-09-06 18:10
comments: true
categories: [redis, lua] 
---

As you may know [redis](http://redis.io) [allows](http://redis.io/commands/EVAL) you to pass a [Lua](http://www.lua.org/) script which will be executed on server side.
What's great about it is that it will be executed atomically, as redis is single-threaded.    
Though beware of long-running scripts as they can hug the server.
 So what good can you do with lua? Actually a lot, and here some simple use cases
 
Count keys by pattern
---------------------

Sometimes you just want to know how many keys named by some pattern you have. 
You can always run [KEYS](http://redis.io/commands/keys) command, but it prints every single key, 
so if you have thousands of them it can be pretty overwhelming. 
  Instead you can use this simple command:

```
EVAL "return #redis.call('KEYS', 'your pattern goes here')" 0
```

For example, let's say we have:

```
127.0.0.1:6379> KEYS *
1) "daodecode:post:2:tags"
2) "daodecode:post:12:tags"
3) "daodecode:post:2:title"
4) "daodecode:post:10:tags"
5) "daodecode:post:10:title"
6) "daodecode:post:12:title"
```

let's get number of keys with tags:

```
127.0.0.1:6379> EVAL "return #redis.call('KEYS', 'daodecode:post:*:tags')" 0
(integer) 3
```

You've got the idea...

Set expiration only if it has not already been set
--------------------------------------------------

Let's say you get some events from somewhere, extract some info from the events and add it to a redis set. 
And you want your set to expire at some point, so you want to set a [ttl](http://redis.io/commands/ttl) for the set, but only once.
 Redis will kindly create a set key if it didn't exist before. 
 But if you will send [EXPIRE](http://redis.io/commands/expire) command along with every [SADD](http://redis.io/commands/sadd) command, 
 redis will kindly overwrite existing ttl with the new value. Lua to the rescue:
 
```
EVAL "if (redis.call('TTL', KEYS[1]) == -1) then redis.call('EXPIRE', KEYS[1], ARGV[1]) end" 1 <KEY_NAME> <TTL>
```

For example:


``` 
127.0.0.1:6379> SADD s1 m1
(integer) 1
127.0.0.1:6379> TTL s1
(integer) -1
127.0.0.1:6379> EXPIRE s1 30
(integer) 1
127.0.0.1:6379> TTL s1
(integer) 22
127.0.0.1:6379> EXPIRE s1 30
(integer) 1
127.0.0.1:6379> TTL s1
(integer) 28
.0.0.1:6379> SADD s2 m1
(integer) 1
127.0.0.1:6379> EVAL "if (redis.call('TTL', KEYS[1]) == -1) then redis.call('EXPIRE', KEYS[1], ARGV[1]) end" 1 s2 30
(nil)
127.0.0.1:6379> TTL s2
(integer) 27
127.0.0.1:6379> EVAL "if (redis.call('TTL', KEYS[1]) == -1) then redis.call('EXPIRE', KEYS[1], ARGV[1]) end" 1 s2 30
(nil)
127.0.0.1:6379> TTL s2
(integer) 20
```

Here you can see that we reset ttl for `s1`, but for `s2` we set ttl only for the first time, the second request didn't change it

Add members to a sorted set only once
-------------------------------------

Redis gives us [sorted sets](http://redis.io/commands#sorted_set) where we can add members along with scores associated with the members. 
If you [ZADD](http://redis.io/commands/zadd) a member which already exists in the set its score will be overwritten. 
It could happen that you may want to add a member only once. The easy way is to check if member doesn't exist with 
[ZSCORE](http://redis.io/commands/zscore) command. 
But those two commands are not atomic and [races](http://en.wikipedia.org/wiki/Race_condition) are possible. Lua is here again:

``` lua
if not redis.call('ZSCORE', KEYS[1], ARGV[1]) 
    then return redis.call('ZADD', KEYS[1], ARGV[2], ARGV[1]) 
    else return 0
end;
```

Example:

```
127.0.0.1:6379> ZADD s1 1 m1
(integer) 1
127.0.0.1:6379> ZSCORE s1 m1
"1"
127.0.0.1:6379> ZADD s1 2 m1
(integer) 0
127.0.0.1:6379> ZSCORE s1 m1
"2"
127.0.0.1:6379> EVAL "if not redis.call('ZSCORE', KEYS[1], ARGV[1]) then return redis.call('ZADD', KEYS[1], ARGV[2], ARGV[1]) else return 0 end;" 1 s1 m1 3
(integer) 0
127.0.0.1:6379> ZSCORE s1 m1
"2"
127.0.0.1:6379> EVAL "if not redis.call('ZSCORE', KEYS[1], ARGV[1]) then return redis.call('ZADD', KEYS[1], ARGV[2], ARGV[1]) else return 0 end;" 1 s1 m2 3
(integer) 1
127.0.0.1:6379> ZSCORE s1 m2
"3"
```

Here we add a member `m1` to set `s1` with score `1`, then we add it again with score `2`. ZSCORE reveals that the score has been changed.
Then with Lua script we add `m1` with score `3`, but only if ZSCORE returns `nil`, which is not true an as such doesn't change the score of `m1`.
But for new member `m2` with score `3` it works. 

These are only 3 simple use cases, but you can find it useful for many other things.

