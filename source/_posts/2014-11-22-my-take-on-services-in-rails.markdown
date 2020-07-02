---
layout: post
title: "My take on services in Rails"
date: 2014-11-25 21:07:23 +0100
comments: true
categories:
- Rails
- Ruby on Rails
- Ruby
- services
- clean code
- clean architecture
- scaling Rails
---

It's been a while since my last post. I was writing my engineer's thesis which caused general disgust towards writing at all. Anyway, these sad times are over and here comes the shiny new blogpost about introducing **service layer in Rails applications**. It does not contain any breakthrough thoughts, but is rather a mixture of ideas I **learned from great Ruby developers**.

<!-- more -->

### Why?

We need a place to store the **domain logic** of our app. If you follow The Rails Way it can be:</br>
a) controller</br>
b) model

Controller is an **entry point** to our application. However, it's not the only possible entry point. I would like to have my logic accessible from:

- Rake tasks
- background jobs
- console
- tests

If I throw my logic into a controller it won't be accessible from all these places. So let's try "skinny controller, fat model" approach and move the logic to a model. But which one? If a given piece of logic involves ```User```, ```Cart``` and ```Product``` models - where should it live?

A class which inherits from ```ActiveRecord::Base``` already has a lot of responsibilities. It handles **query interface, associations and validations**. If you add even more code to your model it will quickly become an unmaintainable mess with hundreds of public methods. How can you change one line of your model's code with confidence that nothing breaks up? It's much easier when the whole class fits into a single screen of code.


### How to implement a service?

A service is just a regular Ruby object. Its class does not have to inherit from any specific class. Its name is a verb phrase, for example ```CreateUserAccount``` rather than ```UserCreation``` or ```UserCreationService```. It lives in ```app/services``` directory. You have to create this directory by yourself, but Rails will autoload classes inside for you.

```ruby app/services/create_user_account.rb
class CreateUserAccount

  def call(params)
    [...]
  end
end
```

The service **does one thing** and has one public method. I used to name it ```perform```, but ```call``` is slightly better. Lambda also responds to ```call``` so in your tests you have the possibility to mock service with lambda, which is quite convenient. 

### Splitting big services

As you implement a service object you may notice that it grows in size. It is usually a sign that this service has **more than one responsibility** and should be composed of a few smaller services. How to organize them? Here comes the example:

```ruby app/services/create_user_account/generate_token.rb
class CreateUserAccount
  class GenerateToken

    def call(user)
      [...]
    end
  end
end
```

```ruby app/services/create_user_account/send_welcome_email.rb
class CreateUserAccount
  class SendWelcomeEmail

    def call(user)
      [...]
    end
  end
end
```

```ruby app/services/create_user_account.rb
class CreateUserAccount
  
  def call(params)
    [...]
    generate_token.call(user)
    send_welcome_email.call(user)
    [...]
  end
end
```

So you have to create a new directory ```app/services/create_user_account``` and place there those extracted services. Each of them should be encapsulated in ```CreateUserAccount``` namespace. Again, Rails autoloads everything for you.

### Dependency injection

How can you obtain instance of this "child service" in your "parent service"? You could write something similar to:

```ruby app/services/create_user_account.rb
class CreateUserAccount
  
  def call(params)
    [...]
    generate_token = GenerateToken.new
    generate_token.call(user)
    [...]
  end
end
```

In such a case you **hardcode** instantiation of ```GenerateToken``` service inside ```CreateUserAccount```. Your are not able to easily provide mock implementation of ```GenerateToken``` inside your tests so you cannot test ```CreateUserAccount``` service in isolation.

The simple yet powerful solution [is described here](http://solnic.codes/2013/12/17/the-world-needs-another-post-about-dependency-injection-in-ruby/). Let's apply it in our case:

```ruby app/services/create_user_account.rb
class CreateUserAccount

  def self.build
    new(GenerateToken.build, SendWelcomeEmail.build)
  end

  def initialize(generate_token, send_welcome_email)
    @generate_token = generate_token
    @send_welcome_email = send_welcome_email
  end
  
  def call(params)
    [...]
    @generate_token.call(user)
    [...]
  end
end
```

```ruby app/services/create_user_account/generate_token.rb
class CreateUserAccount
  class GenerateToken

    def self.build
      new
    end

    [...]
  end
end
```

```ruby app/services/create_user_account/send_welcome_email.rb
class CreateUserAccount
  class SendWelcomeEmail

    def self.build
      new
    end

    [...]
  end
end
```

We pass all dependencies to the constructor of the service. But we also provide a **factory method** - ```build``` which knows the sane way to build the service.  

### Complete example

Here is the complete example which combines everything:

```ruby app/services/create_user_account.rb
class CreateUserAccount

  def self.build
    new(GenerateToken.build, SendWelcomeEmail.build)
  end

  def initialize(generate_token, send_welcome_email)
    @generate_token = generate_token
    @send_welcome_email = send_welcome_email
  end
  
  def call(params)
    # code which creates user model
    [...]
    @generate_token.call(user)
    @send_welcome_email.call(user)
    user
  end
end
```

```ruby app/services/create_user_account/generate_token.rb
class CreateUserAccount
  class GenerateToken

    # this service has no dependencies
    def self.build
      new
    end

    def call(user)
      [...]
    end
  end
end
```

```ruby app/services/create_user_account/send_welcome_email.rb
class CreateUserAccount
  class SendWelcomeEmail

    # this service has no dependencies
    def self.build
      new
    end

    def call(user)
      [...]
    end
  end
end
```

#### Update 27.12.2014

Following suggestions in the comments, I've prepared a dummy app to demonstrate usage of service objects. I hope it's at least a bit more concrete than the examples above. Here is the link: https://github.com/adamniedzielski/service-objects-example

#### Update 06.05.2019

Four and half years after writing this blog post and the Rails community still
does not have a standard way of writing service objects :) You can read what my
fellow programmer Paweł thinks about [service objects in Rails](https://pawelurbanek.com/2018/02/12/ruby-on-rails-service-objects-and-testing-in-isolation/).
