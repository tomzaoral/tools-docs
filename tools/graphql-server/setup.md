---
title: Adding a GraphQL endpoint
order: 202
description: How to add a GraphQL endpoint to your server.
---

GraphQL Server has a slightly different API depending on which server integration you are using, but all of the packages share the same core implementation and options format.

<h2 id="graphqlOptions">GraphQL Server options</h2>

GraphQL Server accepts a `GraphQLOptions` object as its single argument. The `GraphQLOptions` object has the following properties:

```js
// options object
const GraphQLOptions = {
  schema: GraphQLSchema,

  // values to be used as context and rootValue in resolvers
  context?: any,
  rootValue?: any,

  // function used to format errors before returning them to clients
  formatError?: Function,

  // additional validation rules to be applied to client-specified queries
  validationRules?: Array<ValidationRule>,

  // function applied for each query in a batch to format parameters before passing them to `runQuery`
  formatParams?: Function,

  // function applied to each response before returning data to clients
  formatResponse?: Function,

  // a boolean option that will trigger additional debug logging if execution errors occur
  debug?: boolean
})
```

<h3 id="options-function">Passing options as a function</h3>

Alternatively, GraphQL Server can accept a function which takes the request as input and returns a GraphQLOptions object or a promise that resolves to one:

```js
// example options function (for express)
graphqlExpress(request => ({
  schema: typeDefinitionArray,
  context: { user: request.session.user }
}))
```

This is useful if you need to attach objects to your context on a per-request basis, for example to initialize user data, caching tools like `dataloader`, or set up some API keys.

<h2 id="graphqlExpress">Using with Express</h2>

The following code snippet shows how to use GraphQL Server with Express:

```js
import express from 'express';
import bodyParser from 'body-parser';
import { graphqlExpress } from 'graphql-server-express';

const PORT = 3000;

var app = express();

app.use('/graphql', bodyParser.json(), graphqlExpress({ schema: myGraphQLSchema }));

app.listen(PORT);
```

<h2 id="graphqlConnect">Using with Connect</h2>

Connect is so similar to Express that the integration is in the same package. The following code snippet shows how to use GraphQL Server with Connect:

```js
import connect from 'connect';
import bodyParser from 'body-parser';
import { graphqlConnect } from 'graphql-server-express';
import http from 'http';

const PORT = 3000;

var app = connect();

app.use('/graphql', bodyParser.json());
app.use('/graphql', graphqlConnect({ schema: myGraphQLSchema }));

http.createServer(app).listen(PORT);
```

The arguments passed to `graphqlConnect` are the same as those passed to `graphqlExpress`.

<h2 id="graphqlHapi">Using with Hapi</h2>

The following code snippet shows how to use GraphQL Server with Hapi:

```js
import hapi from 'hapi';
import { graphqlHapi } from 'graphql-server-hapi';

const server = new hapi.Server();

const HOST = 'localhost';
const PORT = 3000;

server.connection({
    host: HOST,
    port: PORT,
});

server.register({
  register: graphqlHapi,
  options: {
    path: '/graphql',
    graphqlOptions: { schema: myGraphQLSchema },
  },
});

server.start((err) => {
	if (err) {
		throw err;
	}
});
```

`graphqlOptions` can also be a callback or a promise:

```js
server.register({
  register: graphqlHapi,
  options: {
    path: '/graphql',
    graphqlOptions: (request) => {
      return { schema: myGraphQLSchema };
    },
  },
});
```

<h2 id="graphqlKoa">Using with Koa 2</h2>

The following code snippet shows how to use GraphQL Server with Koa:

```js
import koa from 'koa';
import koaRouter from 'koa-router';
import koaBody from 'koa-bodyparser';
import { graphqlKoa } from 'graphql-server-koa';

const app = new koa();
const router = new koaRouter();
const PORT = 3000;

app.use(koaBody());

router.post('/graphql', graphqlKoa({ schema: myGraphQLSchema }));
app.use(router.routes());
app.use(router.allowedMethods());
app.listen(PORT);
```

`graphqlOptions` can also be a callback that returns a GraphQLOptions or returns a promise that resolves to GraphQLOptions. This function takes a koa 2 `ctx` as its input.

```js
router.post('/graphql', graphqlKoa((ctx) => {
  return { 
    schema: myGraphQLSchema,
    context: { userId: ctx.cookies.get('userId') }
  };
}));
```
