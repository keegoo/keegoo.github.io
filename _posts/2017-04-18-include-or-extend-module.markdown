---
layout: post
title:  "include or extend module"
date:   2017-04-18 16:15:00 +0800
categories: jekyll update
---


### Basics

Let's say we have class A and module Utils.

If I `include Utils` inside class A, then all the **instance** methods of Utils became the **instance** methods of A.

If I `extend Utils` inside class A, then all the **instance** methods of Utils became the **class** methods of A.

For example:

```ruby
module Utils
  def say_hi
    puts "hi, i'm #{Utils.name}."
  end

  def self.wont_be_called
    puts "it's so sad i'm class method and won't be accessed by A"
  end
end
```
```ruby
class A
  include Utils
end

A.new.say_hi      # => hi, i'm Utils.
A.say_hi          # => NoMethodError
A.wont_be_called  # => NoMethodError
```
```ruby
class A
  extend Utils
end

A.new.say_hi      # => NoMethodError
A.say_hi          # => hi, i'm Utils.
A.wont_be_called  # => NoMethodError
```

Several things you should keep in mind:

  - Both `include` and `extend` have nothing to do with class methods of Utils module.
  - Both `include` and `extend` expect a module which means you cannot `include/extend a_class`.
  - Module cannot be instanced. You cannot `a_module.new`.


### extend self

Let's go through some knowledges before dive in:
  - class is meant for data and behavior; module is meant for behavior only.
  - module is more light weight compared with class.

So if you want to wrap behavior only, you'd better choose module over class.

Following ruby core libs are all modules:

```ruby
Base64.class    #=> Module 
Benchmark.class #=> Module 
FileUtils.class #=> Module 
Math.class      #=> Module
```

In following example. 

As module cannot be instanced, how can I access these useful methods?

```ruby
module Utils
  def useful_method_1
    puts "do this"
  end

  def useful_method_2
    puts "do that"
  end

  def self.will_be_called
    puts "so happy i can be accessed by #{Utils.name}"
  end
end

Utils.new.useful_method_1       # => NoMethodError, undefined method `new' for Utils:Module
Utils.useful_method_1           # => NoMethodError
Utils.will_be_called            # => so happy i can be accessed by Utils
```

It turns out we could `extend self` inside module Utils. This will turn Utils' instance methods into class methods. Then we could access them by `Utils.method_name`.

```ruby
module Utils
  def useful_method_1
    puts "do this"
  end

  def useful_method_2
    puts "do that"
  end

  def self.will_be_called
    puts "so happy i can be accessed by #{Utils.name}"
  end

  extend self
end

Utils.useful_method_1           # => do this
Utils.useful_method_2           # => do that
Utils.will_be_called            # => so happy i can be accessed by Utils
```

The instance methods `useful_method_1` and `useful_method_2` are still there. So you still can use `include/extend Utils` inside class A to access them as instance methods.


### self.included hook

`self.included` will be called when the module been included by other class. It's one pupular Ruby meta programming hook.

Take this for example:

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

Examples [refered to](http://www.railstips.org/blog/archives/2009/05/15/include-vs-extend-in-ruby/). In this example, I want `useful_method_1` became A's class method and `useful_method_2` became A's instance method.

```ruby
module Utils
  def self.included(boss)
    boss.extend(ClassMethods)
  end

  module ClassMethods
    def useful_method_1
      puts "do this"
    end
  end

  def useful_method_2
    puts "do that"
  end
end

class A
  include Utils
end

A.useful_method_1         # => do this
A.new.useful_method_2     # => do that
```