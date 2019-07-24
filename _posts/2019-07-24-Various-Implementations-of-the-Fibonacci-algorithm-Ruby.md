---
layout: post
title: Various implementations of the Fibonacci sequence algorithm in Ruby
---

The Fibonacci algorithm counts the sequence of numbers:

> 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144..

Furmula is quite simple:
- 0 is start number, further is 1, and again 1.
- Next number is counter by adding two previous numbers (here: 1 + 1 = 3).
- Further we repeat last step ( adding two previous numbers: 1 + 3 = 5). We can repeat this step forever.

## Rule:
> xn = xn-1 + xn-2 - where: `n` is a number of place in the sequnce, so `n - 1` - is number before and `n + 1` if number after `n`

## Challenge
Let say that we want find the first 35 elements of the Fibonacci sequence in the shortest time. How should we implement this algorithm to achieve this?

## Benchmark
Ruby has built benchmark tool. Here is a [documentation](https://ruby-doc.org/stdlib-2.5.3/libdoc/benchmark/rdoc/Benchmark.html).

We will use [benchmark-ips](https://github.com/evanphx/benchmark-ips) because it offers generating nice, redable reports and easily configurable warmup.
To use it we have to install `benchmark-ips` gem and require benchmark/ips module:
`gem install benchmark-ips`
`require "benchmark/ips"`

## Algorithms

### Recursion

The most popular solution to the problem using recursion. Implementation looks quite simple:

```ruby
def fib_recursion(n)
  return n if [0,1].include?(n)
  fib_recursion(n-1) + fib_recursion(n-2)
end
```

![_config.yml]({{ site.baseurl }}/images/posts/fibonacci/fibonacci_tree.png)
{: style="text-align: center;"}
source: [paulmouzas.github.io](https://paulmouzas.github.io/2015/01/23/memoization-fibonacci.html)
{: style="text-align: right; font-size: 10px;"}

As you can see - it recursively finds values for `n - 1` and `n - 2`

### Recursion with memoization

This implementation is based on previous one with additional caching. If you took a lookt at the tree above, you probably noticed that a some calculations are repeated (especially those with lower `n`). We can memoize them to gain (I hope) quite a big boost of performance:

```ruby
def fib_recursion_memoization(n)
  return memo[n] unless memo[n].nil?
  return n if [0,1].include?(n)
  memo[n] = fib_recursion_memoization(n-1) + fib_recursion_memoization(n-2)
end

def memo
  @memo ||= {}
end
```

### Non recursive version

In this example we used an interation. At first glance it seems to be slower than previous implementation, because we iterate n times that gives us O(n) of time complexity.

```ruby
def fib_non_recursive(n)
  prev_prev, prev, current_number = 0, 0, 1
  for i in 2..n
    prev_prev = prev
    prev = current_number
    current_number = prev_prev + prev
  end
  current_number
end
```

### Golden ratio

We can also use [Golden Ratio](https://en.wikipedia.org/wiki/Golden_ratio#Relationship_to_Fibonacci_sequence) to count n-element of fibonacci sequence. Formula is presented bellow, where φ (phi) is a golden number.

![_config.yml]({{ site.baseurl }}/images/posts/fibonacci/golden_ratio.png)
{: style="text-align: center;"}

```ruby
def fib_golden_ratio(n)
  sqrt = Math.sqrt 5
  golden_number = (1 + sqrt) / 2
  ((golden_number ** n - (1 - golden_number) ** n) / sqrt).to_i
end
```

### Matrix

```ruby
require "matrix"

def fib_matrix(n)
  (Matrix[ [1,1], [1,0] ] ** n)[1, 0]
end
```
### Fast doubling
https://www.nayuki.io/page/fast-fibonacci-algorithms

## Benchmark time 
