---
layout: post
title: "Beautiful Ruby, convenient Rails"
date: 2015-05-26 18:21:27 +0200
comments: true
categories:
- Ruby
- Ruby on Rails
- Rails
- programmer happiness
- beautiful syntax
- Ruby syntax
---

In this blog post I want to reflect on things which make **Ruby** a **beautiful language** to read and write, and on things which make **Ruby on Rails** a **convenient tool** to quickly prototype. 

This is an **introductory** level blog post.

<!-- more -->

## Ruby

{% blockquote Yukihiro "Matz" Matsumoto, the Creator of Ruby %}
The goal of Ruby is to make programmers happy.
{% endblockquote %}

Programming languages are rarely optimized for **programmers' happiness**. Usually they are advertised as "fast", "easy to learn", "low level", "high level", "functional with static typing", but "pleasant to write" isn't on this list.

I want to show you basic Ruby constructs, but focus on these **little details** which make programming in Ruby fun. 

### Data structures

```ruby
str = "this is a string"
array = ["first", "second", "third"]
hash = { "key" => "value", "next_one" => "its value" }
```

What can we see here? First of all - **dynamic typing**. Next, in Ruby there are two basic collections - Array and Hash. This is simple, but allows to model many structures.

### Readable expressions

```ruby
str = "the,quick,brown,fox,jumps,over,the,lazy,dog"
str.split(',').join(' ').gsub("the", "a").upcase
```

Even if you don't know Ruby at all, this code looks quite understandable. First you split on comma, then you join with space, replace all "the" with "a" and finally transform the whole string to upper case. This is possible thanks to **rich standard library** which saves you from reinventing the wheel. 

### Defining methods

```ruby
def generate_password(length)
  ('a'..'z').to_a.shuffle.first(length).join
end
```

This is a method definition, but as you can see there is **no return statement**. Last evaluated expression is the return value of the method. This makes one line methods even shorter.

We also have here the instance of ```Range``` class - ```('a'..'z')``` which is smart enough to iterate over characters.

I want to stress one more detail here - ```first``` method. You may wonder what happens if we take more elements than there are in the array:

```ruby
[1, 2, 3].first(5) # => [1, 2, 3]
```

Nothing blows up and we just get all available elements. Ruby makes a **reasonable assumption** that in most cases we want it to return all elements, not throw an exception.

### Conditional expressions

```ruby
str = "lorem ipsum"
if str.end_with?("ipsum")
  str.capitalize!
end
```

Please notice lack of parenthesis around ```if``` condition. What's more, method names may end with **question mark** or **exclamation mark**. By convention those ending with question mark return boolean value. They look nice in conditional expressions. Methods ending with exclamation mark do something dangerous - for example mutate the object (this example) or throw exception.

```ruby
str.capitalize! if str.end_with?("ipsum")
```

When the block inside conditional expression consists of single line we can make it even more compact.

### Loops

```ruby
small_numbers = []
[78, 32, 44, 1, 7, 23, 56, 98, 45].each do |number|
  small_numbers.push(number) if number < 50
end
```

Ruby has "normal" loops, but they are rarely used. When you have to use a loop, you usually want to **iterate over a collection** and do something with each element. In Ruby you can use ```each``` method and pass it a block of code. This block of code will be invoked for each element of the collection.

### Defining classes

```ruby
class User

  def initialize(name)
    @name = name
  end

  def name
    @name
  end

  def name=(name)
    @name = name
  end
end

john = User.new("john")
john.name
```

This is a definition of a simple class. ```initialize``` method is the constructor, we also have getter and setter for ```name```. Variables prefixed with ```@``` are instance variables. What's worth noting - we don't have to explicitly state all fields of the class, we can just assign to a new instance variable and start using it.

```ruby
class User

  def initialize(name)
    @name = name
  end

  attr_accessor :name
end
```

And this is a shorthand syntax which does exactly the same. We replace 7 lines of code with one ```attr_accessor``` macro.

