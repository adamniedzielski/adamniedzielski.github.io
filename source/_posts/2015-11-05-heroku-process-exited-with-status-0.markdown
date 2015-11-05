---
layout: post
title: "Heroku - process exited with status 0"
date: 2015-11-05 09:01:57 +0100
comments: true
categories:
- Ruby
- Rails
- Ruby on Rails
- deployment
- Sidekiq
- Heroku
---

Setting up [Sidekiq](https://github.com/mperham/sidekiq) on **Heroku** should be easy. Everything is so [well-described](https://github.com/mperham/sidekiq/wiki/Deployment#heroku) that I didn't expect any problems.

However, I encountered a **small puzzle** which took me a while to solve. I'm sharing the answer, because I didn't manage to find it anywhere and had to come up with it by myself.

<!-- more -->

This is what we had in Heroku logs:

```
Starting process with command `bundle exec sidekiq`
State changed from starting to up
Process exited with status 0
State changed from up to crashed
```

And that's it. No long exception stacktrace, no verbose errors, no hints - just these 4 lines. I started debugging with:

1. searching for the error in Google -> no results
2. checking the Redis connection -> Sidekiq web console is working just fine, wrong path
3. re-reading Sidekiq documentation about deployment -> no hints there

What I really should do at the beginning is read the logs **carefully** and **think**.

```Process exited with status 0```. The status code 0 means "everything is fine". And it was fine from the point of view of Sidekiq. Sidekiq was exiting, because it was running in the **daemon mode**. Heroku expects processes to run in the **foreground** mode.

The foreground mode is Sidekiq default setting, but some time ago we added ```:daemon: true``` to ```config/sidekiq.yml``` and forgot about it. Removing this line solved the puzzle!
