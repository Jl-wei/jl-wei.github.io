---
layout: post
title: "P, Puts and Print in Ruby"
date: 2019-12-22 12:13:06 +0100
categories: ['Ruby']
---

When you invoke the `print`, `puts` and `p` method in Ruby without any prefix, you are invoking the them from the `Kernel` module. What are the differences between them?

Let's create a new class `C`, we will use it in the following explanation.

```ruby
class C
  def initialize(s)
    @s = s
  end
  def to_s
    @s.to_s
  end
end

c = C.new 'Hello from C'
```



## Print

`print(obj, ...) → nil`

For each object, directly writes *obj*.`to_s` to the program's standard output. Returns `nil`.

```ruby
print 'a', 'b', 'c'
#=>
abc=> nil

print c
#=>
Hello from C=> nil
```



## Puts

`puts(obj, ...) → nil`

For each object, directly writes *obj*.`to_s` followed by a newline to the program's standard output. Returns `nil`.

```ruby
puts 'a', 'b', 'c'
#=>
a
b
c
=> nil

puts c
#=>
Hello from C
=> nil
```



## P

`p(obj) → obj`

`p(obj1, obj2, ...) → [obj, ...]`

`p() → nil`

For each object, directly writes *obj*.`inspect` followed by a newline to the program's standard output.

```ruby
p 'a', 'b', 'c'
#=>
"a"
"b"
"c"
=> ["a", "b", "c"]

p c
#=>
#<C:0x00007fee66812638 @s="Hello from C">
=> #<C:0x00007fee66812638 @s="Hello from C">
```

But what doest the function of method `inspect`?

By default, it returns a string containing a human-readable representation of *obj*. The default `inspect` shows the object's class name, an encoding of the object id, and a list of the instance variables and their values (by calling [inspect](https://ruby-doc.org/core-2.6.5/Object.html#method-i-inspect) on each of them).

The function of `inspect` depends on the implementation. What if we overriding the `inspect` method of class `C`?

```ruby
class C
  def inspect
    "#{@s} inspect".to_s
  end
end

p c
#=>
Hello from C inspect
=> Hello from C inspect
```







