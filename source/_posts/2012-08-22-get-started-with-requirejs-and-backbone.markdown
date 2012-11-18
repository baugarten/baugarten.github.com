---
layout: post
title: "Get Started with requireJS and Backbone"
date: 2012-08-22 18:32
comments: true
categories: [Requirejs, Backbone]
---

This is something I struggled with for a while -- hopefully you don’t. If you want to learn how to use requireJS and backbone to unuglify your client side code, keep reading. There's plently of documentation on both requireJS and backbone but its easier to learn by example. If you need to be convinced why to use requireJS, you can refer to their <a href='http://requirejs.org/docs/why.html#1'>faq</a>. Short story is that its amazing for organization, code health, and although I haven’t launched anything with it (yet), I’m sure the speed gains are tremendous if used correctly. And why use Backbone? Well, if you're not already using a client side javascript framework, you're probably doing it wrong. If you’re using another, then you can still follow along, this deals much more with requireJS.

<!-- more -->

Lets get started with requireJS. One of the main benefits of requireJS is that every dependency of a javascript file is clearly and explicitly stated. There are two main ways to state dependencies. The first is to use a shim config, the second is the wrap all your code in a `define`, that lists your dependencies. There are advantages to both. The downside and upside to using shim is that everything is located in one file. This is nice for centralization but makes modules less portable. For this reason, I prefer the second when possible because you don’t need to copy the shim config whenever you reuse a file. We’re going to use a little bit of both for extra clarity, indirection, and so you can how to  use them. Before we start, make an html page and instead of adding all your javascript files, just add this one line

`<script data-main="scripts/main" src="scripts/require.js"></script>`

This loads `requireJS` and makes the entry point `scripts/main.js`. Here's our main file.

``` javascript main.js
require.config({
    paths: {
        'jquery': 'libs/jquery/jquery',
        'underscore': 'libs/underscore/underscore',
        'backbone': 'libs/backbone/backbone',
        'templates': '../templates',
    },
    shim: {
        'backbone': {
            deps: ['underscore', 'jquery'],
        },
    }
});

require([
    ‘app’,
], function(App) {
    App.initialize();
});
```
I just threw a lot at you. First, I set up a few rules for requireJS, telling it to look for the alias `jquery` at libs/jquery/jquery.js (it automatically appends a .js), `backbone` in libs/backbone/backbone.js, and the same for `underscore`, and then I set up a templates directory in ../templates. Next, I made a shim configuration, telling requireJS that backbone relies on underscore and jquery to work, so requireJS has to load jquery and underscore before backbone. Next, it loads `app.js` and calls `initialize()`.

Now, I have to be honest, I haven’t told you the whole truth about how its loading these libraries. To add another level of indirection and ensure modular, strict dependency declaration, libs/jquery/jquery.js looks a little more like this

``` javascript libs/jquery/jquery.js
define([
   'libs/jquery/jquery-min'
], function() {
   return jQuery;
});
```

which loads the correct jquery and then returns it for other modules to use. This way, when we update versions of jquery we don’t have to touch our `main.js` but instead jquery controls how it loads itself. We do the same for underscore

``` javascript libs/underscore/underscore.js
define([
    'libs/underscore/underscore-min'
], function() {
    return _;
});
```

Backbone is a little different though because it has dependencies, we use `Backbone.noConflict()` to export a local reference to Backbone versus introducing it into the global namespace. 

``` javascript lib/backbone/backbone.js
define([
    'libs/backbone/backbone-min'
], function() {
    _.noConflict();
    $.noConflict();
    return Backbone.noConflict();
});
```
Now that we have all of our dependencies straightened up, lets get back to the whole `require(['app'], function(App) { ... });`. This is the entry point of our application. It loads `app.js` and then executes `App.initialize()`. My `app.js` looks like

``` javascript app.js
define([
    'router'
], function(Router) {
    var initialize = function() {
        Router.initialize();
    }
    return {
        initialize: initialize
    };
});
```
This loads `router.js`, my Backbone router and then from the backbone router, I load the necessary components.

``` javascript router.js
define([
    'jquery',
    'underscore',
    'backbone',
    // whatever other dependencies you need
    ‘models/mymodel’ // or alias it in main.js
], function($, _, Backbone, MyModel) {
    var AppRouter = Backbone.Router.extend({
        // implement your router
    })
    var initialize = function() {
        var router = new AppRouter;
        var myModel = new MyModel;
        // do all the initialization work
    }
    return { initialize: initialize };
});
```

And now your Backbone models look like this:
``` javascript mymodel.js
define([
    'jquery',
    'backbone',
], function ($, Backbone) {
    var MyModel= Backbone.Model.extend({
        // your Backbone model
    });
    return MyModel;
});
```

And now you can add whatever other models, views or collections you want by adding it to `main.js`. Be sure to wrap it in a `define` to state all its dependencies and load it asynchronously wherever you want!

I know this has covered a lot, email me if you have any questions.

If you're still down here, you should follow me on <a href="http://twitter.com/benaugarten">twitter</a>

Or maybe you have a job offer? Check out my <a href="http://benaugarten.com/hireme/">resume</a> or send me an <a href="mailto:baugarten@gmail.com">email</a>. I'm looking for freelance work.

