---
layout: post
title: "Fighting Paperclip Errno::ENOMEM error"
date: 2014-02-05 13:36:03 +0100
comments: true
categories:
- Ruby on Rails
---

We are using [Paperclip](https://github.com/thoughtbot/paperclip) to handle image uploads in one of our applications and after migrating from 4 GB instances to a number of really small 512 MB instances we started to experience an error which looked similar to this:

```
Errno::ENOMEM (Cannot allocate memory - identify -format %wx%h '/tmp/stream20130201-23766-d5htny-0.png[0]')
```

This was rather weird - we are running one app instance on each server, so the amount of free RAM memory is not critically low. In fact it should be enough to run ```identify```.
<!-- more -->

To find the root cause I had to dig deeper into Paperclip and learn how Ruby and Linux handle processes. As you probably know Paperclip uses [ImageMagick](http://www.imagemagick.org) under the hood for image processing. ```identify``` is just one of the binaries provided with ImageMagick.

But how does Paperclip invoke ImageMagick? It utilizes [cocaine](https://github.com/thoughtbot/cocaine) gem (created by [thoughtbot](http://thoughtbot.com/), too). It's a small library for calling commandline programs. It has a few pluggable runners, but all of them are forking the process in some way or another. This means that a new temporary process is created each time ImageMagick is needed. 

{% blockquote Wikipedia http://en.wikipedia.org/wiki/Fork_%28system_call%29 %}
(...) the fork operation creates a separate address space for the child. The child process has an exact copy of all the memory segments of the parent process (...)
{% endblockquote %}

At first I thought: "That's it!". Our Rails process takes ~150 MB, so it's possible that the exact copy won't fit into the remaining free memory. But here goes the second part of the article about forking:

{% blockquote Wikipedia http://en.wikipedia.org/wiki/Fork_%28system_call%29 %}
(...) though if copy-on-write semantics are implemented, the physical memory need not be actually copied. Instead, virtual memory pages in both processes may refer to the same pages of physical memory until one of them writes to such a page: then it is copied.(...)
{% endblockquote %}

Copy on Write is widely implemented in modern Linux distributions. If you are running Linux on your local workstation (I have absolutely no idea how this behaves on a Mac) you can check this out: 

``` ruby
a = (1..50_000_000).to_a
`sleep 120`
```

The first line allocates some significant amount of memory and the second one calls commandline sleep program using [backtick operator](http://ruby-doc.org/core-2.1.0/Kernel.html#method-i-60). If you run this script you will see that sleep is not copying the whole memory owned by Ruby. Copy on Write indeed works!

So why does it sometimes fail with "Cannot allocate memory" on small instances? Let's try another experiment, this time allocating 10 times more memory.

``` ruby
a = (1..500_000_000).to_a

if system('ls')
  puts "system works"
else
  puts "system fails"
end

begin
  `ls`
rescue => e
  puts "ls fails"
else
  puts "ls works"
end
```

You will have to adjust the number in first line in such a way that the Ruby process takes more memory than the amount of remaining free memory. For instance, in my case it looks like that:

* total memory: 8 GB
* memory taken by this Ruby process: 3.6 GB
* memory taken by other stuff: 2 GB
* remaining free memory: 2.4 GB

On my machine running the above script yields:
``` plain
system fails
ls fails
```

So here goes the rule: **to create the child process, free memory must be greater than the memory taken by the parent process**. You can read more about this googling ["linux memory overcommit"](https://www.google.pl/#q=linux+memory+overcommit). Operating system does not know how much memory the child process will need. Potentially it can change the whole memory owned by parent, which would result in a complete copy of parent's memory.

And the solution? I found it in this quite cryptic, but knowledge-packed article:
http://www.oracle.com/technetwork/server-storage/solaris10/subprocess-136439.html

It suggests using posix_spawn() to create the process. Do we have posix_spawn() in Ruby? Yes, we do! https://github.com/rtomayko/posix-spawn

Let's try it out:
``` plain
gem install posix-spawn
```

``` ruby
a = (1..500_000_000).to_a

require 'posix/spawn'
POSIX::Spawn::spawn('ls')
```

This time creating child process should succeed.

Can we use posix-spawn with cocaine? Yes, we can! In fact, there is a section in the Readme devoted to this: https://github.com/thoughtbot/cocaine#posix-spawn

## Final solution

Ensure that you are running the latest version of cocaine:
```
bundle show cocaine
```

It should be 0.5.3 (at the time of writing this blog post). If it's not - update paperclip at least to 3.5.3 - this version for sure works with cocaine 0.5.3. Then add to your Gemfile:

``` ruby Gemfile
gem 'posix-spawn'
```

And ```bundle install```. That's it. Now cocaine uses posix-spawn and invoking ImageMagick succeeds even if the free memory is lower than the memory taken by Rails process.
