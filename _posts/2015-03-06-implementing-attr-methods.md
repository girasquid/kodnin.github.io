---
layout: post
title:  "Implementing the attr_* methods"
date:   2015-03-06 14:00:00
categories: ruby
---

The family of attr_* methods is comprised of `attr_reader`, `attr_writer` and `attr_accessor`. They are convenient shortcuts and defined in the [Module](http://ruby-doc.org/core-2.2.0/Module.html) class. Every class has access to these methods, since Module is the superclass of the [Class](http://ruby-doc.org/core-2.2.0/Class.html) class.

# The basic idea

The idea of reader and writter methods is straightfoward. However, we want our attr_* methods to define these reader and writer methods for us. In order to implement this we will need to work with some metaprogramming techniques.

{% highlight ruby %}
class Hacker
  def pseudonym
    @pseudonym
  end

  def pseudonym=(pseudonym)
    @pseudonym = pseudonym
  end
end

hacker = Hacker.new
hacker.pseudonym = 'kodnin'
hacker.pseudonym
# => "kodnin"
{% endhighlight %}

# Metaprogramming techniques

Before we turn to the attr_* methods we are going to look at the `class_eval` and `define_method` methods, which are often used in metaprogramming. `class_eval` evaluates code in the context of the class it is called on. `define_method` allows us to define instance methods for classes.

{% highlight ruby %}
Hacker.class_eval { self }
# => Hacker

define_method(:break_code) { |code| code.reverse }

break_code('drowssap')
# => "password"
{% endhighlight %}

We can combine this knowledge by defining `break_code` in the context of the Hacker class with `class_eval`. Since `define_method` defines an instance method in the receiver it will be an instance method for Hacker instances.

{% highlight ruby %}
Hacker.class_eval do
  define_method(:break_code) { |code| code.reverse }
end

hacker.break_code('drowssap')
# => "password"
{% endhighlight %}

To access instance variables within an object we can use `instance_variable_get` and `instance_variable_set`.

{% highlight ruby %}
hacker.instance_variable_get(:@real_name)
# => nil

hacker.instance_variable_set(:@real_name, 'John Doe')
# => "John Doe"

hacker.instance_variable_get(:@real_name)
# => "John Doe"
{% endhighlight %}

# Module#attr_reader

Now let us apply these patterns for the attr_* methods. In order to retrieve an instance variable from an object we turn to `instance_variable_get`.

{% highlight ruby %}
describe Accessor do
  it '.attr_reader2' do
    class TestReader
      extend Accessor
      attr_reader2 :foo
    end

    obj = TestReader.new
    expect(obj.foo).to be_nil
    obj.instance_variable_set(:@foo, 'bar')
    expect(obj.foo).to eq('bar')
  end
end
{% endhighlight %}

{% highlight ruby %}
module Accessor
  private

  def attr_reader2(*names)
    class_eval do
      names.each do |name|
        define_method(name) do
          instance_variable_get("@#{name}")
        end
      end
    end
  end
end
{% endhighlight %}

# Module#attr_writer

Similarly, for setting an instance variable of an object we turn to `instance_variable_set`. You can see that we deliberately classify all of our attr_* methods as private, in line with the original methods.

{% highlight ruby %}
describe Accessor do
  it '.attr_writer2' do
    class TestWriter
      extend Accessor
      attr_writer2 :foo
    end

    obj = TestWriter.new
    expect(obj.foo = 'bar').to eq('bar')
    expect(obj.instance_variable_get(:@foo)).to eq('bar')
  end
end
{% endhighlight %}

{% highlight ruby %}
module Accessor
  private

  def attr_writer2(*names)
    class_eval do
      names.each do |name|
        define_method("#{name}=") do |val|
          instance_variable_set("@#{name}", val)
        end
      end
    end
  end
end
{% endhighlight %}

# Module#attr_accessor

`attr_accessor` can be implemented in terms of `attr_reader` and `attr_writer` since it combines its functionality. The `*` (splat) operator is used to correctly pass on a variable number of names.

{% highlight ruby %}
describe Accessor do
  it '.attr_accessor2' do
    class TestAccessor
      extend Accessor
      attr_accessor2 :foo
    end

    obj = TestAccessor.new
    expect(obj.foo).to be_nil
    expect(obj.foo = 'bar').to eq('bar')
    expect(obj.foo).to eq('bar')
  end
end
{% endhighlight %}

{% highlight ruby %}
module Accessor
  private

  def attr_accessor2(*names)
    attr_reader2(*names)
    attr_writer2(*names)
  end
end
{% endhighlight %}

So, in implementing `attr_reader`, `attr_writer` and `attr_accessor` we have seen some metaprogramming techniques. We looked at `class_eval` and `define_method` to define our methods in the proper context. `instance_variable_get` and `instance_variable_set` were used to retrieve and set internal state for an object. That was it, we have implemented the attr_* methods. If you want to review this code you can visit [Understanding Ruby](https://github.com/kodnin/understanding-ruby).
