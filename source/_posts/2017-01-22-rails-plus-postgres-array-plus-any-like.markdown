---
layout: post
title: "Rails + Postgres Array + ANY LIKE"
date: 2017-01-22 17:09:11 +0100
comments: true
categories:
- Rails
- Ruby on Rails
- Ruby
- Postgres
- PostgreSQL
---

Ruby on Rails has a [good support](http://guides.rubyonrails.org/active_record_postgresql.html#array) for Postgres Array type. I really like using this feature when creating a separate database table sounds like over-engineering. In this short post I want to share my solution for the following problem: "**find a record for which any of its tags contains a given string**".

<!-- more -->

Let's use `Book` model with `categories` array field as an example. We start by generating a migration file:

```
rails g model Book name:text categories:text
```

and then we have to change it and specify `array: true, default: []`.

```ruby
class CreateBooks < ActiveRecord::Migration[5.0]
  def change
    create_table :books do |t|
      t.text :name
      t.text :categories, array: true, default: []

      t.timestamps
    end
  end
end
```

As a next step we create a few records for testing:

```ruby
Book.create!(name: 'Traitors Of Time', categories: ['science fiction', 'romance', 'drama'])
Book.create!(name: 'Dead At The Beginning', categories: ['romance', 'horror'])
Book.create!(name: 'Gangsters And Kings', categories: ['action and adventure', 'mystery', 'drama'])
```

If we want to find records by the exact name of the category it is very simple. Postgres has it covered with `ANY` function:

```ruby
Book.where(":name = ANY(categories)", name: "drama")
```

How about making our search a little bit more robust and supporting pattern matching? In case of a regular field we would do that using `ILIKE`:

```ruby
Book.where("name ILIKE :name", name: "%ing%")
```

Unfortunately this approach does not work with `ANY`:

```ruby
Book.where(":name ILIKE ANY(categories)", name: "%action%")
```

This query returns zero results, because the pattern containing `%` has to be on the right hand side of `ILIKE`.

```ruby
Book.where("ANY(categories) ILIKE :name", name: "%action%")
```

And this is an unsupported syntax producing a syntax error.

### Solution

We can work around this problem by using `array_to_string` function:

```
Book.where("array_to_string(categories, '||') ILIKE :name", name: "%action%")
```

`array_to_string` concatenates all the categories, using `||` as a separator between them and then we can use `ILIKE` in a usual way. The assumption here is that the pattern will never contain `||`.

This approach may not be the best idea if the number of rows is big. However, for a table with a couple hundreds rows it was the simplest solution I could come up with.
