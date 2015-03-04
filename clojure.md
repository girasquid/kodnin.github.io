---
layout: page
title: Clojure
permalink: /clojure/
---

[Clojure](http://clojure.org) is a dynamic, functional, programming language. As a Lisp dialect with immutable data structures it enables multithreaded designs. Clojure appeared as an [open-source](https://github.com/clojure/clojure) project in 2007, initiated by [Rich Hickey](https://twitter.com/richhickey).

{% highlight clojure %}
(defn fib-step [[a b]]
  [b (+ a b)])

(defn fib-seq []
  (map first (iterate fib-step [0 1])))

(take 10 (fib-seq))
;; => (0 1 1 2 3 5 8 13 21 34)
{% endhighlight %}
