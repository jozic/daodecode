---
layout: post
title: "Helper functions for building streams in Racket"
date: 2013-02-25 00:05
comments: true
publish: false
categories: [racket, lots of parens]
---

In [Racket](http://racket-lang.org) you can define [streams](http://docs.racket-lang.org/reference/streams.html) which are kind of lazy lists.
 So here are a couple of helper functions for easier stream creation:
``` racket
; returns stream
(define (stream-builder seed current-element next-element)
  (letrec ([lazy-seq (lambda (x)
                       (cons (current-element x) (lambda () (lazy-seq (next-element x)))))])
    (lambda () (lazy-seq seed))))

; returns stream built sequantialy
(define (seq-stream-builder seed current-element)
  (stream-builder seed current-element (lambda (x) (+ 1 x))))
```
And usage:

``` racket
(define nats (seq-stream-builder 1 identity)) ; define stream of natural numbers
(nats) ; returns '(1 . #<procedure>)
((cdr (nats))) ; returns '(2 . #<procedure>)
```