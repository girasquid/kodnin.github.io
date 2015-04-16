---
layout: page
title: JavaScript
permalink: /javascript/
---

[JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript) is a dynamic, prototype-based, programming language. It has first-class functions and supports imperative, object-oriented and functional styles. JavaScript appeared in 1995 and was designed by [Brendan Eich](https://twitter.com/BrendanEich).

{% highlight javascript %}
var factorial = function(n) {
  if (n == 0 || n == 1) {
    return 1;
  } else {
    return n * factorial(n - 1);
  }
};

[1, 2, 3, 4, 5].map(factorial);
// => [1, 2, 6, 24, 120]
{% endhighlight %}