### Everything is an object

```ruby
user.class # => User
user.class.class # => Class
user.class.class.class # => Class
```

In Ruby everything is an object. Even integers are objects of class ```Fixnum```. What's more interesting - every class is an object of class ```Class``` and ```Class``` class is also an object of class ```Class```.

### Functional programming

```ruby
[78, 32, 44, 1, 7, 23, 56, 98, 45].map do |number|
  number * 2
end
# => [156, 64, 88, 2, 14, 46, 112, 196, 90]

["John", "Kate", "George", "Victoria"].map do |name|
  name[0].downcase
end
# => ["j", "k", "g", "v"]
```

Method ```map``` is quite similar to ```each```. It invokes block of code for each element of the collection, but the result of the block (last evaluated expression) is pushed into the new array and this array is returned.

```ruby
[78, 32, 44, 1, 7, 23, 56, 98, 45].select do |number|
  number < 50
end
# => [32, 44, 1, 7, 23, 45]
```

This one is very readable - we take only these elements which match condition in the block.

### Switch statement

```ruby
case value
when "test"
  puts "test string"
when 1..10
  puts "between 1 and 10"
when /\Afoo/
  puts "starts with foo"
when String
  puts "another string"
else
  puts "something else"
end
```

Switch statement in Ruby is powerful. We can match by value, by range membership, by regular expression, by class, ... And there are no awful ```break```s here.

## Ruby on Rails

Ruby as a programming language is beautiful, but Ruby on Rails as a web framework emphasizes convenience. The code below may seem a little bit "hacky", but it's convenient.

### present? or blank?

```ruby
nil.present? # => false
"".present? # => false
[].blank? # => true
```

### Date, time

```ruby
10.minutes.ago # => Tue, 26 May 2015 17:39:55 BST +01:00
3.days.from_now.beginning_of_day + 2.hours # => Fri, 29 May 2015 02:00:00 BST +01:00
```

I cannot imagine more readable syntax to work with date and time.

### ... and timezones

```ruby
Time.zone.name # => "London"
5.hours.ago # => Tue, 26 May 2015 13:06:00 BST +01:00
5.hours.ago.in_time_zone("Warsaw") # => Tue, 26 May 2015 14:06:19 CEST +02:00
```

No more programmer's nightmares about timezones!

### try

```ruby
["John", "Kate", nil, "Robert"].map do |name|
  name.downcase
end
```

This code blows up with ```NoMethodError: undefined method `downcase' for nil:NilClass```. This is a common situation: we expect a string, but are given a nil value. Rails has a quick fix for it:

```ruby
["John", "Kate", nil, "Robert"].map do |name|
  name.try(:downcase)
end
# => ["john", "kate", nil, "robert"]
```

Calling ```try``` invokes a method if it's defined for given object or returns nil.

### File size

```ruby
2.kilobytes # => 2048
3.megabytes # => => 3145728
```

This converts to bytes - I have yet to use it in practice :)

### Displaying numbers

```ruby
4567.to_s(:human) # => "4.57 Thousand"
1500000.to_s(:human) # => "1.5 Million"
```

### Ordinal numbers

```ruby
21.ordinalize # => "21st"
45.ordinalize # => "45th"
```

This probably could be easily implemented by you, but why you would want to do it if it's already done and tested?

### Grouping

```ruby
[1, 2, 3, 4, 5, 6, 7, 8].in_groups_of(3) # => [[1, 2, 3], [4, 5, 6], [7, 8, nil]]
[1, 2, 3, 4, 5, 6, 7, 8].in_groups(2) # => [[1, 2, 3, 4], [5, 6, 7, 8]]
```

This may be useful when dividing array into columns or rows.

### Plural form

```ruby
"apple".pluralize # => "apples"
"fish".pluralize # => "fish"
"foot".pluralize # => "foots" Whoops!
```

## Your ideas

Do you have your favorite examples of beautiful Ruby syntax? Or convenient Rails methods? Share them in the comments!
