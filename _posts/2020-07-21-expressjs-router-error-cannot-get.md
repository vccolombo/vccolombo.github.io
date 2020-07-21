---
title: "Express.js router error 'Cannot GET /something'"
excerpt: "How I fixed this simple error when implementing an MVC architecture in Node.js"
date: 2020-07-21T11:50:30-03:00
categories:
  - Blog
tags:
  - Express.js
  - Node.js
---

I was trying to write a web API using [Express.js routers](https://expressjs.com/en/guide/routing.html), when I encountered the following error while making a GET request to /users:

```html
<!-- GET /users -->
<body>
  <pre>Cannot GET /users</pre>
</body>
```

I spent a lot of time on Google trying to find what was the issue, with no success. My code was the following:

```javascript
// app.js
var express = require('express');

var users = require('./routes/usersRoute');

var app = express();
app.use('/users', users);

app.listen(3000);
```

```javascript
// routes/userRoute.js
var express = require('express');
var router = express.Router();

var controller = require('../controllers/usersController');

router.get('/users', controller.get);

module.exports = router;
```

## The problem

By specifying `app.use('/users', users)` in app.js, Express was already defining that requests to the `/users` endpoint would be routed to the users Router. It means that when I defined `router.get('/users', controller.get)` in userRoute.js, I was mistakenly asking Express to route requests to `/users/users` instead of `/users`.

## The solution

Just change the get path to `router.get('/', controller.get)` in the Router.

Hope this can help people with the same issue as me.