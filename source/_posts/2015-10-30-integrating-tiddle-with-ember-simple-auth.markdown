---
layout: post
title: "Integrating Tiddle with Ember Simple Auth"
date: 2015-10-30 14:16:32 +0100
comments: true
categories:
- Ruby
- Ruby on Rails
- Rails
- gem
- Tiddle
- JSON
- API
- token authentication
- Devise
- multiple tokens
- javascript
- Ember.js
- Ember Simple Auth
---

**Tiddle** is my gem which provides Devise strategy for **token authentication** in API-only Ruby on Rails applications. Its selling point is that it supports **multiple tokens per user**.

**Ember Simple Auth** provides a nice abstraction over implementing authentication in Ember.js apps. In this blog post I will show you how to implement a custom Ember Simple Auth strategy for Tiddle.

<!-- more -->

If you have never heard of Tiddle you should check out my previous blog post about it - [Token authentication with Tiddle](/blog/2015/04/04/token-authentication-with-tiddle). It describes how to set up Tiddle in Ruby on Rails application.

For the purpose of this example I'm going to assume that Rails application is running on ```localhost:3000```.

Firstly you have to follow the steps described in [Ember Simple Auth - Basic Usage](https://github.com/simplabs/ember-simple-auth#basic-usage), namely:

- login / logout link in application template
- ```invalidateSession``` action in application controller
- login route, template and controller
- ```ApplicationRouteMixin``` in the application route

Our protected route is ```posts``` route:

```javascript app/routes/posts.js
import Ember from 'ember';
import AuthenticatedRouteMixin from 'ember-simple-auth/mixins/authenticated-route-mixin';

export default Ember.Route.extend(AuthenticatedRouteMixin, {
  ajax: Ember.inject.service(),

  model: function() {
    return this.get('ajax').request('/posts.json');
  }
});
```

As you can see I'm extending ```AuthenticatedRouteMixin``` to prevent access for unauthenticated users. This will transition the user to the login route when she's not authenticated.

Secondly, I'm using a custom service for communicating with the API. Let's take a look at it:

```javascript app/services/ajax.js
import Ember from 'ember';
import AjaxService from 'ember-ajax/services/ajax';
import { UnauthorizedError } from 'ember-ajax/errors';

export default AjaxService.extend({
  session: Ember.inject.service(),
  host: 'http://localhost:3000',

  request(url, options) {
    this.get('session').authorize('authorizer:tiddle', (headers) => {
      this.set('headers', headers);
    });

    return this._super(url, options).
      catch((error) => {
        if (error instanceof UnauthorizedError) {
          if (this.get('session.isAuthenticated')) {
            this.get('session').invalidate();
          }
        }
        else {
          throw error;
        }
      });
  }
});
```

I'm extending the service provided by [ember-ajax](https://github.com/ember-cli/ember-ajax) - a nice wrapper over ```jQuery.ajax```. It's calling the authorizer to get the correct authentication headers (```X-USER-EMAIL``` and ```X-USER-TOKEN```). We are also catching all ```401 Unauthorized``` responses to invalidate the session. This accounts for the situation when session expired on the backend, but our frontend still thinks it's authenticated.

Now we have to implement our custom authenticator:

```javascript app/authenticators/tiddle.js
import Ember from 'ember';
import Base from 'ember-simple-auth/authenticators/base';

export default Base.extend({
  ajax: Ember.inject.service(),

  restore(data) {
    return new Ember.RSVP.Promise((resolve, reject) => {
      if (!Ember.isEmpty(Ember.get(data, 'email')) && !Ember.isEmpty(Ember.get(data, 'token'))) {
        resolve(data);
      } else {
        reject();
      }
    });
  },
  authenticate(email, password) {
    return this.get('ajax').post('/users/sign_in.json', { data: { user: { email, password }}}).
      then(response => {
        return { email, token: response.authentication_token };
      });
  },

  invalidate(data) {
    return this.get('ajax').del('/users/sign_out.json');
  }
});
```

The interface consists of three methods:

1. ```restore``` - This is used to restore the session based on what is saved in the session store (Local Storage with fallback to cookies). We just have to check that ```email``` and ```token``` are present.

2. ```authenticate``` - Here we are using our ```ajax``` service to make a request to the API, passing email and password. Then we return the object which contains email and the authentication token. Ember Simple Auth persists it in the session storage.

3. ```invalidate``` - We are making the request to the API to remove the current token from the database. Ember Simple Auth takes care about clearing the session storage.

The last missing part is our custom authorizer. It should return the correct HTTP headers:

```javascript app/authorizers/tiddle.js
import Ember from 'ember';
import Base from 'ember-simple-auth/authorizers/base';

export default Base.extend({
  authorize(sessionData, block) {
    const headers = {
      "X-USER-EMAIL": Ember.get(sessionData, 'email'),
      "X-USER-TOKEN": Ember.get(sessionData, 'token')
    };

    block(headers);
  }
});
```

```authorize``` method receives the data from the session store so it's really easy to build ```X-USER-EMAIL``` and ```X-USER-TOKEN``` headers required by Tiddle on the server side.

### Summary

Thanks to Ember Simple Auth we have a clean implementation of authentication logic and many features working out of the box. You can [check out the complete example here](https://github.com/adamniedzielski/tiddle-ember-demo)
