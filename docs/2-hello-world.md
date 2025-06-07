# Hello World

Let's start with a simple "Hello World" example.
We are planning to cover the following topics:

- Creating a new Fastify instance
- Adding a route
- Testing that the route works

We start by writing a test first; create a file called `hello-world.test.js` with the following content:

```javascript
import { test } from 'node:test';
import server from './hello-world.js';

test('hello world', async (t) => {
  const response = await server.inject({
    method: 'GET',
    url: '/hello',
  });

  t.assert.equal(response.statusCode, 200);
  t.assert.equal(response.payload, 'Hello World!');
});
```

Let's break it down:
we are importing a server instance from the file `hello-world.js`, then we are calling the `inject` method on the server instance, providing some information about the request we want to make. Then we are asserting that the response we receive has a status code of 200 and a payload of 'Hello World!'.

## First Solution

As we can see from the test we are expecting that the file `1.js` has a default export. That default export is an object with an `inject` method that when called it returns an object with a `statusCode` property and a `payload` property. Let's write enough code to make the test pass.

```javascript
const server = {
  inject: async () => {
    return {
      statusCode: 200,
      payload: 'Hello World!',
    };
  },
};

export default server;
```

if we run `node --test` we should see the following output:

```
✔ hello world (0.581583ms)
ℹ tests 1
ℹ suites 0
ℹ pass 1
ℹ fail 0
ℹ cancelled 0
ℹ skipped 0
ℹ todo 0
ℹ duration_ms 60.927416
```

All good, now we need to include a real fastify instance (this book is called "Learn Fastify with Test" for a reason) iterating over this test until we have a working solution.

## Creating a Fastify Instance

To create a fastify instance we need to import the `fastify` module. Before importing it we need to install it using npm. Run `npm install fastify`.

After that we can modify the content of `hello-world.js` such that we export the fastify instance we are creating.

```javascript
import fastify from 'fastify';

const server = fastify();

export default server;
```

if we re-run the test we see the following output:

```
✖ hello world (13.837208ms)
ℹ tests 1
ℹ suites 0
ℹ pass 0
ℹ fail 1
ℹ cancelled 0
ℹ skipped 0
ℹ todo 0
ℹ duration_ms 193.418958

✖ failing tests:

test at code/hello-world/hello-world.test.js:4:1
✖ hello world (13.837208ms)
  AssertionError [ERR_ASSERTION]: 404 == 200
      at assert.<computed> [as equal] (node:internal/test_runner/test:254:18)
      at TestContext.<anonymous> (file:///Users/jpm/Documents/personal/learn-fastify-with-test/code/hello-world/hello-world.test.js:10:12)
      at process.processTicksAndRejections (node:internal/process/task_queues:105:5)
      at async Test.run (node:internal/test_runner/test:981:9)
      at async startSubtestAfterBootstrap (node:internal/test_runner/harness:297:3) {
    generatedMessage: true,
    code: 'ERR_ASSERTION',
    actual: 404,
    expected: 200,
    operator: '=='
  }
```

makes sense since we are not declaring any routes. By default Fastify will respond with a 404 status code for any request that does not match a declared route. This behavior can be customized by registering a route handler for the `404` status code but we will see how further along in this tutorial.

So let's create a route `GET /hello`;
edit `hello-world.js` like so:

```javascript
const server = fastify();

server.get('/hello', async (request, reply) => {
  reply.send('Hello World!');
});

export default server;
```

if we re-run `node --test` we should see a passing test.
Let's explore further what we have done so far.

## Separating Fastify creation from server startup

As we can deduce from this little example, separating the creation of the Fastify instance from the server startup allows us to easily test our routes without having to handle server startup and related concerns. It's a good hygiene practice and it can be done on a module import level like we did in this example or by using a factory function. The second approach is the one recommended by Fastify maintainers.

## Injecting Requests

Fastify offers a convenient way to test your server without having to start a server on a specific port. This is achieved by using the `inject` method provided by Fastify. The `inject` method internally uses a package called `light-my-request`. This package creates a mocked request that is then passed to the server as it would be passed to an actual running instance. This helpful abstraction allows us to write fast and reliable tests.
