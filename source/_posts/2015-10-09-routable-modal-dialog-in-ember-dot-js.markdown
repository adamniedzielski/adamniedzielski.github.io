---
layout: post
title: "Routable modal dialog in Ember.js"
date: 2015-10-09 12:18:53 +0200
comments: true
categories:
- javascript
- Ember.js
- UX
- frontend
---

I **hate** using **modal dialog** when the new content could be easily nested in the same page. I just don't like the user experience of a popup. Why limit yourself to a small box when you can show it in the full-blown layout?

However, sometimes clients insist on a modal in their design and we have to deal with it. In this blog post I will show you how to make the modal dialog a little bit better - **give it its own URL**.

<!-- more -->

Lack of own URL is the pain point of many modal dialog implementations. Imagine that you have a list of articles and when a user clicks "edit" the form is rendered in the modal dialog, covering the list. One day the form stops working and the user submits a bug report with the URL of the page. What is the URL in the bug report? Of course, it's the URL of the list, because showing the modal does not change the current URL. Happy debugging!

In fact, the situation doesn't have to be related to bugs at all. It's also true for any other case when user would like to **share a link to a specific page**. Take the next example: **reloading the page**. User reloads the page and the modal dialog is gone.

When I learned that routable modal dialog can be **easily implemented in Ember.js** I was very happy, because the above issues are important for me. I've never implemented similar solution while working with modal dialogs in AngularJS, but in fact I never really searched for it. If you have an example please share it in the comments!

The workhorse of this example is going to be [Ember Modal Dialog](https://github.com/yapplabs/ember-modal-dialog). This is a **wonderful addon** which in fact does the job for us. In this example I'm going to use Ember CLI 1.13.8, Ember 2.1.0 and Ember Modal Dialog 0.8.1.

Let's start with creating the new application:

```
ember new modal-dialog-demo
```

and change version of Ember to 2.1.0. We can also remove Ember Data, because we will mock data with static objects and [ember-data-fixture-adapter](https://github.com/emberjs/ember-data-fixture-adapter/) is no longer maintained.

As the next step, install ```ember-lodash```. Then add code which renders a simple list of posts, with nested ```show``` route:

```javascript app/router.js
import Ember from 'ember';
import config from './config/environment';

var Router = Ember.Router.extend({
  location: config.locationType
});

Router.map(function() {
  this.route('posts', { path: '/' }, function() {
    this.route('show', { path: '/posts/:post_id' });
  });
});

export default Router;
```

```javascript app/routes/posts.js
import Ember from 'ember';

export default Ember.Route.extend({
  postRepository: Ember.inject.service(),

  model: function() {
    return this.get('postRepository').findAll();
  }
});
```

```javascript app/routes/posts/show.js
import Ember from 'ember';

export default Ember.Route.extend({
  postRepository: Ember.inject.service(),

  model: function(params) {
    return this.get('postRepository').findById(params.post_id);
  }
});
```

{% raw %}
```html app/templates/posts.hbs
<ul>
  {{#each model as |post|}}
    <li>
      {{post.title}}
      {{#link-to 'posts.show' post}}
        show
      {{/link-to}}
    </li>
  {{/each}}
</ul>

{{outlet}}
```

```html app/templates/posts/show.hbs
{{model.title}}
```
{% endraw %}


We also have to implement this ```PostRepository``` service. Here it is:

```javascript app/services/post-repository.js
import Ember from 'ember';
import _ from 'lodash/lodash';

export default Ember.Service.extend({
  FIXTURES: [
    {
      id: 1,
      title: "Some random title"
    },
    {
      id: 2,
      title: "Another silly blog post"
    },
    {
      id: 3,
      title: "Why should I care?"
    },
    {
      id: 4,
      title: "And yet another one to fill the space"
    }
  ],

  findAll: function() {
    return this.get('FIXTURES');
  },

  findById: function(id) {
    return _.findWhere(this.get('FIXTURES'), { id: parseInt(id) });
  }
});
```

{% render_partial _includes/codebase_health_check_ad.html %}

Now it's time to replace the nested post view with a modal dialog.

Start by installing ```ember-modal-dialog``` and ```ember-cli-sass``` as described in the [README](https://github.com/yapplabs/ember-modal-dialog/blob/84157c49497adf33024924a8156fe7ce7b5ef6a3/README.md):

```
ember install ember-modal-dialog
ember install ember-cli-sass
```

and replace empty ```app/styles/app.css``` with following ```app/styles/app.scss```:

```css
@import "ember-modal-dialog/ember-modal-structure";
@import "ember-modal-dialog/ember-modal-appearance";
```

Next, we have to modify ```app/templates/posts/show.hbs``` to wrap its content with ```modal-dialog```:

{% raw %}
```html
{{#modal-dialog close="goBackToList"}}
  {{model.title}}
{{/modal-dialog}}
```
{% endraw %}

And as a final step let's define ```goBackToList``` action which handles clicks outside of the modal dialog:

```javascript app/routes/posts/show.js
import Ember from 'ember';

export default Ember.Route.extend({
  // [...]
  actions: {
    goBackToList: function() {
      this.transitionTo('posts');
    }
  }
});
```

And that's all! Wasn't it easy? We have our **routable modal dialog with just a have lines of code**. You can check out this example [on Github](https://github.com/adamniedzielski/modal-dialog-demo). Also, big shout-out to [Ember Modal Dialog maintainers](https://github.com/yapplabs/ember-modal-dialog/#credits)!
