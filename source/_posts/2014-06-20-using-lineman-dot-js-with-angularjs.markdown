---
layout: post
title: "Using Lineman.js with AngularJS"
date: 2014-06-20 21:10:48 +0200
comments: true
categories:
- javascript
- tools
- AngularJS
- Lineman.js
---

I've recently started **playing with AngularJS**. Coming from the Rails world I expected **a structure** enforced by the framework, good **default configuration** and separation into development, test and production environments.

It seems that the way to start a new project suggested by the creators of AngularJS is to clone [angular-seed project](https://github.com/angular/angular-seed). While this seed project worked for me during development and testing, I was **a bit lost** when it came to **deploying** to production.

<!-- more -->

The README is [quite vague here](https://github.com/angular/angular-seed#running-the-app-in-production) and the same can be said about [the comment in index.html](https://github.com/angular/angular-seed/blob/master/app/index.html#L31-33)

Should I **change** this one line each time I make a deployment, then **copy** the files to my server and finally **undo** the change? Next thing - there is nothing said about minification in the README. As a fresh AngularJS developer I was quite disappointed that I wasn't able to have my app up and running quickly.

#### Lineman.js enters the stage

And there I stumbled upon Lineman.js. At first I thought - *"yet another tool sprung in the complex world of JavaScript"*. But then I decided to give it a try.

Lineman.js provides a good structure for your client-side application and is **opinionated** about how you should run your commands. This means that **Lineman.js makes** a couple of **decisions** for me, which I won't be able to make as a developer learning AngularJS.

Lineman.js takes care about development and testing stages, but does not forget about **pushing code to production**. It knows that you want your code to be concatenated and minified in production.

You can start with [lineman-angular-template](https://github.com/linemanjs/lineman-angular-template), which gives you some nice defaults such as:

* [grunt-ngmin](https://github.com/btford/grunt-ngmin) config, so you can write:

```javascript
angular.module('app').
  controller('ArtistController', function ($scope, $routeParams) {
    ...
  });
```

instead of:

```javascript
angular.module('app').
  controller('ArtistController', ['$scope', '$routeParams', function ($scope, $routeParams) {
    ...
  }]);
```

and your code will still be minified correctly

* [sourcemaps](http://www.html5rocks.com/en/tutorials/developertools/sourcemaps/)

* template precompilation with [grunt-angular-templates](https://github.com/ericclemmons/grunt-angular-templates) so you can avoid making AJAX request for each template

* unit tests with [Jasmine](https://github.com/pivotal/jasmine) and [testem](https://github.com/airportyh/testem), re-run after each file change

{% img center /images/lineman/jasmine.png %}

* end-to-end tests with [Protractor](https://github.com/angular/protractor)

* effortless deployment to Heroku with [custom Heroku Buildpack](https://github.com/linemanjs/heroku-buildpack-lineman)


#### Conclusion

Lineman.js is very easy to learn - [start from its homepage](http://linemanjs.com). You can check my [AngularJS toy project - classky](https://github.com/adamniedzielski/classky-angular) which I built with Lineman.js. Everything worked out of the box and I cannot see any negative sides of using this tool. **Recommended!**
