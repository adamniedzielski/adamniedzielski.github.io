---
layout: post
title: "wroclove.rb 2014"
date: 2014-03-21 18:00:00 +0100
comments: true
categories: 
- Ruby
- Ruby on Rails
- conference
- Java
---

Here you can find a couple of my personal opinions about talks at [wroclove.rb 2014](http://wrocloverb.com/) which took place in March 14th - 16th in Wrocław. I tried to keep my feedback 100% positive in order not to upset anybody, but if you are - write a comment and we'll discuss.

<!-- more -->

## Day 0 - Soft skills 

### Developer Oriented Project Management
Robert Pankowecki - [@pankowecki](https://twitter.com/pankowecki)  
slides: http://pankowecki.pl/wrocloverb2014/index.html#/

I had heard this presentation before - during [LRUG](http://lrug.pl) in February and I can say that it was improved a bit since that time. Robert Pankowecki presented a complete methodology, which is similar to **religion** - you have to believe it or not. The three fundamental rules are:

1. Story Of Size One
2. Leave Tasks Unassigned
3. Take The First Task

While I agree with rules 1 & 2 I am reluctant to accept rule 3. I can imagine a developer who wants to stay **frontend-only**. This doesn't mean that he doesn't want to learn new things - in fact, when the choice of "the most rockstarish JavaScript framework" is changing every month there is **pretty much to learn there**. Of course, the same thing applies to backend development.

Nobody blames cardiologist who is not willing to develop as psychiatrist - if you know what I mean. **Division into frontend and backend** sounds fine to me.

### 15 Minutes Rails Application
Marcin Stecki - [@madsheepPL](https://twitter.com/madsheepPL)  
slides: http://talks.madsheep.pl/talks/rapid#/

The title was kind of a trick to catch the attention, in fact the presented application was built in 75 minutes, but this was still too fast to build a **maintainable solution**. And in my opinion this was the key point of Marcin Stecki - you cannot build anything very **fast** without paying the **price**. I liked the talk for its pace, light style and jokes.

### Please Stand Up!
Piotr Misiurek - [@PiotrMisiurek](https://twitter.com/PiotrMisiurek)  
slides: http://bit.ly/1eVoSha

This talk was controversial and I would remove a few sentences from it (the version which I heard at LRUG in February was perfectly fine), but in the face of criticism I want to say that it had many **good parts**. I liked the talk for the courage of the speaker. People often talk about soft skills in a very general way, using impersonal narration. Piotr Misiurek described his **own story**, gave examples from his life, so we could see that the solutions presented really work.

### The Story of Bringing Ideas to Life
Jeppe Liisberg - [@jeppeliisberg](https://twitter.com/jeppeliisberg)  
slides: http://bit.ly/OAoUVB

The talk was great when we consider art of giving presentations. It was filled with a bit of irony, sarcasm or maybe just looking from a distance. I liked the chart which showed that **nothing is so simple**:

{% img http://pbs.twimg.com/media/BiuAklaIAAALNxG.jpg:large %}

Big plus for the courage to say **"I failed, but look how much I learned from my failure"**. I think that this great approach is quite uncommon in Poland - we rarely talk about our failures.

## Day 1

### Application Architecture: Boundaries, Object Roles & Patterns
Adam Hawkins - [@adman65](https://twitter.com/adman65)  
slides: https://speakerdeck.com/ahawkins/rethinking-application-architecture

To me the main point of the presentation was: **keep your business logic separate from the framework**. Adam Hawkins told us that we should **not trust Rails** and we should be careful when we use gems, because this may force us into maintaining the code which we didn't write.
He described form objects, repository pattern and extraction of use cases to separate classes.

One of the questions asked after the talk was: "Will you use this approach for all projects?". And the answer was: "No, only for the big ones". I think that this should have been included in the talk itself. What seems perfectly reasonable for **long-running** projects with **complex** business logic may not be appropriate while developing **simple** apps which do not differ much from **CRUD**.
Steve Klabnik wrote a [great post](http://words.steveklabnik.com/rails-has-two-default-stacks) in which he stressed that we should remember about beginners and try not to **confuse** them.

### Can We Write Perfect Tests? - Maybe!
Markus Schirp - [@\_m_b_j_](https://twitter.com/_m_b_j_)  
slides: http://slid.es/markusschirp/mutation-testing-fight-2

Markus Schirp was selling [Mutant gem](https://github.com/mbj/mutant) and I bought it. Of course, it has to be used with a bit of consideration, because **100% mutation coverage** is extremely hard to achieve and rarely necessary. However, when you test a couple of **core classes** which contain for instance finance logic then they really have to be covered in 100%.

### Integration Tests Are Bogus
Piotr Szotkowski - [@chastell](https://twitter.com/chastell)  
slides: http://talks.chastell.net/wrocloverb-2014/#/

Piotr Szotkowski presented many valuable ideas about **testing**. He described the purpose of unit tests, integration tests and end-to-end tests. There was a lot of great code examples in the slides, all sharing the same humorous theme. [Bogus](https://github.com/psyho/bogus) was used to avoid **mocking** and **stubbing** methods that do not exists. Piotr also introduced **contract tests** which I will have to check out soon. In general, the whole presentation was packed with knowledge and certainly I'll get back to it.

### Q&A Session - Metrics
Piotr Solnica - [@\_solnic_](https://twitter.com/_solnic_) & Markus Schirp - [@\_m_b_j_](https://twitter.com/_m_b_j_)

I remember three main conclusions:

- there is no silver bullet, do not apply **code metrics** blindly
- **colors** are better than **numbers**, especially when you want to show code metrics to your clients
- code metrics should not fail your build

### Migrating to Clojure. So much fn.
Jan Stępień - [@janstepien](https://twitter.com/janstepien)  
slides: https://speakerdeck.com/jan/wroc-love-dot-rb-migrating-to-clojure-so-much-fn

I was really impressed by the extremely **professional** delivery, full control over the voice, posture, audience, pace and questions. Jan Stępień appeared to have a great knowledge about Clojure (I had such impression but I'm not able to verify it). Learning a **functional programming language** is on my personal development todo list.

### From ActiveRecord to Events
Emanuele Delbono - [@emadb](https://twitter.com/emadb)  
slides: http://www.slideshare.net/emadb/wroclove-rb

Emanuele Delbono showed us the complete solution for the architecture which he contrasted with the regular **Rails Lasagna architecture**. Then he presented the pros and the cons of such an architecture. I would have to think carefully before applying this in a project - I have the impression that some changes are painful in this architecture, especially event changes (and for sure event changes will happen).

{% render_partial _includes/codebase_health_check_ad.html %}

## Day 2

### Ruby Arrays on Steroids
Michael Feathers - [@mfeathers](https://twitter.com/mfeathers)

This was more like an **inspiration** than technical talk. Michael Feathers presented APL and J languages to broaden our horizons. He explained the advantages of concise code in the context of building better and better **abstractions**. He told the history of **paradigm shift** from procedural to object oriented languages and as far as I understood - he meant that we should be ready for the next such shift. Michael Feathers also said that he values full stack developers in the boadest possible sense - someone who knows his stuff from assembler to business domain.

### Micro Libraries FTW
Piotr Solnica - [@\_solnic_](https://twitter.com/_solnic_)  
slides: https://speakerdeck.com/solnic/micro-libraries-ftw

Piotr Solnica advocated small libraries which have **single responsibility**, are stable and easy to investigate on your own. Such library has low maintenance costs, because it can be easily replaced by your own solutions if you find out that it causes more troubles than it gives benefits. What's more, Piotr encouraged to follow **UNIX philiosophy** - chain the gems and build from small bricks. I liked the idea that frameworks should be decomposed into microlibraries and serve a role of a **convenient facade**. If the facade becomes a burden for you - stop using it and use those microlibraries directly.

### Objectify Your Forms: Beyond Basic User Input
Danny Olson  
slides: http://www.slideshare.net/dannyolson315/objectify-your-forms

Danny Olson provided us with an in-depth analysis of **form object pattern**. It was a great technical talk, full of code examples sharing one real-word theme - wonderful startup idea of Online Meme-Based Ice Cream Ordering Service. I liked his rule of thumb:
{% blockquote %}
Use form object when you persist more than one object.
{% endblockquote %}

### Q&A Session - Legacy Rails
Michael Feathers - [@mfeathers](https://twitter.com/mfeathers), Adam Hawkins - [@adman65](https://twitter.com/adman65),  
Andrzej Krzywda - [@andrzejkrzywda](https://twitter.com/andrzejkrzywda)

Starting from scratch is easy, working with **legacy code** - not. This was a great discussion how to deal with legacy applications when you don't have tests to support you and you have little understanding of the code.

{% blockquote Andrzej Krzywda %}
Working with legacy code is kind of success measure - for startups - it means that business works
{% endblockquote %}

{% blockquote Michael Feathers %}
People want to rewrite the application because they don't understand the code and they cannot rewrite the application because they don't understand the code.
{% endblockquote %}


### Ruby: Write Once, Run Anywhere
Michał Taszycki - [@mehowte](https://twitter.com/mehowte)  
slides: https://speakerdeck.com/mehowte/ruby-write-once-run-anywhere

I missed this talk - write you description in the comment and I'll put it here giving you credit.

### Frontend choices
Alex Coles - [@myabc](https://twitter.com/myabc)  
slides: https://speakerdeck.com/myabc/frontend-choices

I missed this talk - write you description in the comment and I'll put it here giving you credit.


## Lightning talks (selection)

I was present only during the first round of lighting talks (Day 1). I was not able to remember all of them (they were so lightning :) ), but here you can find three of them:

#### Aleksander Dąbrowski - [@_tjeden](https://twitter.com/_tjeden)

This lightning talk was a reaction to general **Rails critism** during the conference. Aleksander Dąbrowski was defending Rails saying that majority of Rails developers are working on projects for which **Rails is just fine**. I think that this is a sane point of view - unnecessary complexity is unnecessary. I'm not saying that Rails does not have issues - it does, but the truth lies somewhere in the middle and the tools have to be chosen for a specific job.

#### Marta Paciorkowska - [@a_meba](https://twitter.com/a_meba)

Marta Paciorkowska presented [Chef Browser](https://github.com/3ofcoins/chef-browser) web application. Default **Chef** server interface isn't rich at all, but Chef server exposes API for manipulating it - thank you for creating this helpful tool.

#### Arne Brasseur - [@plexus](https://twitter.com/plexus)

I loved the idea behind his lightning talk. **Play your code** during lightning talk for so much win - you can speed up when you have to, people will laugh and you can show real code execution.  
I will have to give it a try!

<br>
In general, I really liked the idea of audience clapping right after speaker reached the time limit of five minutes. The speaker could only smile, say "thank you" and let the next person speak. That's 100% positive interruption.

## General thoughts

Many speakers at wroclove.rb expressed their **concerns about Rails**, which appears to be a **hot topic** in Ruby community in the recent months. Some of the attendees responded with **"we don't want your Java in our Ruby"** attitude. I didn't talk with enough number of people about their thoughts after the presentations, so I don't know what the "average reception" was. 

What I want to point out is: people who give talks at conferences are naturally **somewhere ahead**, they speak because they have some experience and they met with hard problems. As a result - they present some **complex solutions**. These complex solutions work for them, because their problems require these complex solutions.

On the other hand, people who attend the conference may have less experience, they may work with applications where business logic is **much simplier** and hence they are happy with what Rails gives them.

Going back to **Java** topic - this seems to be a **symbol of unnecessary complexity** for Rubists. However, the design patterns aren't specific to Java at all. I think it's not possible to avoid these good object-oriented principles just because you write code in different language and framework. Ruby on Rails is great for small applications, because you don't have to introduce unnecessary complexity when you don't need it. And I believe Ruby on Rails **can be great for big applications**, but then you need this complexity, because your application is ehm... big and complex.

As far as I remember, Michael Feathers said something like: 

{% blockquote %}
Every couple of years we reinvent the same principles and patterns. But there is one thing we can do - we can implement these patterns in a better, more flexible way.
{% endblockquote %}

## Thanks

I want to thank the **organizers** for the amazing conference. It was really, really well-organised event and I'm impressed how smooth everything went.

I want to thank all the **speakers** for their fascinating talks. You gave me a bunch of new ideas to consider.

I want to thank **my employers @goodylabs** for paying for my conference ticket and whole stay in Wrocław.

