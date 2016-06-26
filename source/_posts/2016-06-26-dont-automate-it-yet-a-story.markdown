---
layout: post
title: "Don't automate it (yet) - a story"
date: 2016-06-26 09:41:03 +0200
comments: true
categories:
- software engineering
- essay
- product design
- agile
---

Programmers love to approach new business problems thinking about frameworks, libraries, tools or methods which can be used. However, my experience shows that quite the opposite way is most successful - **postpone writing the code** as long as it's possible.

<!-- more -->

### Context of the story

I want to share a story to illustrate this method. My client - Oxbridge Notes is a platform for selling students' notes which focuses on law notes in particular. About a year ago they decided that they want to expand to the law tutoring market and **build a solution for connecting tutors with tutees**.

Of course, there are some strong competitors in the tutoring market, but Oxbridge Notes has the advantage of reaching the majority of law students in the United Kingdom. On the other hand, for a small company it's quite difficult to conduct proper research about profitability of such a venture. They also didn't have a huge budget for advertising the product at all costs - it had to catch on. In this situation it really made sense to **build the product incrementally** and assess the results at the each stage.

### Finding tutors

Obviously you cannot launch a tutoring platform without having some tutors on board. That's why we started by building an application form for tutors (I used this example when describing [form objects](/blog/2016/01/08/contextual-validations-with-form-objects/)). That was all we did at the beginning - **just a single form**.

Then we began searching for tutors by launching advertising campaigns and reaching out to people who sell law notes with Oxbridge Notes. Only after receiving a couple of submissions, we built the admin dashboard for reviewing the candidates.

At this point we decided to **wait** with the further implementation and see whether we will get enough submissions to launch the platform. We wanted to validate our assumptions about the number of people we can reach. It took us about two months to get the solid number of accepted tutors, but we made it.

### Manual matching

When we had enough tutors on board I built the form for tutees searching for a tutor. And of course we needed some fancy matching algorithm to connect tutees with tutors, right? Not at all! In the first phase all the matchings were done **manually** by Joe - our customer support and marketing specialist.

At this point we were still uncertain whether we can find students interested in tutoring. Moreover, we didn't know what percentage of the incoming tutoring requests can be matched to our tutors.

At the beginning the number of people asking for a tutor was small so it was possible to handle this manually. Obviously this is an approach which does not scale, but we were operating on a small scale. It's totally **fine to do things which do not scale in the initial phase** if you have a clear plan how to automate them in the future.

### Automation

When the number of tutoring requests began to rise, matching tutees with tutors started to take up large portion of Joe's time. This was the point when we knew that the product was becoming popular and we had to implement the matching algorithm. As you can see we **postponed** it as long as it was possible.

When I finally started to implement the matching algorithm I didn't have to "invent" how it should operate. Joe prepared a detailed instructions for me, based on his domain expertise and the gathered experience. What's more, I had all the previous manual matchings available so I turned them into test cases. That was awesome! I was able to **test the algorithm using real world examples** and compare the results with the answers suggested by a human.

After implementing the algorithm the platform can operate on its own with little supervision required. By postponing the automation we reduced the risk of a business failure and delivered a better product.


---

All credits for suggesting this incremental approach go to [Jack Kinsella](http://www.jackkinsella.ie/) - founder of Oxbridge Notes.
