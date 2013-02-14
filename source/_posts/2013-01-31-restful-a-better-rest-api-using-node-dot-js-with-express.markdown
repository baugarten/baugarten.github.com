---
layout: post
title: "Restful: A Better REST API using Node.js with Express"
date: 2013-01-31 13:06
comments: true
categories: 
---

I'm going to introduce to you how to quickly make a REST API using a library I just built call [Restful](https://github.com/baugarten/node-restful). Restful, inspired by Tastypie for Django, generates API endpoints to perform CRUD operations on [Mongoose](https://github.com/LearnBoost/mongoose) models. Restful integrates in [Express](https://github.com/visionmedia/express) applications. Right now, the only return type is JSON but I plan on adding support for HTML responses and possibly others.

I’m going to walk you through how to make a REST API for two models, Users and Notes.

<!-- more -->

Before we start, you need to install `express` and `node-restful`. I will assume you already have mongodb installed

```
mongod
npm install express
npm install node-restful
```

Users can create notes, notes are created by users, notes have titles and bodies, and so on.
First, lets write a little express boilerplate:

```js index.js
var express = require('express'),
    mongoose = require('mongoose'),
    restful = require('../../');
// Make a new Express app
var app = module.exports = express();

// Connect to mongodb
mongoose.connect("mongodb://localhost/restful");

// Use middleware to parse POST data and use custom HTTP methods
app.use(express.bodyParser());
app.use(express.methodOverride());

app.listen(3000);
```

This creates a new app, tells mongoose to connect to the specified mongodb instance, and finally listens for connections on port 3000

Next, we are going to add users by appending the following to our server

```js index.js
//...
app.use(express.methodOverride());

var hashPassword = function(req, res, next) {
  if (!req.body.password) 
    return next({ status: 400, err: "No password!" }); // We can also throw an error from a before route
  req.body.password = bcrypt.hashSync(req.body.password, 10); // Using bcrypt
  return next(); // Call the handler
}

var sendEmail = function(req, res, next) {
  // We can get the user from res.bundle and status code from res.status and 
  // trigger an error by calling next(err) or populate information that would otherwise be miggins
  next(); // I'll just pass though
}

var User = new restful.Model({
  title: "user",
  methods: ['get', 'put', 'delete', {
    type: 'post',
    before: hashPassword, // Before we make run the default POST to create a user, we want to hash the password (implementation omitted)
    after: sendEmail, // After we register them, we will send them a confirmation email
  }],
  schema: mongoose.Schema({
    username: 'string',
    password_hash: 'string',
  }),
});
User.register(app, '/user'); // Register the user model at the localhost:3000/user

app.listen(3000);
```

This registers a new mongoose model called user, makes endpoints for GET, POST, PUT, DELETE operations, and before creating the user using POST, it hashes the password

Now lets register Notes:

```js index.js
var User = new restful.Model(...);
var validateUser = function(req, res, next) {
  if (!req.body.creator) {
    return next({ status: 400, err: "Notes need a creator" });
  }
  User.Model.findById(req.body.creator, function(err, model) {
    if (!model) return next(restful.objectNotFound());
    return next();
  });
}

var Note = new restful.Model({
  title: "note",
  methods: ['get', 'delete', { type: 'post', before: validateUser }, { type: 'put', before: validateUser }],
  schema: mongoose.Schema({
    title: { type: 'string', required: true},
    body: { type: 'string', required: true},
    creator: { type: 'ObjectId', ref: 'user', require: true},
  }),
});
Note.register(app, '/note');
```

Now lets add a custom route for users where we can list all of the notes they have created.

```js index.js
User.userroute("notes", {
  handler: function(req, res, next, err, model) { // we get err and model parameters on detail routes (model being the one model that was found)
    Note.Model.find({ creator: model._id }, function(err, list) {
      if (err) return next({ status: 500, err: "Something went wrong" });
      //res.status is the status code
      res.status = 200;

      // res.bundle is what is returned, serialized to JSON
      res.bundle = list;  
      return next();
    });
  },
  detail: true, // detail routes operate on a single instance, i.e. /user/:id
  methods: ['get'] // only respond to GET requests
});
```
And the route is registered at the key in the dict, so that above route would be at `/user/:id/notes`

And now we have a whole bunch of validation and routes that were generated!

we have

GET /note  
GET /note/:id  
POST /note  
PUT /note/:id  
DELETE /note/:id  
GET /user  
GET /user/:id  
POST /user  
PUT /user/:id  
GET /user/:id/notes  
DELETE /user/:id  

See! Easy.
The [full source code](https://github.com/baugarten/node-restful/blob/master/examples/notes.js) is available on github, as is a more [modular approach](https://github.com/baugarten/node-restful/tree/master/examples/notes)

