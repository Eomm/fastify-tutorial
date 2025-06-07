# Routing
Fastify is a web server and as such it supports the concept of routing. You can easily define routes and handle requests using Fastify's routing system.

As usual we start by writing some tests to crystalize our expectations.
```javascript
import { test } from 'node:test';
import server from './routing.js';

test('call two different routes', async (t) => {
  await server
    .inject({
      method: 'GET',
      url: '/hello',
    })
    .then((res) => {
      t.assert.equal(res.statusCode, 200);
      t.assert.equal(res.payload, 'Hello World!');
    });

  await server
    .inject({
      method: 'POST',
      url: '/bye',
    })
    .then((res) => {
      t.assert.equal(res.statusCode, 200);
      t.assert.equal(res.payload, 'Goodbye!');
    });
});
```

if we run our tests now with `node --test ./routing.test.js` we should see them fail.
Let's make them pass. Create a new file called `routing.js` and add the following code:

```javascript
import fastify from 'fastify';

const server = fastify();

server.get('/hello', (request, reply) => {
  reply.send('Hello World!');
});

server.route({
  method: 'POST',
  path: '/bye',
  handler: (request, reply) => {
    reply.send('Goodbye!');
  },
});

export default server;
```

As you can see we can define routes in different ways, the result is the same.

## Route Parameter

Let's try to define a dynamic route that accepts a parameter.
As usual we start by writing a test first:

```javascript
test('call a dynamic route', async (t) => {
  const res1 = await server.inject({
    method: 'GET',
    url: '/user/1',
  });

  const res2 = await server.inject({
    method: 'GET',
    url: '/user/2',
  });

  t.assert.equal(res1.statusCode, 200);
  t.assert.equal(res1.payload, 'User 1');

  t.assert.equal(res2.statusCode, 200);
  t.assert.equal(res2.payload, 'User 2');
});
```

As you can see we are calling the route `user/1` and we expect to get back the string `User 1` and when we call `user/2` we expect to get back the string `User 2`. It's clear that the last portion on the URL must be dynamic.

Now if we run our tests now with `node --test ./routing.test.js` we should see them fail with this output:
```
✖ call a dynamic route (1.482834ms)
  AssertionError [ERR_ASSERTION]: 404 == 200
      at assert.<computed> [as equal] (node:internal/test_runner/test:254:18)
      at TestContext.<anonymous> (file:///Users/jpm/Documents/personal/learn-fastify-with-test/code/routing/routing.test.js:37:12)
      at process.processTicksAndRejections (node:internal/process/task_queues:105:5)
      at async Test.run (node:internal/test_runner/test:981:9)
      at async Test.processPendingSubtests (node:internal/test_runner/test:678:7) {
    generatedMessage: true,
    code: 'ERR_ASSERTION',
    actual: 404,
    expected: 200,
    operator: '=='
  }
```

Let's now define a dynamic route that accepts a parameter.
Let's add to the implementation file the dynamic route definition.

```javascript
server.get('/user/:id', (request, reply) => {
  reply.send(`User ${request.params.id}`);
});
```

As you can see, with Fastify you can declare the dynamic portion of the URL using the ":" symbol. What's placed after that is going to be used as §the name of the property that will be available in the `request.params` object.
That property will hold the value of the dynamic portion of the URL. Let's now re-run our tests. We should see them pass.

## Query Parameters

As with route parameters, you can also provide a query parameter. Let's write a test for that:

```javascript
test('call a dynamic route with query parameter', async (t) => {
  const res = await server.inject({
    method: 'GET',
    url: '/user/1?name=John',
  });

  t.assert.equal(res.statusCode, 200);
  t.assert.equal(res.payload, 'User 1 named John');
});
```

Let's see them fail. Run `node --test ./routing.test.js`.
You should see the following output:
```
✖ call a dynamic route with query parameter (1.103666ms)
  AssertionError [ERR_ASSERTION]: 'User 1' == 'User 1 named John'
      at assert.<computed> [as equal] (node:internal/test_runner/test:254:18)
      at TestContext.<anonymous> (file:///Users/jpm/Documents/personal/learn-fastify-with-test/code/routing/routing.test.js:51:12)
      at process.processTicksAndRejections (node:internal/process/task_queues:105:5)
      at async Test.run (node:internal/test_runner/test:981:9)
      at async Test.processPendingSubtests (node:internal/test_runner/test:678:7) {
    generatedMessage: true,
    code: 'ERR_ASSERTION',
    actual: 'User 1',
    expected: 'User 1 named John',
    operator: '=='
  }
```

Now let's iterate over it. We should change the implementation of the user route we just created. Let's write enough code to make it pass:

```javascript
server.get('/user/:id', (request, reply) => {
  reply.send(`User ${request.params.id} named John`);
});
```

If we re-run our tests, you can see that the test we have just written passes. But we have the old test failing.

