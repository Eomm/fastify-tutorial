# Logging

Let's see how fastify makes doing logging right, a walk in the park.

As usual, let's write a test;
create a file name `logging.test.js` with the following content:

```javascript
import { test } from 'node:test';
import server from './logging.js';

test('should log a message', () => {
  server.log.info('test message');
  server.log.error('test error');
  server.log.warn('test warning');
  server.log.debug('test debug');
});
```

Now if we run the test `node --test ./logging.test.js` we should see it fails since `logging.js` doesn't exist.
Let's create it with the "usual" content:

```javascript
import fastify from 'fastify';

const server = fastify();

export default server;
```

Now we can re-run the test and see them pass:

```
✔ should log a message (0.442917ms)
ℹ tests 1
ℹ suites 0
ℹ pass 1
ℹ fail 0
ℹ cancelled 0
ℹ skipped 0
ℹ todo 0
ℹ duration_ms 154.765042
```

But wait, where are our logs?
By default, Fastify is silent. It means that we need to explicitly tell fastify that we want to see logs.
To do so, we have to modify the `logging.js` file as follows:

```javascript
import fastify from 'fastify';

const server = fastify({
  logger: true
});

export default server;
```

If we re-run the test using `node --test ./logging.test.js`, we should see something similar to this:
```
{"level":30,"time":1739724656505,"pid":15435,"hostname":"happy-hippo-123","msg":"test message"}
✔ should log a message (0.7175ms)
{"level":50,"time":1739724656505,"pid":15435,"hostname":"happy-hippo-123","msg":"test error"}
{"level":40,"time":1739724656505,"pid":15435,"hostname":"happy-hippo-123","msg":"test warning"}
ℹ tests 1
ℹ suites 0
ℹ pass 1
ℹ fail 0
ℹ cancelled 0
ℹ skipped 0
ℹ todo 0
ℹ duration_ms 167.8325
```

Beautiful! But wait, what are those logs? We didn't ask for a `time` property or a `pid` one. We didn't ask for a `level` property either. We just specified a string. On top of that those logs looks like JSON serializable objects.
Let's see what Fastify is doing here.

## A good logging system
There are several features that make a logging system good, I'll mention a few of them:
First the system should emit logs that are parsable by both human and a machine. A JSON representation is a good choice. It's easy to parse and it's easy to read. We want our logs to be searchable and categorizable. Speaking of categorization a good logging system should also be able to categorize logs by severity level. This tells whom the log is intended for, if they are meant to be reported or act on them or if they are meant to be just stored. This is what Pino, Fastify's logger, does by default. Good stuff!

Wait, but isn't this course called "Learn Fastify with Tests"?
You are correct! Let's step up a notch and test some logs!

## Factory function
The thing is, how can we do that? It would be nice to intercept the log output and assert on it.
Unfortunately, at the moment we can't intercept the log output. Let's refactor our test and our implementation to enable such thing.

Edit `logging.js` such that it exports a function that returns a fastify instance with a custom logger:
```javascript
import fastify from 'fastify';

export default function makeServer() {
  return fastify({
    logger: true,
  });
}
```

Edit `logging.test.js` to utilize such function:
```javascript
import { test } from 'node:test';
import makeServer from './logging.js';

const server = makeServer();

test('should log a message', () => {
  server.log.info('test message');
  server.log.error('test error');
  server.log.warn('test warning');
  server.log.debug('test debug');
});
```

If we re-run the test using `node --test ./logging.test.js` we should see the previous output.
Now the difference is that we are instantiating a fastify instance inside the test file.
We can now move further. Pino, Fastify's logger has a concept of a `stream` that is used to deliver the log messages to a destination. Usually that destination is the console output but we can also use a file or a database.
We now have to provide such stream to the logger and assert on it's output using a mock.
As usual we write the tests first, but maybe a little introduction to mocking is needed.

## Mocks
As the word suggests, mocking is a technique used in testing to replace real objects with fake ones. This allows us to isolate the code we are testing and control its behavior. In our case, we want to mock this "stream" so that we can intercept its output and assert on it. The internal node testing library provides a `mock` utility that can be used to create a mock of a function. Do not worry we are going to cover mocks deeply in this course, you are going to have plenty of time to familiarize with the concept.

Let's create such "stream" that is going to utilize a mock function:

```javascript
import { test, mock } from 'node:test';

//...

const logger = {
  stream: {
    write: mock.fn(),
  },
};

const server = makeServer({
  logger,
});

// ...
```

