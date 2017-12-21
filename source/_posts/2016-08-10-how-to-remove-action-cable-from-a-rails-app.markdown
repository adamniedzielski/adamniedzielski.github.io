---
layout: post
title: "How to remove Action Cable from a Rails app"
date: 2016-08-10 19:04:07 +0200
comments: true
categories:
- Action Cable
- ActionCable
- Rails
- Ruby on Rails
- Ruby
- Rails 5
- Rails 5 upgrade
- delete Action Cable
- deactivate Action Cable
- websockets
---

This is your complete guide to **removing Action Cable from a Rails 5 app**.

Some people need WebSockets in their app while other people do not need it at all or want to use other solution such as [message_bus](https://github.com/SamSaffron/message_bus). Keep your application clean and delete Action Cable specific stuff.

<!-- more -->

**Updated: 21.12.2017**

### Scenarios:

1. [I am upgrading from Rails 4](#upgrading-from-rails4)
2. [I generated Rails 5 app with Action Cable](#generated-rails5-with-action-cable)
3. [I want to generate Rails 5 app without Action Cable](#skip-action-cable)

<h2 id="upgrading-from-rails4">
  I am upgrading from Rails 4
</h2>

1) Open `config/application.rb`.

2) If the file contains
    
```ruby
require 'rails/all'
```

your are unnecessarily loading Action Cable code.

3) Replace `require 'rails/all'` with:

```ruby
require "rails"
# Pick the frameworks you want:
require "active_model/railtie"
require "active_job/railtie"
require "active_record/railtie"
require "action_controller/railtie"
require "action_mailer/railtie"
require "action_view/railtie"
require "sprockets/railtie"
require "rails/test_unit/railtie"
```

4) (optional) Open `Gemfile` and replace:

```ruby
gem "rails"
```

with:

```ruby
gem "activerecord"
gem "activemodel"
gem "actionpack"
gem "actionview"
gem "actionmailer"
gem "activejob"
gem "activesupport"
gem "railties"
gem "sprockets-rails"
```

In general, list only the gems that you are really using as your dependencies.
Please note that trimming down your `Gemfile` will only have an effect if none of your other dependencies depend on `rails` gem. You can verify that by searching
for `rails` in `Gemfile.lock`.

<h2 id="generated-rails5-with-action-cable">
  I generated Rails 5 app with Action Cable
</h2>

1) If in `config/application.rb` you have `require 'rails/all'` - replace it with:

```ruby
require "rails"
# Pick the frameworks you want:
require "active_model/railtie"
require "active_job/railtie"
require "active_record/railtie"
require "action_controller/railtie"
require "action_mailer/railtie"
require "action_view/railtie"
require "sprockets/railtie"
require "rails/test_unit/railtie"
``` 

2) If in `config/application.rb` you have `require "action_cable/engine"` - remove this line.

3) Remove `app/assets/javascripts/cable.js` file.

4) Remove `app/assets/javascripts/channels` directory.

5) Remove `app/channels` directory.

6) Remove `config/cable.yml` file.

7) From `config/environments/production.rb` remove

```ruby
# Mount Action Cable outside main process or domain
# config.action_cable.mount_path = nil
# config.action_cable.url = 'wss://example.com/cable'
# config.action_cable.allowed_request_origins = [ 'http://example.com', /http:\/\/example.*/ ]
```

8) From `Gemfile` remove:

```ruby
# Use Redis adapter to run Action Cable in production
# gem 'redis', '~> 3.0'
```

9) If you added `action_cable_meta_tag` to `app/views/layouts/application.html.erb`, remove it from there.

10) (optional) Open `Gemfile` and replace:

```ruby
gem "rails"
```

with:

```ruby
gem "activerecord"
gem "activemodel"
gem "actionpack"
gem "actionview"
gem "actionmailer"
gem "activejob"
gem "activesupport"
gem "railties"
gem "sprockets-rails"
```

In general, list only the gems that you are really using as your dependencies.
Please note that trimming down your `Gemfile` will only have an effect if none of your other dependencies depend on `rails` gem. You can verify that by searching
for `rails` in `Gemfile.lock`.

<h2 id="skip-action-cable">
  I want to generate Rails 5 app without Action Cable
</h2>

1) `rails new my-app-name --skip-action-cable`

2) (optional) Open `Gemfile` and replace:

```ruby
gem "rails"
```

with:

```ruby
gem "activerecord"
gem "activemodel"
gem "actionpack"
gem "actionview"
gem "actionmailer"
gem "activejob"
gem "activesupport"
gem "railties"
gem "sprockets-rails"
```

In general, list only the gems that you are really using as your dependencies.
Please note that trimming down your `Gemfile` will only have an effect if none of your other dependencies depend on `rails` gem. You can verify that by searching
for `rails` in `Gemfile.lock`.

-------------

If there is anything missing in this guide, please let me know! I want to keep it complete.
