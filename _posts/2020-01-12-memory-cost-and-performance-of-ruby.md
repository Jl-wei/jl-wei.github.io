---
layout: post
title: "Memory cost and performance of Ruby"
date: 2020-01-12 21:54:11 +0100
categories: ['Ruby', 'Performance', 'Garbage collector']
---

This note is inspired by *Ruby Performance Optimization*.

As we know, Ruby is not a high performance language, but what makes Ruby code slow? In this article, I'm going to introduce the relation between Ruby memory cost and performance.

Unlike C/C++, Ruby use a *garbage collector* to for the memory management. The *garbage collector* attempts to reclaim garbage memory occupied by objects that are no longer in use by the program. And the action of garbage collection takes time.

Let's first install different version of Ruby (1.9.3, 2.1.5, 2.6.5), and then run this simple example in different version of ruby:

```ruby
require 'benchmark'

size = 1000
data = Array.new(size) { Array.new(size) { '0' * size } }

time = Benchmark.realtime do
  long_line = data.map(&:join).join
end

puts time.round(2)
```

How long does it takes? 

|                | 1.9.3 | 2.1.5 | 2.6.5 |
| -------------- | ----- | ----- | ----- |
| Execution time | 4.08  | 1.49  | 1.43  |

What if we disable the garbage collection?

```ruby
require 'benchmark'

GC.disable

size = 1000
data = Array.new(size) { Array.new(size) { '0' * size } }

time = Benchmark.realtime do
  long_line = data.map(&:join).join
end

puts time.round(2)
```

Let's run this new program and compare with the original one.

|                       | 1.9.3 | 2.1.5 | 2.6.5 |
| --------------------- | ----- | ----- | ----- |
| GC enabled            | 4.08  | 1.49  | 1.43  |
| GC disabled           | 1.03  | 1.08  | 1.02  |
| % of time spent in GC | 74.8  | 27.5  | 28.7  |

From this table, we can see that the raw performance of all modern Ruby interpreters is about the same, which the main difference in the garbage collector - more than 70% of the time in older Ruby and 25% of the time in modern versions.

Even through Ruby has made a significant performance improvement since Ruby 2.1, it's still not as fast as the GC of other languages, like Java or Go. One of the reason is that Ruby has a higher memory consumption, because in Ruby, everything is an object, which means that programs need extra memory to represent data as Ruby objects. 

As we know, **the more memory we use, the longer the GC takes to complete**. So let's check how much memory does our code takes.

```ruby
require 'benchmark'

GC.disable

puts "#{`ps -o rss= -p #{Process.pid}`.to_i / 1024} MB"

size = 1000
data = Array.new(size) { Array.new(size) { '0' * size } }

puts "#{`ps -o rss= -p #{Process.pid}`.to_i / 1024} MB"

time = Benchmark.realtime do
  long_line = data.map(&:join).join
end

puts "#{`ps -o rss= -p #{Process.pid}`.to_i / 1024} MB"
```

Result:

```bash
12 MB
1090 MB
3002 MB
```

Our object `data` takes about 1 GB, while the `long_line` takes 2 GB. But why does the `long_line` takes 2 GB rather than 1 GB? Because of the `line` object, this `line` we generate inside that block are intermediate results stored into memory until we can finally join them by the newline character.

Let's rewrite the code in a way without intermediate results.

```ruby
require 'benchmark'

GC.disable

puts "#{`ps -o rss= -p #{Process.pid}`.to_i / 1024} MB"

size = 1000
data = Array.new(size) { Array.new(size) { '0' * size } }

puts "#{`ps -o rss= -p #{Process.pid}`.to_i / 1024} MB"

time = Benchmark.realtime do 
  long_line = ''
  size.times do |i|
    size.times do |j|
      long_line << data[i][j]
    end
  end
end

puts "#{`ps -o rss= -p #{Process.pid}`.to_i / 1024} MB"
puts time.round(2)
```

Result:

```bash
12 MB
1090 MB
2044 MB
```

Now we save a lot the memory, how about the performance of the time?

|             | 1.9.3 | 2.1.5 | 2.6.5 |
| ----------- | ----- | ----- | ----- |
| GC enabled  | 4.08  | 1.49  | 1.43  |
| GC disabled | 1.03  | 1.08  | 1.2   |
| Optimized   | 0.85  | 0.76  | 0.73  |

By optimizing the memory use, we get a significant speedup. In Ruby, only when you are sure that the code spends a reasonable time in GC should you look further and try to locate algorithmic complexity or other sources of poor performance.

### Conclusion

* Memory consumption and garbage collection are among the major reasons why Ruby is slow.
* The raw performance of all modern Ruby interpreters is about the same, a memory-optimized program has the same performance in any modern Ruby versions.
* The 80-20 rule of Ruby performance optimization: 80% of performance improvements come from memory optimization, so optimize memory first.
