# Validation

Let's start by defining what validation means in the context of a web server.

We can define it like this: **Validation is the process of checking whether the data received from a client respects certain criteria. If the request does not meet these criteria, the server should reject it.**

Usually, when we talk about validation the concept of a schema is often mentioned. **A schema is a way to describe the shape of some data structure, moreover it's used to describe the criteria mentioned in the definition of validation.**

Fastify offers a way to define a schema for your routes both for defining the shape of the request and the response. The schema syntax chosen by the Fastify team is JSON Schema. This schemas are used to validate automatically the request and the response, returning a (400) error if the validation fails in case of malformed request. Meanwhile the response schema is used to efficiently serialize the response sent by the server returning only the properties defined in the schema (property stripping).

As usual let's start with a test.
Create a file called `validation.test.js` with the following content:

```javascript
import { test } from 'node:test';
import server from './validation.js';

test('validate a request', async (t) => {
  const responseOk = await server.inject({
    method: 'POST',
    url: '/user/new',
    payload: {
      email: 'test@example.com',
    },
  });

  t.assert.equal(responseOk.statusCode, 201);

  const responseError = await server.inject({
    method: 'POST',
    url: '/user/new',
    payload: {
      email: 'invalid-email',
    },
  });

  t.assert.equal(responseError.statusCode, 400);
});
```

As you can see we are calling the `inject` method two times, one with a valid payload and one with an invalid payload. We are expecting to receive status codes according to the validation result.

As usual let's run the test and see it fails `node --test ./validation.test.js`.

Let's now write the minimum amount of code to make the test pass.
Create a file called `validation.js` with the following content:

```javascript
import fastify from 'fastify';

const server = fastify();

server.post('/user/new', (request, reply) => {
  reply.send();
});

export default server;
```

if we re-run the tests we should see them fail with the following error:

```
✖ validate a request (16.766625ms)
  AssertionError [ERR_ASSERTION]: 200 == 400
      at assert.<computed> [as equal] (node:internal/test_runner/test:254:18)
      at TestContext.<anonymous> (file:///Users/jpm/Documents/personal/learn-fastify-with-test/code/validation/validation.test.js:23:12)
      at process.processTicksAndRejections (node:internal/process/task_queues:105:5)
      at async Test.run (node:internal/test_runner/test:981:9)
      at async startSubtestAfterBootstrap (node:internal/test_runner/harness:297:3) {
    generatedMessage: true,
    code: 'ERR_ASSERTION',
    actual: 200,
    expected: 400,
    operator: '=='
  }
```

As we can see the first assertion is passing but the second one is failing. It make sense. We aren't validating anything yet. Let's implement manually some validation logic, just enought to make this assertion pass.

```javascript
server.post('/user/new', (request, reply) => {
  if (request.body.email && request.body.email.includes('@')) {
    reply.send();
  } else {
    reply.status(400).send();
  }
});
```

This is not a good validation logic and it can be easily bypassed. Let's see how we can utilize JSON Schema validation alongside with Fastify to make this validation more robust.

```javascript
server.post(
  '/user/new',
  {
    schema: {
      body: {
        type: 'object',
        properties: {
          email: { type: 'string', format: 'email' },
        },
        required: ['email'],
      },
    },
  },
  (request, reply) => {
    reply.send();
  },
);
```

You can provide additional configuration to the route definition utilizing the second parameter of the route definition. Here you can define a schema that, in this case, will validate the request body. Here Fastify is doing some magic behind the scenes as it will automatically parse the request body as JSON and then checking if the shape of the result object matches the schema provided.

It's now clear from our tests that in case of a failed validation, Fastify will automatically send a response with a status code of 400.

We can run our tests again.
All green? Good!

## Validating other request parts

Fastify let's us validate route parameters, query parameters and headers using the same approach. You can do so by providing the relative schema in the route definition.

Like this:

```javascript

server.post(
  '/validate',
  {
    schema: {
      body: {
        type: 'object',
        properties: {
          email: { type: 'string', format: 'email' },
        },
      },
      headers: {
        type: 'object',
        properties: {
          'x-special-header': { type: 'string', format: 'email' },
        },
      },
      cookies: {
        type: 'object',
        properties: {
          'x-special-cookie': { type: 'string', format: 'email' },
        },
      },
    },
  },
  (request, reply) => {
    reply.send();
  },
);
```
