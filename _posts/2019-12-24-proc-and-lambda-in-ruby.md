---
layout: post
title: "Proc and Lambda in Ruby"
date: 2019-12-24 19:56:36 +0100
categories: ['Ruby']
---

In Ruby, you can use `Proc` objects as anonymous functions. The `Proc` objects are also *callable* objects, which means that you can send a `call` message to it and the code inside the `Proc` object will be executed.

Procs are coming in two flavors: lambda and non-lambda (regular procs). What are the differences between them?



## Creation

- Lambdas can be created by `lambda {}` or `-> {}`
- Regular procs can be created by `Proc.new {}` or `proc {}`

```ruby
lambda {}.lambda?     #=> true
-> {}.lambda?         #=> true

proc {}.lambda?       #=> false
Proc.new {}.lambda?   #=> false
```



## Return

- In lambdas, `return` means exit from this lambda
- In regular procs, `return` means exit from embracing method (and will throw `LocalJumpError` if invoked outside the method)

```ruby
def test_return
  -> { return 3 }.call      # just returns from lambda into method body
  proc { return 4 }.call    # returns from method
  return 5
end

test_return   # => 4, return from proc
```



## Arguments

- In lambdas, arguments are treated in the same way as in methods: strict, with `ArgumentError` for mismatching argument number, and no additional argument processing;
- Regular procs accept arguments more generously: missing arguments are filled with `nil`, single [Array](https://ruby-doc.org/core-2.6.5/Array.html) arguments are deconstructed if the proc has multiple arguments, and there is no error raised on extra arguments.

```ruby
l = lambda {|x, y| "x=#{x}, y=#{y}" }
l.call(1, 2)      #=> "x=1, y=2"
l.call([1, 2])    # ArgumentError: wrong number of arguments (given 1, expected 2)
l.call(1, 2, 8)   # ArgumentError: wrong number of arguments (given 3, expected 2)
l.call(1)         # ArgumentError: wrong number of arguments (given 1, expected 2)

p = proc {|x, y| "x=#{x}, y=#{y}" }
p.call(1, 2)      #=> "x=1, y=2"
p.call([1, 2])    #=> "x=1, y=2", array deconstructed
p.call(1, 2, 8)   #=> "x=1, y=2", extra argument discarded
p.call(1)         #=> "x=1, y=", nil substituted instead of error
```



## Block

By default, the block of code is treated as a regular proc.

```ruby
def test_arguments
  [[1, 2]].map {|x, y| "x=#{x}, y=#{y}" } # It won't cause ArgumentError
end
test_arguments    #=> ["x=1, y=2"]

def test_return
  10.times {|x| return 'inside block'}
  return 'outside block'
end
test_return       #=> "inside block"
```

