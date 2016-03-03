---
layout: post
title: 'Emails with "View in Browser" link in Node.js'
date: 2015-10-16 16:52:10 +0200
comments: true
categories:
- Node.js
- Koa
- generators
- sending emails
- promises
- javascript
---

"View in Browser" link is a common feature of email campaign systems like MailChimp. It's a really nice idea - when you code your shiny CSS-heavy emails you want to have a fallback for malfunctioning email clients.

But what if you want to implement this feature on your own? Read on to learn **how to send emails in Node.js with "View in Browser" link and inlined CSS**.

<!-- more -->

We will use [Nodemailer](https://github.com/andris9/Nodemailer) for email **transport** and [node-email-templates](https://github.com/niftylettuce/node-email-templates) for rendering **templates** and **inlining** CSS styles. The code was tested under Node.js 4.2.1.

Let's start with implementing just the email transport. Nodemailer has "callback-based" API while I prefer to use **generator based flow control** offered by [co](https://github.com/tj/co). That's why we need [Q](https://github.com/kriskowal/q) to convert "callback-based" API to "promise-based" API which can be used by co.


```javascript send.js
var co = require("co");
var nodemailer = require('nodemailer');
var Q = require("q");

function *send() {
  var transporter = nodemailer.createTransport({
    port: 1025
  });

  var rawSend = Q.nbind(transporter.sendMail, transporter);
  yield rawSend({
      from: 'test1@example.com',
      to: 'test2@example.com',
      subject: 'Test',
      text: 'Test123!'
  });
}

co(function* () {
  yield send();
}).catch(function (err) {
  console.error(err.stack);
});
```

We are using the default SMTP transport here. You can use [MailCatcher](http://mailcatcher.me/) for development, to quickly have something running. For production I recommend checking out other transports like [nodemailer-mandrill-transport](https://github.com/rebelmail/nodemailer-mandrill-transport).

The next step is to use node-email-templates. I decided to use [Jade](http://jade-lang.com/) for templating and [Sass](http://sass-lang.com/) as CSS preprocessor:

```bash
npm install --save jade
npm install --save node-sass
```

We have to implement ```renderContent``` function, create HTML template and add some CSS styles:


```javascript send.js
// [...]

function *send() {
  var transporter = nodemailer.createTransport({
    port: 1025
  });

  var content = yield renderContent();

  var rawSend = Q.nbind(transporter.sendMail, transporter);
  yield rawSend({
      from: 'test1@example.com',
      to: 'test2@example.com',
      subject: 'Test',
      html: content.html
  });
}

function *renderContent() {
  var templateDir = path.join(__dirname, 'invitation');
  var invitation = new EmailTemplate(templateDir);
  var context = { userName: "Adam" };

  var render = Q.nbind(invitation.render, invitation);
  return yield render(context);
}

// [...]
```

```jade invitation/html.jade
h1 Hello
  span.name  #{userName}!
p This is some content for you
```

```scss invitation/style.scss
h1 {
  .name {
    color: red;
  }
}
```

Inside ```context``` object we pass values for interpolation in the template. Please also note that Sass styles are **automatically** compiled to CSS and inlined for us!

How are we going to implement "**View in Browser**" functionality? My approach is simple - store generated email content in MongoDB, assign it some long random identifier and serve this content via a web application.

Let's start by creating a function which connects to MongoDB and inserts email content identified by the token (UUID). I'm hardcoding address of local MongoDB instance for the purpose of this example.

```javascript
function *send() {
  var transporter = nodemailer.createTransport({
    port: 1025
  });

  var token = uuid.v4();

  var content = yield renderContent();

  var rawSend = Q.nbind(transporter.sendMail, transporter);
  yield [rawSend({
      from: 'test1@example.com',
      to: 'test2@example.com',
      subject: 'Test',
      html: content.html
  }), saveContent(token, content)];
}

function *saveContent(token, content) {
  var connect = Q.nbind(MongoClient.connect, MongoClient);
  var db = yield connect("mongodb://localhost:27017/nodejs-email-demo");

  var collection = db.collection('emails');

  var insert = Q.nbind(collection.insertOne, collection);
  yield insert({
    token: token,
    content: content.html
  });

  db.close();
}
```

Please note that we are executing ```rawSend``` and ```saveContent``` **concurrently**, because we are passing array of promises to ```yield```.

As the next step, let's extract the code related to database communication to a separate module - ```db.js```. We need ```find``` function which shares some code with ```save``` function so we extract to avoid duplicating the code.


```javascript send.js
function *saveContent(token, content) {
  yield db.save(token, content.html);
}

```

```javascript db.js
var MongoClient = require('mongodb').MongoClient;
var Q = require("q");

function *connect() {
  var conn = Q.nbind(MongoClient.connect, MongoClient);
  return yield conn("mongodb://localhost:27017/nodejs-email-demo");
}

module.exports = {
  save: function*(token, content) {
    var db = yield connect();
    var collection = db.collection("emails");

    var insert = Q.nbind(collection.insertOne, collection);
    yield insert({
      token: token,
      content: content
    });

    db.close();
  },

  find: function*(token) {
    var db = yield connect();
    var collection = db.collection("emails");

    var find = Q.nbind(collection.findOne, collection);
    var email = yield find({ token: token });

    db.close();
    return email;
  }
}
```

For our web application we will use ```koa``` with ```koa-router```. Koa is based on generators and allows us to implement the functionality with only a couple lines of code:

```javascript app.js
var koa = require('koa');
var app = koa();
var router = require('koa-router')();
var db = require("./db");

router.get('/emails/:id', function*() {
  var email = yield db.find(this.params.id);

  if (email) {
    this.body = email.content;
  }
});

app.use(router.routes()).use(router.allowedMethods());
app.listen(3000);
```

Please mind that in the real application, you will probably have a shared MongoDB connection and won't connect from scratch for each incoming request.

The final step is to **add the link** to our web application inside the email template:

```jade invitation/html.jade
h1 Hello
  span.name  #{userName}!
p This is some content for you

a(href=viewInBrowserUrl) View in Browser
```

```javascript send.js
function *renderContent(token) {
  var templateDir = path.join(__dirname, 'invitation');
  var invitation = new EmailTemplate(templateDir);
  var context = {
    userName: "Adam",
    viewInBrowserUrl: "http://localhost:3000/emails/" + token
  };

  var render = Q.nbind(invitation.render, invitation);
  return yield render(context);
}
```

If you launch the web application and MailCatcher:

```bash
node app.js
mailcatcher
```

You should be able to send email with:

```bash
node send.js
```

"View in Browser" link will show content of the email served by our web application.

You can check out the complete example [on Github](https://github.com/adamniedzielski/nodejs-email-demo).
