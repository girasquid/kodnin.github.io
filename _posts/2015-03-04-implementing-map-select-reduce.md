---
layout: post
title:  "Implementing map, select and reduce"
date:   2015-03-04 14:00:00
categories: ruby
---

Mixing in the [Enumerable](http://ruby-doc.org/core-2.2.0/Enumerable.html) module provides a class traversal, searching and sorting functionality. In return the module expects a class to have the `each` and `<=>` methods. To deepen my personal understanding of Enumerable I decided to implement its established `map`, `select` and `reduce` methods. In the context of the [Array](http://ruby-doc.org/core-2.2.0/Array.html) class I will show you the expectations and implementation I came up with. At the end of this post I will also provide a link to an equivalent endeavor in the [Hash](http://ruby-doc.org/core-2.2.0/Hash.html) class context.

# Methods, blocks and yield

In order to implement `each` for a class, but also when implementing its functional cousins, we need to understand how methods process blocks and pass arguments to them. In the end we want to be able to provide our own blocks to these methods.

{% highlight ruby %}
def give_me_a_block
  arg = 10
  val = yield(arg)
  val
end

give_me_a_block { |x| x * x }
# => 100

# is equivalent to:

def give_me_a_block
  yield(10)
end

give_me_a_block { |x| x * x }
# => 100
{% endhighlight %}

`arg` is a local variable in the `give_me_a_block` method. It is passed into the block using the `yield` method. `x` is a block parameter that gets assigned the value of `arg`. `x` is then used as a local variable in the block. The `yield` method returns the value of applying the block to its arguments. This value is assigned to `val` and can be reused.

# A test object

Let us define an Array so we can use it in our expectations.

{% highlight ruby %}
describe Array do
  let(:array) { [1, 2, 3] }
end
{% endhighlight %}

# Array#each

A requirement for the relevant Enumerable methods to work is the presence of an `each` method in a class. Array comes with this method, but why not reinforce our understanding of blocks and the `yield` method? When passed a block the `each` method typically returns the object it was called on. Therefore we use the accumulator `acc` to check if it correctly iterates over all elements from the test object.

{% highlight ruby %}
describe Array do
  it '#each2' do
    acc = []
    expect(array.each2 { |n| acc.push(n + 1) }).to eq(array)
    expect(acc).to eq([2, 3, 4])
  end
end
{% endhighlight %}

{% highlight ruby %}
class Array
  def each2
    i = 0
    while i < length
      yield(self[i])
      i += 1
    end
    self
  end
end
{% endhighlight %}

# Mix Enumerable into Array

Before we define `map`, `select` and `reduce` in our own Enumerable module we mix the empty module into the Array class. This way the methods, that are yet to be defined, will be available as instance methods on Array when we start defining them in the module.

{% highlight ruby %}
module Enumerable2
end

class Array
  include Enumerable2
end
{% endhighlight %}

# Enumerable#map

`map` is defined in terms of `each` and sets up an accumulator `acc`. All processed elements returning from the block are pushed onto `acc` and then the accumulator is returned. This idea of defining a method in terms of `each` and using an accumulator will also be present in the implementations of `select` and `reduce`.

{% highlight ruby %}
describe Array do
  it '#map2' do
    expect(array.map2 { |n| n * 3 }).to eq([3, 6, 9])
  end
end
{% endhighlight %}

{% highlight ruby %}
module Enumerable2
  def map2
    acc = []
    each2 { |elem| acc.push(yield(elem)) }
    acc
  end
end
{% endhighlight %}

# Enumerable#select

In the `select` method only those elements for which the block returns a truthy value are pushed onto the accumulator. ``select`` acts as a filter because of this conditional accumulation.

{% highlight ruby %}
describe Array do
  it '#select2' do
    expect(array.select2 { |n| n.even? }).to eq([2])
  end
end
{% endhighlight %}

{% highlight ruby %}
module Enumerable2
  def select2
    acc = []
    each2 { |elem| acc.push(elem) if yield(elem) }
    acc
  end
end
{% endhighlight %}

# Enumerable#reduce

This particular implementation of the `reduce` method requires a starting value for the accumulator `acc`, passed in as an argument. Also it requires you to provide a block with two block parameters instead of one. The first, named `sum` in the expectation, is used to iteratively build up a value. The second block parameter `n` represents the current element. This allows the method to sum all elements in the test object. In the implementation `acc` is assigned the starting value and then passed to the `yield` method, together with the current element `elem`. From there on it is reassigned the return value of the provided block on each iteration and finally returned.

{% highlight ruby %}
describe Array do
  it '#reduce2' do
    expect(array.reduce2(0) { |sum, n| sum + n }).to eq(6)
  end
end
{% endhighlight %}

{% highlight ruby %}
module Enumerable2
  def reduce2(acc)
    each2 { |elem| acc = yield(acc, elem) }
    acc
  end
end
{% endhighlight %}

Voila, all our methods are now in line with the given expectations. Please be aware that this is not a full implementation of the original Enumerable methods. For example, calling `map` without a block should return an [Enumerator](http://ruby-doc.org/core-2.2.0/Enumerator.html), you should be able to call `reduce` without an argument or provide a [Symbol](http://ruby-doc.org/core-2.2.0/Symbol.html) as its sole argument, and so on. Nonetheless, I hope the implementation of these methods has given you a better understanding of their semantics and inner workings. If you are interested in the Hash implementation or if you want to check the code visit [Understanding Ruby](https://github.com/kodnin/understanding-ruby).
