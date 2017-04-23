---
layout: post
title:  "self in Ruby"
date:   2017-04-23 00:15:00 +0800
categories: jekyll update
---

### Executable class

Typically a *class* was consist of instance methods and class methods. What if we put some logical codes directly into class definition?

For example:

```ruby
class A
  ENABLED = true

  if ENABLED
    def do_this
      puts "do this"
    end
  else
    def do_that
      puts "do that"
    end
  end

  puts "hi, I'm #{self}"
end
```

If you execute above code, it will output "hi, I'm A". And *class* A will have `#do_this` instance method defined. 

This means Ruby will execute the codes between `class` and `end` and use the result as class definition.

### self definition

If you search on the web, you'll find two expressions on self:

  - `self` is a special variable that points to the object that "owns" the currently executing code.
  - `self` always refers to an instance.

In above example we can tell `self` refers to *class* A which meets the first expression - *points to the object that "owns" the currently executing code*.

It also meets the second expression as **A**(class) itself is an instance of class `Class`.

Following example demonstrate `self` in different situations:

```ruby
class A
  # inside class body, so `self` refers to A
  puts self         # => A

  def foo
    # inside an instance method, so `self` refers to instance of A
    return self
  end

  # inside class body, so `self` refers to A; same as `def A.bar`
  def self.bar
    # inside a class method, so `self` refers to A
    return self
  end
end

puts A.new.foo      # => #<A:0x007fc1cc072480>
puts A.bar          # => A
```

Here's a very good [stackoverflow answer](http://stackoverflow.com/a/12078094/1476512).