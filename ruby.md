---
layout: page
title: Ruby
permalink: /ruby/
---

[Ruby](https://www.ruby-lang.org) is a dynamic, object-oriented, programming language. With an expressive syntax it enables simplicity and productivity. Ruby appeared as an [open-source](https://github.com/ruby/ruby) project in 1995, initiated by [Yukihiro Matsumoto](https://twitter.com/yukihiro_matz).

{% highlight ruby %}
class Person
  def initialize(name)
    @name = name
  end

  def introduce
    "Hello, I'm #{@name}!"
  end
end

Person.new('Matz').introduce
# => "Hello, I'm Matz!"
{% endhighlight %}
