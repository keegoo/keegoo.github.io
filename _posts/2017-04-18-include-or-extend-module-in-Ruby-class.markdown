---
layout: post
title:  "include or extend module in Ruby class"
date:   2017-04-18 16:15:00 +0800
categories: jekyll update
---


### Basics

Let's say we have *class* A and *module* Utils.

If we `include Utils` inside A, then all of Utils' **instance** methods became A's **instance** methods.

If we `extend Utils` inside A, then all of Utils' **instance** methods became A's **class** methods.

For example:

```ruby
module Utils
  def say_hi
    puts "hi, i'm #{Utils.name}."
  end
end
```
```ruby
class A
  include Utils
end

A.new.say_hi      # => hi, i'm Utils.
A.say_hi          # => NoMethodError
```
```ruby
class A
  extend Utils
end

A.new.say_hi      # => NoMethodError
A.say_hi          # => hi, i'm Utils.
```

Two things we should keep in mind:

  - Both `include` and `extend` won't touch Utils' class methods.

```ruby
module Utils
  def self.say_hi # => say_hi is a class method this time
    puts "hi, i'm #{Utils.name}."
  end
end

class A
  include Utils
end

A.say_hi        # => NoMethodError

class B
  extend Utils
end

B.say_hi        # => NoMethodError
```

  - Both `include` and `extend` expect a *module* which means you cannot `include/extend a_class`.

```ruby
class Utils       # Utils is a class this time
  def say_hi
    puts "hi, i'm #{Utils.name}."
  end
end

class A
  include Utils   # => TypeError: wrong argument type Class (expected Module)
end
```

### extend self

By using `include/extend`, class A could make use of Utils' instance methods(a.k.a Mixin). 

But as *module* cannot be instanced, how can *module* access these useful instance methods itself?

For example: 

```ruby
module Utils
  def useful_method_1
    puts "do this"
  end

  def useful_method_2
    puts "do that"
  end
end

Utils.new.useful_method_1       # => NoMethodError, undefined method `new' for Utils:Module
Utils.useful_method_1           # => NoMethodError
```

It turns out we could `extend self` inside module Utils. This will turn Utils' instance methods into Utils' class methods. Then we could access them by `Utils.method_name`.

```ruby
module Utils
  def useful_method_1
    puts "do this"
  end

  def useful_method_2
    puts "do that"
  end

  extend self
end

Utils.useful_method_1           # => do this
Utils.useful_method_2           # => do that
```

The instance methods `useful_method_1` and `useful_method_2` are still there. So you still can use `include/extend Utils` inside class A to access them as instance methods.


### self.included hook

`self.included` will be called when the module been included by other class. It's one pupular Ruby meta programming hook.

For example:

```ruby
module Utils
  def self.included(boss)
    puts "wow, you're the boss: #{boss}"
  end
end 

class A
  include Utils
end
# =>  wow, you're the boss: A
```

With this hook, as the author of Utils module, we could decide which methods become A's **instance** methods and which methods become A's **class** methods. 

Examples [reference](http://www.railstips.org/blog/archives/2009/05/15/include-vs-extend-in-ruby/). In this example, I want `useful_method_1` became A's class method and `useful_method_2` became A's instance method.

```ruby
module Utils
  def self.included(boss)
    boss.extend(ClassMethods) # => boss refers to class A
  end

  module ClassMethods
    def useful_method_1       # => become A's class method
      puts "do this"
    end
  end

  def useful_method_2         # => become A's instance method
    puts "do that"
  end
end

class A
  include Utils
end

A.useful_method_1         # => do this
A.new.useful_method_2     # => do that
```
