---
layout: post
title: "Black-box testing"
date: 2014-05-19 20:53:52 +0200
comments: true
categories:
- Ruby
- Ruby on Rails
- testing
- black-box
- white-box
- Capybara
- calabash-android
- Android testing
---

This time I want to share my thoughts concerning two approaches to testing - **black-box** testing and **white-box** testing. I gave a talk about it at [Lodz Ruby User Group](http://lrug.pl/). You can find the slides [here](http://adamniedzielski.github.io/presentations/black-box-testing/).

The presentation took place some time before [DHH's keynote at Railsconf 2014](http://www.confreaks.com/videos/3315-railsconf-keynote-writing-software) and before [#isTDDDead discussion](https://twitter.com/search?q=%23isTDDDead) began. However, there is a connection: DHH encourages to write ["higher level system tests"](http://david.heinemeierhansson.com/2014/tdd-is-dead-long-live-testing.html) and in this post I express slightly more balanced opinion on black-box testing.

<!-- more -->

### White-box

When trying to describe black-box testing it's easier for me to start with white-box testing and then **contrast** it with black-box testing. That's because I believe that **majority of Rails developers** is more familiar with white-box testing. 

White-box testing is often called **glass-box testing**, because we imagine that our application under test is put inside a glass box. The application has its well-defined borders, but we can still look through the borders and see individual components or sub-systems. However, we don't look inside the box just for fun, we use the knowledge about **application internals** to write better tests.

### Black-box

{% img center http://adamniedzielski.github.io/presentations/black-box-testing/blackbox.jpg 300 %}

What's that? This is our app in a black box.

When it comes to testing we don't care what is inside. It can be RoR, Sinatra, Node.js, Java EE or anything you want. We don't care, because we want to test our application from the **user's perspective**. 

We are testing our application from the user's perspective, so we have to use the same interface to communicate with it as a user does. In case of web application a **browser** is this interface. 

The whole idea of black-box testing seems to be extremely appealing at first. Our customers don't care whether all unit tests are passing. Probably they don't even know what unit tests are. Our **customers care about the business value** of the application. They want our application to be usable from the user's point of view. 
If we test all business scenarios from the user's point of view and the tests are passing we can truly say that our application has business value.

### Capybara

Capybara is a powerful **gem** which provides a layer of abstraction over browser drivers and exposes simple DSL which lets you interact with the application similarly to a user using a **browser**. As you can see the DSL is intuitive and readable:

```ruby
visit('/users/sign_in')

fill_in 'Login', :with => 'user@example.com'
fill_in 'Password', :with => 'password'

click_link 'Sign in'

expect(page).to have_content('Success!')

```

In this post I'm not going to describe how to use Capybara, because it is already covered in depth by others. What's more, [Capybara README](https://github.com/jnicklas/capybara/blob/master/README.md) is a wonderful guide so I just encourage you to read it.

### Who is Real User?

{% blockquote Capybara README %}
Capybara helps you test web applications by simulating how a real user would interact with your app. [...]
{% endblockquote %}

The term **real user** is important here. What does "real user" actually mean? Let's examine this example of a spec:

```ruby

describe "Edit post page" do

  context "when user is the creator of the post" do
    
    before do
      user = FactoryGirl.create(:user)
      login_as(user, :scope => :user)

      @post = FactoryGirl.create(:post, :user => user)

      visit edit_post_path(@post)
    end

    it "shows previous content of the post" do
      expect(page).to have_content(@post.content)
    end
  end
end

```

We are testing that a previous content of a post appears on the edit post page - very simple scenario. In order to test it we have to have logged in user and an existing post - so we prepare it.

But, wait! Do you have a user of your application who:

* has **direct access to the database** so he can create his user account without using the registration form and he can create a new post without using the proper form?
* can **bypass the authentication** logic of your app?

I don't have such a user and I hope you neither (this would be quite scary). So, this scenario is **not** really testing from the **real user's** perspective. This is only some approximation of the real user's perspective and we can make it a better approximation:

```ruby
describe "Edit post page" do

  context "when user is the creator of the post" do
    
    before do
      # register using sign up form
      # this should also make this user signed in

      # navigate to new post page
      # fill in the form and hit "Create!" button

      visit edit_post_path(Post.last)
    end

    it "shows previous content of the post" do
      expect(page).to have_content(Post.last.content)
    end
  end
end

```

Does this spec provide better value to your test suite than the previous one? - read on and you will see.


### Black-box testing Android apps

For majority of people Ruby == Rails, but there are also other fields where you can use Ruby. I was programming in Ruby for half a year without even touching Rails. I was working as a Test Automation Engineer and writing **automated tests for Android apps**. This gave me some experience in testing and I still find these lessons relevant when testing RoR apps.

I was using a gem called [calabash-android](https://github.com/calabash/calabash-android). This gem is responsible for installing the application on real device or emulator, communicating with it and executing some **basic UI actions** like click, scroll or insert text. Tests are described with [Cucumber](https://github.com/cucumber/cucumber) and particular test steps are implemented using DSL exposed by calabash-android. If you are interested you can [read more about the architecture of calabash-android](http://blog.lesspainful.com/2012/03/07/Calabash-Android/), but this gist shows how similar capabilities of Capybara and calabash are:

```ruby

# see https://github.com/calabash/calabash-android

query("edittext index:1", :setText => "test@example.com")

wait_for(:timeout => 5) { element_exists("button marked:'Save'") }
touch("* marked:'Save'")

check_element_exists("view marked:'confirmation'")

```

Testing with calabash-android is **purely black-box**. You interact with the application in the same way as a user does. There is no easy way to play with the internal state of the application or mock external services.

In general, testing Android applications is not an easy thing and I consider **Android** platform to be much **less testable** than Ruby on Rails framework (at least this was the state of things one year ago). Everything is coupled with the platform and even the unit tests have to be run on the device or emulator (however there are some [attempts to overcome this](http://robolectric.org/)). That's why our black-box tests with calabash-android were **the only automated tests in the project** I was working on.

### The results

I was doing my best, but the **value** of the automated test suite which I created was rather **low**. I will describe 4 reasons why it didn't work properly as a safety net for developers:

* **Slow** - The whole test suite was taking **over two hours** to execute. Developers didn't have immediate feedback after committing their changes, so they didn't pay much attention to the results of the tests.

* **Fragile** - Black-box tests depend heavily on the **UI** and they fail after UI changes. Each "false alarm" causes a loss in confidence.

* **Depedent on API responses** - The application we were building was using data from **external API**. The data we were receiving was dynamic and varied in time. This resulted in the horrifying cases when some scenario could be tested only during a specific time of the day. 

  We tried to live with that and built some kind of "intelligent tests" - they changed the test scenario based on the received data. This inverted the regular flow of testing from "prepare environment, execute, check" to "test everything which can be tested in a given environment". Of course, this was nowhere near confidence.

* **Strong coupling** - Every business scenario consists of a series of steps. In case of our Android application it could be shown as:

  ```screen 1 -> screen 2 -> screen 3 -> screen 4 -> screen 5 -> screen 6 -> screen 7```

  What if we want to test screen 7? We have to go through all the screens which are earlier in the flow. What if a test fails on screen 3? We know that screen 1 works and we know that screen 2 works. Moreover, we know that screen 3 doesn't work. But how about screens 4, 5, 6 and 7? We know nothing about them at this point.


All these factors combined resulted in a **test suite which didn't provide any confidence**. We still had to do manual testing and we did smoke testing before each weekly release to the customer. That happened because we were **not trusting our tests**.

### Lessons learned

The results were miserable, but I learned a lot from this experience. 

The first lesson is: **black-box AND white-box**. Black-box tests have to be combined with white-box ones. It's almost impossible to test all business logic in black-box tests, because you don't have full control over the environment. Setting up proper environment for test execution can be very time consuming and will result in failures in this introductory stage. Sometimes setting up environment is **just impossible** - for example when you depend on an external API as we did. 

When doing white-box testing you should **mock any external service**. Testing business logic using "production" of external service will make your tests fail randomly. Moreover, it's not possible to check all corner cases when you don't control the responses.

For high quality description on when to mock please read [Uncle Bob's point of view](http://blog.8thlight.com/uncle-bob/2014/05/10/WhenToMock.html).


### When black-box testing adds value

So, what's the purpose of this black-box testing? Maybe we should **just avoid it**, maybe it's just an idea which sounds great when described, but fails when applied in real life? 

I don't think so! Black-box tests are valuable, but only when used correctly. They help you avoid a situation when all the white-box tests are **passing**, but the **application isn't booting up**. 

When you mock external services in white-box tests you have to make sure that their real interfaces are the same. This doesn't mean repeating the same tests for all business logic. This means using the external services in the simplest possible scenario and checking that **your code is not throwing errors at you**.

It's also worth to write black-box tests for a **key business scenario** in your app. If you are running an online store - make sure that a user can add a product to the cart and pay for it. When you want to make a deployment to production and this scenario is passing you have some confidence - "at least my key business feature is usable".


### How to black-box

When writing black-box tests focus on happy path. Or even **very, very happy path**. It is pointless to test complex business logic rules in black-box tests. It should be already tested thoroughly on the lower level of white-box tests.

Imagine a **bored Quality Assurance Engineer**. Your application is standing between him and his coffee break. He wants to see all the most important screens of your app as quickly as possible. He will always choose a button which takes him forward and will never try to diverge from the happy path. In every form he will enter random data just to reach next screen quickly. Your black-box tests should be like this QA Engineer.

### Summary

1. Black-box tests don't make sense without white-box tests.
2. In black-box tests you should test only the happy path - testing business logic is pointless there.
