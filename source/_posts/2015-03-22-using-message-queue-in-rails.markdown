---
layout: post
title: "Using message queue in Rails"
date: 2015-03-22 20:14:31 +0100
comments: true
categories:
- Rails
- Ruby on Rails
- Ruby
- message queue
- RabbitMQ
- application architecture
- design patterns
- clean code
- clean architecture
- asynchronous communication
---

This post describes the **application architecture pattern** which is (in general) nothing new, but (from my experience) rarely applied in the Rails world. I'm talking about the nice and simple abstraction - **message queue**. But let me start by describing the goals I want to achieve and some alternative solutions.

<!-- more -->

### Goals

* **split** application into a few smaller applications

Smaller applications are **easier to reason about**. You don't have to go through 50 classes, you can just read 10, because it's all you've got. When a new developer joins the team he has nice onboarding if you can tell him: "hey, start with this small piece of code, everything you need to know to implement this new feature is encapsulated here". 

* **separate** code for concepts which are not logically connected

Sometimes our application consists of concepts which are not strictly connected. For example, "presenting available products and managing information about available supplies" may have little to do with "taking payments from customers". If you are fixing bug in "presenting available products" part, you probably don't care about "taking payments from customers" part. Hence, you would like the application structure to show this **clear division**.

* use **new** languages and frameworks

We, developers, want to try and learn new languages, libraries, frameworks and technologies. If you make a small application with a shiny new tool and fail - the consequences are less severe, because you can quickly rewrite this small application. If you are going to make one big application, you will think twice before introducing a new tool. So, in some way, **smaller applications minimize the risk**.

### Solution 1: One database, multiple apps

This is the very first idea which may come to your mind - just point multiple applications to the one shared database. **Been there, done that, won't do that again!** Data is associated with validation logic. Either you duplicate this logic in every app or you extract it to Rails engine gem. Both solutions are hard to maintain (think about running migrations...) and you still have **strong coupling** in your system.

One case when this approach may work - one read-write app and many read-only apps, but I haven't tried it. 

### Solution 2: Expose REST API

As Rails devs we are pretty familiar with REST, so we can expose REST API in one of our apps and call this API in the other. This approach has many solid use cases so here I'm just listing some **weak points** to take into consideration:

- usually requests in Ruby are **blocking** - calling app has to wait for the response even if it's not interested in it

- requires authentication - we have to somehow ensure that our internal API is not used... well, externally.

- everything happens in server process - if you are calling your internal API you may end up using the same server process which is used for handling requests of your "real users". You would like to give your "real users" priority.

- calling app has **knowledge about receiving app** - you have to know which endpoints should be called and which parameters be passed. This introduces coupling.

### Solution 3: Message queue

Message queue is a really **nice abstraction**. Publisher just leaves messages at one end of the "pipe", consumer reads messages from the other end of the "pipe". It is **asynchronous**, because publisher does not wait for his message to be processed. Moreover, it decouples publisher from consumer, because publisher does not care what happens with his message and who will read it.

This architecture is also **resistant** to outages, at least when we assume that the queue service rarely breaks. If the consumer is not processing messages, nothing prevents publisher from adding more of them to the queue. When consumer starts to function again, it will process messages from the buffer (if they didn't take all of your memory).

### When it shines?

Message queue is really useful if we have some processing which happens out of the main business flow and the main business flow does not have to wait for the results of this processing. The most common example is custom **event tracking** - own analytics module. We just publish an event and continue execution without slowing anything down.

### RabbitMQ

RabbitMQ is a popular choice for message queue service, especially in Rails world. Honestly, I haven't tried different implementations, because RabbitMQ really **has everything I need**. If you have any experience - please share it in the comments.

There are Ruby gems for communicating with RabbitMQ and it's also easy to install and configure.

### Concepts

{% img center http://adamniedzielski.github.io/presentations/message-queue/rabbit1.png %}

In this diagram there are presented some concepts introduced by RabbitMQ. **Publisher** leaves messages in the **exchange**. Then they are routed from the **exchange** to multiple **queues**. There are many routing algorithms available - https://www.rabbitmq.com/tutorials/amqp-concepts.html#exchanges

**Workers** grab messages from **queue**. If there are multiple **workers** connected to one **queue**, they will be load balanced and the message will be delivered only to one of them.

### Easy case

If you feel overwhelmed - don't worry. Here is what you should start with:

{% img center http://adamniedzielski.github.io/presentations/message-queue/rabbit2.png %}


### Publishing

Now it's time for some code. It's really simple, because integration with RabbitMQ is simple. We will use two gems - ```bunny``` and ```sneakers```.

```ruby
# Gemfile
gem 'bunny'

# an initializer
connection = Bunny.new(host: 'localhost')
connection.start
channel = connection.create_channel

# a service
class RabbitPublisher

  def initialize(channel)
    self.channel = channel
  end

  def publish(exchange_name, message)
    exchange = channel.fanout(exchange_name, durable: true)
    exchange.publish(message.to_json)
  end

  private
    attr_accessor :channel
end
```

### Receiving

```ruby
# Gemfile
gem 'sneakers'

# an initializer
Sneakers.configure  daemonize: true,
                    amqp: "amqp://localhost",
                    log: "log/sneakers.log",
                    pid_path: "tmp/pids/sneakers.pid",
                    threads: 1,
                    workers: 1

# app/workers/events_worker.rb
class EventsWorker
  include Sneakers::Worker
  from_queue "events", env: nil

  def work(raw_event)
    event_params = JSON.parse(raw_event)
    SomeWiseService.build.call(event_params)
    ack!
  end
end
```

For details refer to documentation of [bunny](https://github.com/ruby-amqp/bunny) and [sneakers](https://github.com/jondot/sneakers).
