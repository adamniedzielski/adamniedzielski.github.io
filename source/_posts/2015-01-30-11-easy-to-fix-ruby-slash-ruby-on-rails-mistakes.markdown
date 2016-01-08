---
layout: post
title: "10 easy-to-fix Ruby / Ruby on Rails mistakes"
date: 2015-01-31 16:20:11 +0100
comments: true
categories: 
- Ruby
- Rails
- Ruby on Rails
- refactoring
---

Programmers make mistakes. Some of them are just annoying (for others to read) and some are really **dangerous**. Here is my selection of 10 mistakes done by Ruby / Ruby on Rails developers. These tips are easy to follow and can **save you much time** of later debugging.

<!-- more -->

### 1. Double negative and complex conditionals

```ruby
if !user.nil?
  # ...
end

unless user.blank?
  # ...
end

unless user.active? || address.confirmed?
  # ...
end
```

Double negative is hard to read. Every time I encounter it, I spend a couple of seconds on **parsing the condition**. Use the API which Rails gives you - ```user.present?``` instead of ```!user.blank?```.

I also rarely see any usage for ```unless```, especially with complex conditionals connected by ```&&``` and ```||```. How fast can you decide when ```unless user.active? || address.confirmed?``` will fire?

### 2. Using save instead of save! and not checking return value

```ruby
user = User.new
user.name = "John"
user.save
```

What is wrong with this piece of code? It will **fail silently** when user cannot be saved. There will be no single trace of this failure in your logs and you will spend time wondering: "why there are no users in the database". If you expect that data is valid and model should be always saved successfully, then use bang versions - ```save!```, ```create!``` and so on. Use ```save``` only when you handle the return value:

```ruby
if user.save
  # ...
else
  # ...
end
```

### 3. self when it's not needed

```ruby
class User
  attr_accessor :first_name
  attr_accessor :last_name

  def display_name
    "#{self.first_name} #{self.last_name}"
  end
end
```

In this case writing ```self.first_name``` is completely unnecessary, because ```first_name``` will do. This is of course just matter of style and has no other negative consequences than overly verbose code. Please mind that you need ```self``` in assignments: ```self.first_name = "John"```.

### 4. N + 1 queries

This is a vast topic, but I will try to give the simplest example. You want to display a list of posts with names of authors. ```Post``` model ```belongs_to :user```. If you just do ```Post.limit(10)``` and then call ```post.user.name``` in your views, you will make a **separate database query** for each user. That's because Rails has no single chance to guess that you need users when you make the first query in the controller.

It's easy to spot N + 1 queries problem just by looking at server's logs:

```
Template Load (0.4ms)  SELECT  "templates".* FROM "templates"   ORDER BY "templates"."id" desc LIMIT 30 OFFSET 0
  Collection Load (0.2ms)  SELECT  "collections".* FROM "collections"  WHERE "collections"."id" = ? LIMIT 1  [["id", 1]]
  Collection Load (0.1ms)  SELECT  "collections".* FROM "collections"  WHERE "collections"."id" = ? LIMIT 1  [["id", 6]]
  CACHE (0.0ms)  SELECT  "collections".* FROM "collections"  WHERE "collections"."id" = ? LIMIT 1  [["id", 6]]
  CACHE (0.0ms)  SELECT  "collections".* FROM "collections"  WHERE "collections"."id" = ? LIMIT 1  [["id", 6]]
  Collection Load (0.1ms)  SELECT  "collections".* FROM "collections"  WHERE "collections"."id" = ? LIMIT 1  [["id", 3]]
  CACHE (0.0ms)  SELECT  "collections".* FROM "collections"  WHERE "collections"."id" = ? LIMIT 1  [["id", 3]]
  Collection Load (0.1ms)  SELECT  "collections".* FROM "collections"  WHERE "collections"."id" = ? LIMIT 1  [["id", 2]]
  CACHE (0.0ms)  SELECT  "collections".* FROM "collections"  WHERE "collections"."id" = ? LIMIT 1  [["id", 2]]
  CACHE (0.0ms)  SELECT  "collections".* FROM "collections"  WHERE "collections"."id" = ? LIMIT 1  [["id", 2]]
  CACHE (0.0ms)  SELECT  "collections".* FROM "collections"  WHERE "collections"."id" = ? LIMIT 1  [["id", 1]]
  CACHE (0.1ms)  SELECT  "collections".* FROM "collections"  WHERE "collections"."id" = ? LIMIT 1  [["id", 1]]
  Collection Load (0.1ms)  SELECT  "collections".* FROM "collections"  WHERE "collections"."id" = ? LIMIT 1  [["id", 4]]
```

You have to be **explicit** at telling what you need from the database. In the easy cases Rails ```includes``` method will do. You can read more about it in Rails guides - http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations

### 5. Boolean field with three possible values

Boolean is supposed to have two possible values - ```true``` and ```false```, right? And how about ```nil```? If you do not specify default value and ```null: false``` in your migrations, you end up with boolean field with three possible values - ```true```, ```false``` and ```nil```. This leads to nasty code like:

```ruby
# post is new, not published, not rejected
if post.published.nil?
  # ...
end

# post is published
if post.published
  # ...
end

# post is new or rejected
unless post.published
  # ...
end
``` 

If you need three possible states - use string field with three **well-defined** values.

{% render_partial _includes/codebase_health_check_ad.html %}

### 6. Orphaned records after destroy

When you destroy a model and it is required by associated records, you should handle it. It's easy to find such cases:

```ruby
class Post < ActiveRecord::Base
  belongs_to :user
  validates_presence_of :user
end
```

User is required for post. Hence, we have to write:

```ruby
class User < ActiveRecord::Base
  has_many :posts, dependent: :destroy
end
```

### 7. Using code from app/ in migrations

Let's say you have the following model:

```ruby
class User < ActiveRecord::Base
  ACTIVE = "after_registration"
end
```

and you want to add ```points``` field to it. So you create a migration. But you would also like to handle existing users: 10 points for active and 0 for the rest. You add to your migration:

```ruby
User.where(status: User::ACTIVE).update_all(points: 10)
```

It works and you are happy. Time passes by and you decide to remove ```User::ACTIVE``` constant. Your **migrations are now broken**, you cannot run them from scratch, because ```User::ACTIVE``` is undefined.

Never use code from ```app/``` directory in migrations. If you need to update existing data and do it in a few environments (development, staging, production) create a Rake task and delete it once it's executed in every environment.

### 8. Leaving puts

Leaving ```puts``` in the code after some debugging session **pollutes** server logs and output of tests. Use ```Rails.logger.debug``` so it's later possible to adjust the desired log level.

### 9. Not using map

I've seen such code many times:

```ruby
users = []
posts.each do |post|
  users << post.user
end
```

This is exactly the case for using ```map```, which is shorter and more idiomatic:

```ruby
users = posts.map do |post|
  post.user
end
```

### 10. Not using Hash#fetch

```ruby
name = params[:user][:name]
```

What's wrong with this code? It will throw ```NoMethodError: undefined method `[]' for nil:NilClass``` if there is no ```user``` key in the hash. If you expect the key to always be present, use ```Hash#fetch```:

```ruby
name = params.fetch(:user)[:name]
```

This will give you a meaningful exception - ```KeyError: key not found: :user```.


## And you?

This is my selection of Ruby / Ruby on Rails mistakes. And what are your "favorite mistakes"? **Share them in the comments!**
