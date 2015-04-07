---
layout: post
title: "Token authentication with Tiddle"
date: 2015-04-04 18:29:32 +0200
comments: true
categories:
- Ruby
- Ruby on Rails
- Rails
- gem
- Tiddle
- JSON
- API
- token authentication
- Devise
- multiple tokens
---

This post describes how to use [Tiddle](https://github.com/adamniedzielski/tiddle/) - gem for token authentication which I created. Tiddle is a Devise authentication strategy which supports **multiple tokens per user**.

<!-- more -->

### Motivation

I needed token authentication strategy for Devise in JSON API application. I started by using [Simple Token Authentication gem](https://github.com/gonzalo-bulnes/simple_token_authentication). It served me well until I realized I need support for multiple tokens per model.

Imagine a following scenario: your API has **two clients** - web application and mobile application. User A signs in to web application and receives a token. Then he signs in to mobile application and receives the **very same token** (because there is only one token per user). At this point, user A decides to log out from web application, so we generate new token for him and the old token becomes invalid. What is the consequence? User A is also logged out of mobile application.

This is **not a user-friendly behaviour**. We don't want the user to be logged out of all applications, we want him to be logged out of just one application. The solution to this problem is to issue a new token after each successful log in attempt and store **multiple tokens** per each user. This is possible with Tiddle.

### Integration with your app

Installation goes as usual, but I include it here for completeness. Add this line to your application's Gemfile:

```ruby
gem 'tiddle'
```

And then execute ```bundle install```.

You have to **create model** which holds authentication tokens. You can choose arbitrary model name, but following fields have to be included: ```body```, ```last_used_at```, ```ip_address``` and ```user_agent```. For example, you can generate such a migration:

```
rails g model AuthenticationToken body:string user:references last_used_at:datetime ip_address:string user_agent:string
```

Then you have to specify ```:token_authenticatable``` in the model used for Devise authentication:

```ruby
class User < ActiveRecord::Base
  devise :database_authenticatable, :registerable,
         :recoverable, :trackable, :validatable,
         :token_authenticatable # here it is

  [...]
end
```

This model should also include **association** called ```authentication_tokens``` (the name is important), so tokens can be looked up and created inside Tiddle.

```ruby
class User < ActiveRecord::Base
  [...]

  has_many :authentication_tokens
end
```

The last step is to subclass ```Devise::SessionsController``` as [described in Devise documentation](https://github.com/plataformatec/devise#configuring-controllers). In ```create``` action we check provided email and password. If they are valid, we create new authentication token and return it in the response. In ```destroy``` action we expire current token (or do nothing if user is not authenticated).

```ruby
class Users::SessionsController < Devise::SessionsController

  def create
    user = warden.authenticate!(auth_options)
    token = Tiddle.create_and_return_token(user, request)
    render json: { authentication_token: token }
  end

  def destroy
    Tiddle.expire_token(current_user, request) if current_user
    render json: {}
  end

  private

    # this is invoked before destroy and we have to override it
    def verify_signed_out_user
    end
end
```

**And that's it!** If you want to require authenticated user in some controller, just follow standard Devise way:

```ruby
class PostsController < ApplicationController
  before_action :authenticate_user! # nothing fancy

  def index
    render json: Post.all
  end
end
```

Every request to this endpoint has to include ```X-USER-EMAIL``` and ```X-USER-TOKEN``` headers, so authentication strategy can look up user by email and then check validity of token.

### Usage in AngularJS client

This is a short example of token authentication written in AngularJS. In our client we have to:

* provide email and password to **obtain** token
* **save** email and token in cookies
* **send** email and token with every request

This is a simplified controller action which makes request to our API:

```javascript
angular.module("app").
  controller("LoginController", function ($scope, $http, $cookies) {

    $scope.login = function() {
      $http.post('http://localhost:3000/users/sign_in.json', { user: { email: $scope.email, password: $scope.password } }).
        then(function (response) {
          $cookies.user_email = $scope.email;
          $cookies.user_token = response.data.authentication_token;
        });
    };
  });
```

And this is a **request interceptor** which adds authentication headers to every request:

```javascript
angular.module("app").
  config(function($httpProvider) {
    $httpProvider.interceptors.push(function($cookies) {
      return {
        'request': function(config) {
          config.headers['X-USER-EMAIL'] = $cookies.user_email;
          config.headers['X-USER-TOKEN'] = $cookies.user_token;
          return config;
        }
      };
    });
  });
```

### Deleting old tokens

After some time your database may be full of old tokens which are no longer used. They are the result of sign-ins which were never followed by sign-out. Tiddle has it covered - you can invoke ```Tiddle.purge_old_tokens(user)```. You can do it in a Rake task:

```ruby
task purge_old_authentication_tokens: :environment do
  User.find_each do |user|
    Tiddle.purge_old_tokens(user)
  end
end
```

### Conclusion

I hope someone can benefit from Tiddle, that's why I open sourced it. If you find any bugs, please report them at Github Issues.
