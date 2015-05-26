---
layout: post
title: "Learn Chef - it's hard but worth it"
date: 2014-03-10 19:49:41 +0100
comments: true
categories:
- Chef
- automation
- infrastructure
---

Recently I've been forced by the circumstances to learn [Chef](http://www.getchef.com/chef/). It's simply **not possible** to manage about 15 servers **without any automation tool**. I'm not a master of shell scripting (neither I like it), so using Ruby in this field sounds great to me.

<!-- more -->

#### Update 26.05.2015

Today I would rather recommend using [Ansible](http://docs.ansible.com/) than Chef. I made a switch some time ago. Ansible is just simpler and works great for my purposes.

#### Here goes the original post:

I said *"forced to learn"*, although it's not that I don't like learning new technologies. In fact, I love it! However, there are technologies which are pleasure to learn and those which are painful. I approached Chef a couple of times before - read a blog post and then headed to the [official docs](http://docs.opscode.com/), but quickly backed out after reading a few pages. I don't know what the reason was - the layout, the language or the organisation of the pages. Maybe it was just me not focused enough on the content - although I tried very hard.

What I want to say in the blog post is that learning Chef was really hard for me. And if you are in a similar situation as I was - **you tried to learn Chef but failed** - I want to give you a bit of motivation to start again.

There are many benefits:

* Performing the same operation on each server is no longer a problem. When you have to repeat exactly the same set of commands on the second server - it's just **boring**. And when people are bored they often make mistakes. Each consecutive server means a greater chance of making a mistake. Not to mention *"oh! I fotgot about this one server"* mistakes.

* Consistent state on every server. No more surprises about the location of config files, log files and running services. You can be explicit about software versions to ensure that everything runs in **exactly the same way everywhere**

* **Fast** bootstraping of a new server. You can have a new server up and running in under 15 minutes or less (depends on whether you have to compile nginx with [passenger](https://www.phusionpassenger.com/) or not... :) ).

* Configuration **testing** is easy - just change the version of some package, deploy it to a fresh server instance, check if it works, destroy the test server and apply configuration to all your servers (assuming that you use cloud instances and bootstraping a new server for an hour costs you nothing)

* Community-driven **recipes**. You can leverage knowledge and best practices of other people. Probably they know better what the best default config is.

* Dependencies between servers stored in **central server**. Simple use case: *"open port 8080 on node A for nodes with role app"*. Far better than hardcoding IP addresses and forgetting about it when a new node is added. And that's just tip of the iceberg - you can do more with [Chef search](http://docs.opscode.com/essentials_search.html#partial-search)


I hope that the benefits above will motivate you again to start learning Chef. If you have any points which can be added to the list - **put them in the comments!**
