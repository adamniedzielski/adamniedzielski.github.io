---
layout: post
title: "Read before you commit"
date: 2015-12-11 07:55:11 +0100
comments: true
categories:
- git
- workflow
- high quality code
- software engineering
- software craftsmanship
---

This is going to be a short piece on one particular **habit** which I practise when writing code. It's extremely **simple** and obvious, but I'm surprised how many programmers skip it.

<!-- more -->

Let's start with the advice:

{% blockquote %}
Read the code before you commit it. Every file. Every single line. Just go through the changes which you have done and read them all. You can even do it twice.
{% endblockquote %}

And here goes the explanation:

This won't be a lost time. Developers often say that they "**forgot** to remove these debug logs" or "**forgot** to delete this binding.pry". This just won't happen if you read the code. You will never "**accidentally** commit .DS_Store file" when you just review the list of files which you are committing.

You can use your favourite tool to look at the staged changes. Mine is [Git Gui](https://git-scm.com/docs/git-gui), but it's not really about the tool. It's about the **habit** of reviewing the changes before you commit them.

Here are some questions which you should ask yourself when going through all these lines of code:

1. **Is it really necessary?**

    What happens if you revert the changes in this line? Are they **required** for the rest of the code to work? Maybe it was you first solution, but then you had a better idea, introduced some other change and this one is now unnecessary?

2. **Do you understand why it works?**

    If you know that a piece of code works, but you don't understand **why**, then you should learn it. You should learn it before committing the code. The chances are that you missed some edge cases if you don't understand the general principles.

3. **Is it related to the feature?**

    You may have decided that it's generally a good idea to change the log level in "development" environment to "debug". However, it shouldn't be included with the other changes related to the current feature / bug fix. Why? Because the person reading your code may assume that it's somehow related and necessary for other changes to work. What's more, the changes are now tied in the single commit, so you cannot decide to merge one and leave the other. Just **make a separate commit for every unrelated change**.

4. **Is it inline with the best practices?**

    Have you followed code style conventions for your codebase? Maybe your changes include some repetition which can be avoided by extracting a method? How about [moving part of that 15 lines controller action to a service object](/blog/2014/11/25/my-take-on-services-in-rails/)?

{% blockquote %}
Doesn't it take up too much time? I'm in a hurry, we have deadlines, you know.
{% endblockquote %}

No. Nope. **It doesn't take up too much time.** This time is a time well spent. You will soon be even more in a hurry if you commit some changes without reviewing them. If you make any mistake, the combined effort to find it, communicate it, fix it, commit it and deploy it, will be always higher than the effort to just read the code.

**Read the code before you commit it.**
