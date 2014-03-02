---
layout: post
title: "JSON API with rails-api and active_model_serializers"
date: 2014-03-02 10:44:41 +0100
comments: true
categories:
- Ruby on Rails
- JSON
- API
- rails-api
- active_model_serializers
---

This post is based on my presentation given at [Lodz Ruby Users Group](http://lrug.pl/). The slides are [here](http://adamniedzielski.github.io/presentations/api-serializers/) (in Polish).

## Rails and JSON

Dealing with JSON in Rails is pretty easy and straightforward. The support is built into the framework, request data is automatically available in ```params``` hash. Then we just have to say ```render :json``` and we are done. 

This may be sufficient if JSON is one of the formats which we use. For instance, we can render both JSON and HTML in the same controller action. Or we may create an action intended for AJAX call which changes some model and returns JSON with ```status``` field.

On the other hand, we may use Ruby on Rails for building JSON API - app with the assumption that JSON is the only format we support. In such a case we know that the app won't render HTML. There is no View layer (or you can say that JSON is your View layer). We probably won't need cookies support and the code specific to handling browser requests.

<!-- more -->

## rails-api

Ruby on Rails has modular structure - you can opt out of the things which you don't want to use. It's described thoroughly in [this wonderful blog post](http://www.amberbit.com/blog/2014/2/14/putting-ruby-on-rails-on-a-diet/). But if we are talking about API-only Rails app [somebody](https://github.com/rails-api/rails-api#maintainers) already did it for us and published in the gem called **rails-api**.

#### Why should you care?

My personal reason is: "don't require things which you won't use". There might be bugs hiding there or security vulnerabilities connected with it. And of course there are other reasons:

* more lightweight application

I generated two fresh applications - one using ```rails-api new``` and the second using ```rails new```. The environment was consistent - Ruby 2.1.0, Rails 4.0.2. Then I started the server with ```rails s``` and measured RAM memory taken by server process. The results are following:

-> **37.4 MB vs. 43.8 MB for rails-api** <-

This is 15% less. Obviously when you start adding new gems the relative difference will be smaller.

* faster application

The same benchmarking environment as above. I created controller action which loads 100 ```User``` records from the database and renders them as JSON. I placed exactly the same code in rails-api application and regular rails application. Then I measured the server response time for a bunch of requests and took the avarage value. It looks like:

-> **31.0 ms vs. 35.2 ms for rails-api** <-

This is 12% faster.

* useful generator

The controller scaffold which comes with rails-api is really cool. It disables ```new``` and ```edit``` actions (they don't make sense because we do not display forms) and provides sensible default code inside other actions.

* easy installation and no maintenance costs

Last but not least, rails-api is super easy to install and learn. All you have to do to bootstrap new app is:

```bash
gem install rails-api
rails-api new my_app
```

What's more, you can re-enable some modules disabled by rails-api or just completely remove rails-api without any problem. rails-api is not a burden. It's worth to spend 10 minutes of your time learning it. [Check out the docs](https://github.com/rails-api/rails-api/blob/master/README.md)

## active_model_serializers

If you want to build JSON API it's good to have control over the generated JSON. And here we need the second gem - [active_model_serializers](https://github.com/rails-api/active_model_serializers).

Disclaimer: there are other gems meant to solve similar problem, for example [rabl](https://github.com/nesquena/rabl) and [Jbuilder](https://github.com/rails/jbuilder). I haven't used them in a project yet, so I can't make any comparisons. But active_model_serializers works for me.

Let's look at a sample serializer:

```ruby user_serializer.rb
class UserSerializer < ActiveModel::Serializer
  attributes :id, :first_name, :last_name, :email

  has_one :address
  has_many :packages
end
```

Serializers live in ```app/serializers``` directory. As you can guess ```UserSerializer``` is used to serialize ```User``` model. Using ```attributes``` we define what should be included in the generated JSON. You may think that this is repeating yourself (the fields are already defined in database schema), but in my opinion you rarely render all the fields, more often you want to hide some internals.

As you can see we embed associated records using familiar syntax: ```has_one``` and ```has_many```. Serializing ```Address``` and ```Package``` will be handled using ```AddressSerializer``` and ```PackageSerializer```, respectively. 

#### No magic

```ruby user_serializer.rb
class UserSerializer < ActiveModel::Serializer
  
  [...]

  attributes :full_name, :email_address

  def full_name
    "#{object.first_name} #{object.last_name}"
  end

  def email_address
    object.email
  end

  [...]
end
```

Serializers are just classes which inherit from ```ActiveModel::Serializer```. You can define regular methods and the model being serialized is accessed by ```object``` method.

And how does the code inside ```UsersController``` look like? It's here:

```ruby users_controller.rb
[...]

  def index
    @users = User.includes(:address, :packages)

    render json: @users
  end

[...]
```

Pretty simple, isn't it? You just say: "I want JSON" and you have JSON rendered with proper serializer.

The generated JSON will look similar to this:

```json
{
  "users":
    [
      {
        "id": 1,
        "first_name": "Some String",
        "last_name": "Another String",
        [...]
        "address": { "street": "Yet another string" },
        "packages": [
          { "id": 2, "status": "delivered" },
          { "id": 5, "status": "lost" }
        ]
      }
    ]
}
```

#### More about associations

Let's imagine following use case: we want to get information about a package given its id. And we want it to contain information about the owner (user). Difficult? Not really!

```ruby package_serializer.rb
class PackageSerializer < ActiveModel::Serializer
  attributes :id, :status

  has_one :user
end
```

What's different here from model code? Package ```belongs_to``` user, but here it says ```has_one``` user. From the point of view of serializers ```belongs_to``` is exactly the same as ```has_one```, hence we have only ```has_one``` and ```has_many```.

Now let's go back to ```UsersController#index``` after our changes. We hit the action again and what do we get this time?

```bash
     Failure/Error: Unable to find matching line from backtrace
     SystemStackError:
       stack level too deep
```

That's an infinite loop: user contains package, package contains user, user contains package...

How can we avoid this?

#### Solution #1

```ruby user_serializer.rb
class UserSerializer < ActiveModel::Serializer
  [...]

  has_many :packages, :embed => :ids

  [...]
end
```

```json
{
  "users":
    [
      {
        "id": 1,
        [...]
        "package_ids": [2, 5]
      }
    ]
}
```

We can use ```:embed``` option and include only ids of the packages. This has a drawback: if a client of our API wants not only id of the package, but also its status then he will have to make a separate request for each package. Certainly this is a situation which we want to avoid.

#### Solution #2

```ruby user_serializer.rb
class UserSerializer < ActiveModel::Serializer
  [...]

  has_many :packages, :serializer => ShortPackageSerializer

  [...]
end
```

```ruby short_package_serializer.rb
class ShortPackageSerializer < ActiveModel::Serializer
  attributes :id, :status
end
```

We use ```:serializer``` option to specify different serializer than that inferred from naming conventions. We create ```ShortPackageSerializer```, which doesn't contain user embeded. What's more you can put ```ShortPackageSerializer``` and ```PackageSerializer``` in the inheritance hierarchy so you DRY.

In my opinion this solution is pretty clean. We have separate class for each representation and we are able to share some pieces of code by using inheritance. Of course, this may become too complex if the inheritance hierarchy grows very big. However if we limit ourselves to 2-3 serializers per model the code should stay clear and maintainable.

## Summary

Use rails-api if you are building API-only application. It's easy to learn and maintenance won't hit you later, because you can opt out without any problems.

Use active_model_serializers for better control over the generated JSON.

#### Sample application

You can find my sample application [here](https://github.com/adamniedzielski/narwhale). It's a university project, so don't expect it to be a complete solution for any problem. However, it contains examples for everything which I covered above (plus some extra things like specs for the API).
