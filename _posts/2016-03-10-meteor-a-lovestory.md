---
title: Meteor.js - A lovestory
updated: 2016-03-10 23:20
category: Meteor.js
tweet: 708063278568710144
---

Meteor.js is a **full stack javascript framework** for building modern web applications on a unified javascript stack (MongoDB, Node.js, Browser). If you want to build applications fast, it's definitely the way to go.
This article provides a rough overview of some selected concepts and techniques that I use for side projects.

Some key benefits of Meteor.js are:

- Barely any dev-ops from the first line to production
- Sophisticated template engine
- Realtime per default
- Painless Cordova mobile builds

Meteor.js is very well [documented](http://docs.meteor.com/#/full/) and has a huge package-catalog called [Atmosphere](https://atmospherejs.com/).

## Installation

First things first: You need [Node.js](http://lmgtfy.com/?q=install+node.js) - yolo.

```sh
curl https://install.meteor.com/ | sh
```

That's it for the cool kids (more for the [windows peeps](https://install.meteor.com/windows) - painless too - it's a .exe lol). You now have ```meteor```. That's all you need.

## Project bootstrapping

```sh
meteor create leproject
meteor
```

We create a project named „leproject“ (:guardsman: obviously ur own project name). After running the project with ```meteor``` we are ready for code. The dev environment includes a running MongoDB host,  Node.js server and live reload with [hotcodepush](http://info.meteor.com/blog/meteor-hot-code-push). Meteor.js also creates some sample files to get you up and running.

## App structure

Basically its simple: **All files are concatenated** and shipped. :metal:

There are some „special“ directories inside your project affecting load order, destination of script files, and some other characteristics.

- **/client** javascript that should solely be shipped to the client. Will not affect Node.js. Files in this directory are executed before other client code. Equivalent to ```if (Meteor.isClient) {}``` wrapped code.
- **/server** as you can guess this is the opposite of client:  ```if (Meteor.isServer) {}```
- **/public** served as-is to the client. Images, icons and other static unprocessed assets.
- **/private** Only accessible from server code. Can be loaded via [Assets api](http://docs.meteor.com/#assets).
- **/lib** common code like collections and utilities. Loaded first.

When Meteor.js concatenates javascript, there are some rules for controlling the order. From first to last:
*HTML templates > files beginning with main > files inside lib > deeper Path > alphabetical*

Read [more about load order](http://docs.meteor.com/#/full/fileloadorder) on meteor.com.

## Views

Meteor.js' in-house template engine is called **blaze**. It's declarative, simple and reactive. The matching template language is called spacebars, which is Meteor.js' dialect of Handlebars.

> „Blaze fulfills the same purpose as Angular, Backbone, Ember, React, Polymer, or Knockout, but is much easier to use.“

It's also possible to switch the engine to [react.js](https://www.meteor.com/tutorials/react/creating-an-app) or [angular.js](https://www.meteor.com/tutorials/angular/creating-an-app).

Example template from app boilerplate:
{% raw %}
```html
<head>
  <title>yolo</title>
</head>

<body>
  <h1>Welcome to Meteor!</h1>
  {{> hello}}
</body>

<template name="hello">
  <button>Click Me</button>
  <p>You ve pressed the button {{counter}} times.</p>
</template>
```
{% endraw %}

## UI flow

Every template has its own data scope with **helpers**. User interactions are handled via **events**. That's all.

```js
Session.setDefault('counter', 0);

Template.hello.helpers({
  counter: function () {
    return Session.get('counter');
  }
});

Template.hello.events({
  'click button': function () {
    Session.set('counter', Session.get('counter') + 1);
  }
});
```

## Preprocessing Less

A very common task is setting up your CSS preprocessor. Meteor.js' amazing package system handles this extremely painless:

```sh
meteor add less
```

Add a less file and it’s processed before publishing. Source map support, a watcher for live reload and everything. Same for coffee script, sass whatsoever.

## Deploying :open_mouth:

After 5 minutes of sophisticated hacking, it's time to deploy our web application. From my experience, this is the part where the beer comes in. Searching for a poster, setting up Linux, installing a web server, configuring databases … :beer:
As I mentioned in the very first paragraph, Meteor.js is about rapid prototyping. So forget all the beers stuff about configuration and installing apt stuff. We do it the Meteor.js way:

```sh
meteor deploy leproject
```
:guardsman: Cpt. obvious again: leproject is your project name.

Your app is now live at leproject.meteor.com - Officially this is for prototyping only and is no production solution. It’s a fast way to get all the dev-ops stuff out of the way and just build the MVP.

## Mobile builds

As Meteor.js has Cordova built-in, this is also a very straight forward process. Add the platform and run it.

```sh
meteor add-platform ios
meteor run ios
```

This will fire up the iOS simulator from Xcode and boots your app in a PhoneGap wrapper.

```meteor run ios-device``` starts the application on a connected iOS device. More about [running on mobile](https://www.meteor.com/tutorials/blaze/running-on-mobile).

## Collections & Data

Meteor.js' persistent data layer is MongoDB with some more magic.

Data fetching is done with [DDP](https://www.meteor.com/ddp) (Distributed Data Protocol). Just remember its **REST for Websockets**. Everything in Meteor.js is real-time by default, which is done by livequery a live database connector for Mongo (theoretically also MySQL #wtf). From a very high perspective: You execute a query, receive the result and all subsequent changes to this very query via web socket. :raised_hands:

Meteor.js also feels fast because of a concept called **Optimistic UI**. All the data that is transferred between MongoDB and clients gets cached in a minimongo. If a template gets rendered it instantly renders with data from minimongo and then updates with data from MongoDB.

Enough theory - now some code. Somewhere in your global scope:

```js
Yos = new Mongo.Collection('yos');
```

Add a new event to a template containing a form:

```js
Template.form.events({
  "submit form": function (event) {
    event.preventDefault();

    Yos.insert({
      createdAt: new Date()
    });
  }
});
```

You just successfully added a record and pushed it in realtime to all clients observing a query like:

```js
Template.list.helpers({
  yos: function() {
    return Yos.find({}, {limit:10, sort:{createdAt:-1}});
  }
});
```

## MVC

As Meteor.js doesn't come with a built-in router there are some 3rd party routers. Meteor.js guide recommends [flow-router](https://atmospherejs.com/kadira/flow-router). Another common router is [Iron Router](https://github.com/iron-meteor/iron-router).

Install iron router via Atmosphere - again easy as firing:

```sh
meteor add iron:router
```

Sample route:

```js
Router.route('/', function () {
  this.render(
    'Index',
    {data: {iAmFlag: 1}}
  );
});
```

## „We need user accounts“

Implementing authentication using ember.js or similar client frameworks can be really time-consuming: Implementation for the client, a backend service that does the user logic, session handling, emailing … yadda yadda yadda. As Metero.js is a full stack framework **Meteor.js packages are also full stack**.

So let's point you to the fast lane:

```sh
meteor add accounts-ui accounts-password
```

Bruh. Done.

{% raw %}
In your template include the ```{{> loginButtons}}``` partial to register/login. And you have ```{{#if currentUser}}``` available to do stuff for logged in users.
{% endraw %}

Same with OAuth: ```meteor add accounts-facebook``` let's you do a fully fledged Facebook OAuth. Woooot? :scream:

## Seo

```sh
meteor add spiderable
```

> Uses the AJAX Crawling specification published by Google to serve HTML to compatible spiders (Google, Bing, Yandex, and more).

Uses phantom.js to serve a „spiderable“ static version of your javascript application behind the ```?_escaped_fragment_=```


## Final words

I :green_heart: Meteor.js for getting non-funky stuff out of the way. You can build fast and modern stuff very efficiently and focus on the more refined stuff in later iterations.
