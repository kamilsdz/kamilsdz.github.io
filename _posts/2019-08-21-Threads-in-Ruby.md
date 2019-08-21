---
layout: post
title: Threads in Ruby - when does it make sense?
---

CRuby has a Global Lock Interpreter, that imposes certain limitations on us. I wrote about it a bit in my previous article: [Multiprocessing in Ruby ](https://naturaily.com/blog/multiprocessing-in-ruby){:rel="nofollow"}.

But does this means that multithreading in Ruby is useless? Of course not! And in this short post I’ll show you why.

### IO

We know that with GIL only one thread `can execute at a time`. Going further - there is something like an execution queue and thanks to it we can make use of the threads.

> Most modern CPUs use an instruction queue. Several instructions are waiting in the queue, ready to be executed. Separate electronic circuitry keeps the instruction queue full while the control unit is executing the instructions. [Robert G. Plantz](https://bob.cs.sonoma.edu/IntroCompOrg-RPi/sec-progexec.html)

Take a look at this simple example:

```ruby
require "benchmark/ips"

def execute
  1 + 1
  sleep 1
  2 + 2
end

Benchmark.ips do |bm|
  bm.time = 3
  bm.warmup = 1

  bm.report("single thread") do
    10.times { execute }
  end

  bm.report("multi threaded") do
    threads = []
    10.times do
      threads << Thread.new { execute }
    end
    threads.each { |thread| thread.join }
  end

  bm.compare!
end
```

➜ 
```
Warming up --------------------------------------
       single thread     1.000  i/100ms
      multi threaded     1.000  i/100ms
Calculating -------------------------------------
       single thread      0.100  (± 0.0%) i/s -      1.000  in  10.034372s
      multi threaded      0.997  (± 0.0%) i/s -      3.000  in   3.009947s

Comparison:
      multi threaded:        1.0 i/s
       single thread:        0.1 i/s - 10.00x  slower
```


What happened? When we executed the second example - threads have executed code 10 times faster. The reason for this is simply - our `execute` method uses `sleep`.

When one thread doesn’t compute anything (when is waiting) - CPU starts executing instructions from the second thread. It works because waiting (ie. for I/O) isn’t an execution - it is just a waiting, so next tasks from the queue can be executed - that's how concurrency works. 

I presented this in a simplified diagram below:

![_config.yml]({{ site.baseurl }}/images/posts/ruby-threads/scheme.png)
{: style="text-align: center;"}

The conclusion is simple. We can use multithreading to improve performance wherever the thread has to wait - mainly in Input-Output operations.