As you can clearly see we have changed the behavior of our route not respecting anymore the previous expectation. Tests are useful not only to shorten the "development to validation" cycle but also they are a way to "set in stone" a behavior that you need to be unchanged. While new features are being developed, tests ensure that the existing behavior remains consistent.
Another small thing. You can isolate a specific test, or group of tests, in your test suite by using `test.only` or `it.only` or `describe.only` function. Editing our test file like so:

```javascript
test.only('call a dynamic route', async (t) => {
  const res1 = await server.inject({
    method: 'GET',
    url: '/user/1',
  });

  const res2 = await server.inject({
    method: 'GET',
    url: '/user/2',
  });

  t.assert.equal(res1.statusCode, 200);
  t.assert.equal(res1.payload, 'User 1');

  t.assert.equal(res2.statusCode, 200);
  t.assert.equal(res2.payload, 'User 2');
});
```
And then you can run `node --test --test-only ./routing.test.js`.
You should see an output similar to this:

```
✖ call a dynamic route (9.137708ms)
  AssertionError [ERR_ASSERTION]: 'User 1 named John' == 'User 1'
      at assert.<computed> [as equal] (node:internal/test_runner/test:254:18)
      at TestContext.<anonymous> (file:///Users/jpm/Documents/personal/learn-fastify-with-test/code/routing/routing.test.js:38:12)
      at process.processTicksAndRejections (node:internal/process/task_queues:105:5)
      at async Test.run (node:internal/test_runner/test:981:9)
      at async startSubtestAfterBootstrap (node:internal/test_runner/harness:297:3) {
    generatedMessage: true,
    code: 'ERR_ASSERTION',
    actual: 'User 1 named John',
    expected: 'User 1',
    operator: '=='
  }
```

Isolating tests can be used for debugging, if you log something to the console or placing breakpoints in your code you don't risk to be disturbed by other tests.

Let's edit our source code to fix this test:

```javascript
server.get('/user/:id', (request, reply) => {
  const name = request.query.name;
  const nameString = name ? ` named ${name}` : '';
  reply.send(`User ${request.params.id}${nameString}`);
});
```

We need to add the second part on the string only when the `name` query parameter is present.
Let's see if this implementation works. Run the whole test suite again: `node --test ./routing.test.js`.
You should see an output similar to this:

```
✔ code/routing/routing.js (112.400334ms)
✔ call two different routes (9.014916ms)
✔ call a dynamic route (0.633417ms)
ℹ 'only' and 'runOnly' require the --test-only command-line option.
✔ call a dynamic route with query parameter (0.451417ms)
ℹ tests 4
ℹ suites 0
ℹ pass 4
ℹ fail 0
ℹ cancelled 0
ℹ skipped 0
ℹ todo 0
ℹ duration_ms 129.273333
```

If you take a closer look at the output, you can see that nodejs is detecting that we have a test that is declared as `.only`. This is because we forgot to remove it from our test suite. Still node did the right thing and executed the whole test suite. It's important since if we accidentally commit a test that is marked as `.only`, and we relay on our test suite to be green to trigger a deployment we don't accidentally ignore failing tests. In this way, to run only specific tests, we are explicit in two places: in the test invocation and in the test suite.

Anyway we can now remove the `.only` from the test and run the whole test suite again.
All I see is green, fun times.

## Request headers

Let's quickly analyze how we can access the request headers sent by the caller.
Fastify offers a `request.headers` object that contains all the headers sent by the caller, already parsed and normalized. Fastify will lower case all the headers keys for consistency, so you don't have to worry about case sensitivity when accessing headers. For example, if you send a header with the key `Content-Type`, you can access it using `request.headers['content-type']`.

Let's test this behavior. Add the following test:

```javascript
test('normalize headers', async (t) => {
  const res = await server.inject({
    method: 'GET',
    url: '/headers',
    headers: {
      'Content-Type': 'application/json',
      'my-Header': 'my-value'
    },
  });

  t.assert.equal(res.statusCode, 200);
  t.assert.ok(res.payload.includes('application/json my-value'));
});
```

Let's write the implementation for the `/headers` route.

```javascript
server.get('/headers', (request, reply) => {
  reply.send(request.headers['content-type'] + ' ' + request.headers['my-header']);
});
```

If you run the test suite, you should see that the test is passing.
This is to quickly demostrate this Fastify behaviour. As you can see `content-type` has been "normalized", otherwise you would have to check for both `content-type` and `Content-Type`.

This happens also on custom defined headers. In this example `my-Header` has been normalized to `my-header`.
Fewer edge cases. Great stuff!

## Conclusions

We discussed the following:

- You can define routes in fastify using the object notation or the "http method function" notation.
- You can define dynamic routes in fastify by using the ":" token to define the route parameters. Those parameters are available in the `request.params` object.
- You can get the query parameters sent by the caller from the request object using `request.query`.
- You can get the headers sent by the caller from the request object using `request.headers`.
- You can run only a single test by using the `.only` function in conjunction with the `--test-only` command-line option.