Let's brake it down. First we are importing the `mock` utility from the `node:test` module. Then we create a `logger` object with a `stream` property that has a `write` method. If this seem a little arbitrary, take a look at the Fastify and Pino documentation regarding this part. Basically Pino expects a stream and every stream has to have a `write` function. That function is then called each time there is a log message to be delivered. Our assertion strategy is based on the fact that we can assert on the arguments passed to the `write` function, let's do it:

```javascript
test('should log a message', (t) => {
  server.log.info('test message');

  t.assert.ok(
    !!logger.stream.write.mock.calls.find(
      (call) => call.arguments[0].includes('test message'),
    ),
    'Should log "test message"',
  );
});
```

wow, that's a small mess. Let's break down the assertion.

First we retrieve all the calls made by the code during the test to our mocked function.
Then we find if one of those calls has as a first argument, a string (remember that the logs are JSON like strings) that contains the message we are looking for. If we find it we are happy and the assertion passes. Otherwise it fails.

If we run now the test (`node --test ./logging.test.js`) we should see the following output:

```
✔ should log a message (0.871125ms)
ℹ tests 1
ℹ suites 0
ℹ pass 1
ℹ fail 0
ℹ cancelled 0
ℹ skipped 0
ℹ todo 0
ℹ duration_ms 162.858958
```

All good. Now we can start refactoring our test. First creating a helper function that will make the assertion for us:

```javascript
function assertLoggedMessage(t, message) {
  t.assert.ok(
    !!logger.stream.write.mock.calls.find((call) =>
      call.arguments[0].includes(message),
    ),
    `Should log "${message}"`,
  );
}
```

If we replace the assertion with our helper function we get:

```javascript
test('should log a message', (t) => {
  server.log.info('test message');
  assertLoggedMessage(t, 'test message');
});
```

Running again the tests we should see them pass.
Great stuff!

## Request Logging
By default Fastify logs all the requests and responses. Let's try to test that a call has been made by asserting on that behavior. Create another test:

```javascript
test('should log the request', async (t) => {
  await server.inject({
    method: 'GET',
    url: '/hello',
  });

  assertLoggedMessage(t, '/hello');
});
```

I admit, this is not very useful. Let's introduce a requirement on the `/hello` route. First it has to exists. Second, if there is a query parameter `name` it should log a message `Fastify says hello to` followed by the name.
As usual we implements the test:

```javascript
test('should say hello to Jhon', async (t) => {
  await server.inject({
    method: 'GET',
    url: '/hello',
    query: {
      name: 'John',
    },
  });

  assertLoggedMessage(t, 'Fastify says hello to John');
});
```

After running the tests we should see the following error:

```
✖ should log the request (12.512541ms)
  AssertionError [ERR_ASSERTION]: Should log "Fastify says hello to John"
      at assertLoggedMessage (file:///Users/jpm/Documents/personal/learn-fastify-with-test/code/logging/logging.test.js:32:12)
      at TestContext.<anonymous> (file:///Users/jpm/Documents/personal/learn-fastify-with-test/code/logging/logging.test.js:28:3)
      at process.processTicksAndRejections (node:internal/process/task_queues:105:5)
      at async Test.run (node:internal/test_runner/test:981:9)
      at async Test.processPendingSubtests (node:internal/test_runner/test:678:7) {
    generatedMessage: false,
    code: 'ERR_ASSERTION',
    actual: false,
    expected: true,
    operator: '=='
  }
```

Let's make this test pass with the minimum amount of code:

```javascript
import fastify from 'fastify';

export default function makeServer({ logger }) {
  const server = fastify({
    logger,
  });

  server.get('/hello', (request, reply) => {
    server.log.info('Fastify says hello to John');
    reply.send();
  });

  return server;
}
```

This code is of course not complete since the `name` query parameter is not handled.
First we need to create tests that fails and then we iterate over and make them pass.
Let's say hello to Maria.

```javascript
test('should say hello to Maria', async (t) => {
  await server.inject({
    method: 'GET',
    url: '/hello',
    query: {
      name: 'Maria',
    },
  });

  assertLoggedMessage(t, 'Fastify says hello to Maria');
});
```

See the test fails and then let's write an implementation that handles the `name` query parameter:

```javascript
server.get('/hello', (request, reply) => {
  const { name } = request.query;
  server.log.info(`Fastify says hello to ${name}`);
  reply.send();
});
```

I hope your tests are green as mine are. I'm excited to see your progress!
Let's implement the last functionality. If no name is provided we should not salute anyone.
To do that we introduce a new test:

