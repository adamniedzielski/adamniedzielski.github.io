---
layout: post
title: "Different approach to Ecommerce on Rails"
date: 2016-03-29 07:46:07 +0200
comments: true
categories:
- Ruby
- Rails
- Ruby on Rails
- ecommerce
- Spree
- events
- open-source
---

I always wanted to build my own **ecommerce platform from scratch**. That's not because I don't like to reuse existing code - I really love to. That's because I believe that we can do better. Now I have a chance to do it and you can **follow my work on Github**.

<!-- more -->

**I'm not a big fan of [Spree](https://github.com/spree/spree)**. I do agree that it may be the best cost effective solution for some projects. However, the problem space of ecommerce is huge. It's difficult to create a library (or framework) which suits everyone's needs. Some people sell physical goods and some people sell digital content. Some people want to have immensely complex discount rules and some people are fine with simple percentage discounts. Some people sell products in different variants and some people offer fully customizable products. You can see where it's going.

At https://www.oxbridgenotes.co.uk I helped in the **migration from Spree to our custom solution**. We didn't use many of the features which Spree provides (mainly because we sell digital content) and we had a lot of our own code anyway (mainly because we create products dynamically).

There are still some concepts from Spree left in the codebase and patterns which are difficult to get rid of. The one thing which I find really painful is **debugging failures in the checkout flow**. Imagine a following story:

1. anonymous visitor adds a product to the cart
2. adds a second product to the cart
3. logs in
4. clicks "Pay" and is redirected to PayPal
5. cancels and goes back to the store
6. adds a third product to the cart
7. clicks "Pay" and is redirected to PayPal
8. payment fails and he is redirected back to the store
9. he removes the second and the third product from the cart
10. clicks "Pay" and is redirected to PayPal
11. finally pays for the order

What we have stored in the database after the last step is a paid order with a single line item. Oh, and of course we have a bunch of logs in our log aggregation service. **It's quite tricky to figure out the exact flow after the fact.**

That's why I want to build an ecommerce platform which is based on events. Instead of storing only the current state (order and line items) I will **store all the events** - for instance ```ProductAddedToCart```, ```ProductRemovedFromCart```, ```PaymentFailed```, ```OrderPaid``` and so on. The events are **immutable** - they never change after being saved to the database. This allows me to inspect the state of any checkout process in any given point in time. You can read more about this approach [here](http://blog.arkency.com/2015/03/why-use-event-sourcing/).

Why am I writing about all this before building the application? Because [you can follow my work on Github](https://github.com/jackkinsella/confessions). This is a real commercial project, but my client [Jack Kinsella](http://www.jackkinsella.ie/) decided to **open source** it. Jack is writing books about launching digital products and this is a platform for selling these books.

I encourage you to leave your feedback. I am still experimenting with these concepts, so I may not get everything right from the beginning, but I believe that reading real production code is more interesting than theoretical explanations.

### [Go to the repository](https://github.com/jackkinsella/confessions)
