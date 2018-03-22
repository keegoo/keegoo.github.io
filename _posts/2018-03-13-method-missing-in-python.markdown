---
layout: post
title:  "method missing in Python"
date:   2018-03-12 19:20:00 +0800
categories: notes
---

Python doesn’t provide such thing as Ruby's `method_missing`. 

But you can impelment one with some Python magics.

### Ruby's method_missing

Ruby's `method_missing` function gives you access inside Ruby object to hanle situation when a method doesn't exist.

```ruby
class Dog

  def bark()                                               # instance method
    puts "wang wang !!!"
  end

  def method_missing(method_name, *arguments, &block)      # instance method missing
    puts 'needs this instance method #{method_name}'
  end

  def self.bark()                                          # class method
    puts "wang wang !!!"
  end

  def self.method_missing(method_name, *arguments, &block) # class method missing
    puts 'needs this class method #{method_name}'
  end
end
```

```ruby
dog = Dog.new
dog.bark()         # =>  wang wang !!!
dog.speak()        # =>  needs this instance method speak

Dog.bark()         # =>  wang wang !!!
Dog.speak()        # =>  needs this class method speak
```

So if you want to create methods dynamically, you could put whatever logic you want inside `method_missing`. For example:

```ruby
class Dog
  def method_missing(method_name, *arguments, &block)
    if method_name =~ /^(speak|say|sing|bark).*/
      puts "wang wang !!!"
    end
  end
end

dog = Dog.new
dog.sing      # => wang wang !!!
dog.say       # => wang wang !!!
dog.bark      # => wang wang !!!
dog.speak     # => wang wang !!!
```

There's no such thing as `method_missing` in Python. To implement `method_missing` you need to have a better understanding of Python innards. 

###  Python instance method missing

Here's the answer first.

```python
# instance method missing, for Python 3.x
class Dog():
  def bark(self):
    print("wang wang !!!")
  
  def __getattr__(self, name):
    def _method_missing(*args, **kwargs):
      print("needs this instance method %s" % name)
    return _method_missing

dog = Dog()                     # => wang wang !!!
dog.speak("A", "B", last="C")   # => needs this instance method speak
# notes: "A", "B" stored in *args; last="C" stores in **kwargs
```

What's happenning under the hood when you invoke `dog.speak()` in Python?

Here's a very good [stackoverflow answer][ruby-s-method-missing-in-python]. 

```text
o.f(x) is a two-step operation: 
  1) get the attribute defined in `o`.
  2) call it with parameter x.
If the first step fails when there's no attribute `f`, 
then `__getattr__` will be invoked. 
And, what `__getattr__` returns must be callable.' 
That's why you need to return a method definition instead of a value.
```

So when you call `dog.speak()`, `__getattr__` will be invoked, and you need to make sure the return is callable.

It's not that elegant, but straightforward engouth.

### Python class method missing

Here's the answer first 

```python
# class method missing, for python 3.x
class Animal(type):
  def __getattr__(self, name):
    def _method_missing(*args, **kwargs):
      print("needs this class method %s" % name)
    return _method_missing

class Dog(metaclass=Animal):
  @classmethod
  def bark(cls):
    print("wang wang !!!")

Dog.bark()                      # => wang wang !!!
Dog.speak("A", "B", last="C")   # => needs this class method speak
# notes: "A", "B" stored in *args; last="C" stores in **kwargs
```

How to understand it?

### type and metaclass

As everything in Python is an object, `class` itself is an object as well. Then who is the **class** of `Dog class`? Or `Dog class` is the instance of who?

The answer is built-in class `type`, `type` serves the role of being the class of classes.

```python
dog.__class__     # => <class '__main__.Dog'>
Dog.__class__     # => <class 'type'>
```

Actually, `tuple, list, int, float` are all objects, they're all `instances of type`.

```python
int.__class__     # => <class 'type'>
tuple.__class__   # => <class 'type'>
list.__class__    # => <class 'type'>
float.__class__   # => <class 'type'>
```

So !!! Let's do a comparison.

`dog` is instance of `Dog class`, and `Dog class` is instance of `type`.

If cannot find `speak` in `dog object`, then `Dog class __getattr__` will be invoked (as shown in instance-method-missing example),

If cannot find `speak` in `Dog class`, then `type __getattr__` will be invoked.

You cannot change `type`'s `__getattr__` as it's buit-in.

But you can change `Dog class`'s class by assign it's **metaclass** to other value. 

```python
# ...
class Dog(metaclass=Animal):      # << -- change class's class.
  @classmethod                    # notes: Animal must inherit from type.
  def bark(cls):
    print("wang wang !!!")
```

Now `Animal class` is the class of `Dog class`. `Animal class __getattr__` will be invoked if `Dog class`'s `speak` cannot be found.

If you want to know more on metaclass, here's a very good tutorial: [python metaclass by example][python-metaclass-by-example].

[ruby-s-method-missing-in-python]: https://stackoverflow.com/a/6955825/1476512

[python-metaclass-by-example]: https://eli.thegreenplace.net/2011/08/14/python-metaclasses-by-example