```javascript
test('should not say hello', async (t) => {
  await server.inject({
    method: 'GET',
    url: '/hello',
  });

  t.assert.ok(
    !logger.stream.write.mock.calls.find((call) =>
      call.arguments[0].includes('Fastify says hello'),
    ),
    `Unexpected log "Fastify says hello"`,
  );
});
```

As usual we need to verify that the test isn't passing.
After running the tests, we should see the following:

```
✖ should not say hello (0.768667ms)
  AssertionError [ERR_ASSERTION]: Unexpected log "Fastify says hello"
      at TestContext.<anonymous> (file:///Users/jpm/Documents/personal/learn-fastify-with-test/code/logging/logging.test.js:49:12)
      at process.processTicksAndRejections (node:internal/process/task_queues:105:5)
      at async Test.run (node:internal/test_runner/test:981:9)
      at async Test.processPendingSubtests (node:internal/test_runner/test:678:7) {
    generatedMessage: false,
    code: 'ERR_ASSERTION',
    actual: false,
    expected: true,
    operator: '=='
  }
```

As usual let's make the test pass:

```javascript
server.get('/hello', (request, reply) => {
  const { name } = request.query;
  if (name) {
    server.log.info(`Fastify says hello to ${name}`);
  }
  reply.send();
});
```

Let's re-run our tests now.
Drum roll... and failure.
Why is that?

## Mocking Pt.2
Our mocks have memory. They remember everything that happens to them. Otherwise we wouldn't be able to assert on them. The problem is that that memory is kept across tests. If we comment all the other tests and we re-run the suite we will see that the test pass. How to address this issue?

Mocks have the ability to reset their state through a method invocation. This method is called `resetCalls()`. Let's update our test to reset our mocks:

```javascript
test('should not say hello', async (t) => {

  logger.stream.write.mock.resetCalls();

  await server.inject({
    method: 'GET',
    url: '/hello',
  });

  t.assert.ok(
    !logger.stream.write.mock.calls.find((call) =>
      call.arguments[0].includes('Fastify says hello'),
    ),
    `Unexpected log "Fastify says hello"`,
  );
});
```

We can now see that the tests are passing.

## Test lifecycle
Having to call `resetCalls` manually on each test is tedious. We can utilize something called lifecycle hooks. These hooks are called before and after each test. We can use them to reset the mocks before each test. Like so:

```javascript
import { test, mock, beforeEach } from 'node:test';

// ...

beforeEach(() => {
  logger.stream.write.mock.resetCalls();
});

// ...
```

We can remove `logger.stream.write.mock.resetCalls();` from the `should not say hello` test.
Re running the test we should see this output:

```
✔ should log a message (0.99825ms)
✔ should say hello to Jhon (11.151834ms)
✔ should say hello to Maria (0.752791ms)
✔ should not say hello (0.52075ms)
ℹ tests 4
ℹ suites 0
ℹ pass 4
ℹ fail 0
ℹ cancelled 0
ℹ skipped 0
ℹ todo 0
ℹ duration_ms 162.483459
```

Happy days!

## BONUS: Refactoring
There are improvements that can be made to our tests.
First we should move all the logger related utilities into a separate file, like so:

```javascript
import { mock, beforeEach } from 'node:test';

export function makeMockLogger() {
  const logger = {
    stream: {
      write: mock.fn(),
    },
  };

  beforeEach(() => {
    logger.stream.write.mock.resetCalls();
  });

  function findLoggedMessage(message) {
    return logger.stream.write.mock.calls.find((call) =>
      call.arguments[0].includes(message),
    );
  }

  return {
    logger,
    assert(t, message) {
      t.assert.ok(!!findLoggedMessage(message), `Should log "${message}"`);
    },
    assertMissing(t, message) {
      t.assert.ok(!findLoggedMessage(message), `Unexpected log "${message}"`);
    },
  };
}
```

In this way we can easily co-locate related assertions functions and mocks.
One thing to notice is the use of lifecycle hooks `beforeEach` in the factory function itself.
This can be a controversial pattern but I like keeping everything in one place, making the right thing to do the default behaviour.

## Conclusions

In this chapter we have learned that:
- Using a factory function to create the fastify instance is a good practice, it lets us configure easily the servers behavior.
- Fastify logs each request.
- We can create mocks to test behaviour that we can't directly assert on.
- Mocks have memory and they need to be reset accordingly to avoid unexpected behavior.
