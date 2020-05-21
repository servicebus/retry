# @servicebus/retry
[![Build Status](https://travis-ci.org/servicebus/retry.svg?branch=master)](https://travis-ci.org/servicebus/retry)
[![codecov](https://codecov.io/gh/servicebus/retry/branch/master/graph/badge.svg)](https://codecov.io/gh/servicebus/retry)

Adds message acknowledge, reject, and retry capability to servicebus messages. A MemoryStore is available for testing and scenarios where processes to not exit/restart when a message is rejected. A RedisStore is available for multiprocess and crash-oriented design, where processes purposefully crash after a message reject and restart. 

### Configuration:

#### MemoryStore

```
const config = require('cconfig')();
const servicebus = require('servicebus');
const retry = require('@servicebus/retry');

const bus = servicebus.bus({
  url: config.RABBITMQ_URL
});

bus.use(retry({
  store: new retry.MemoryStore()
}));

module.exports = bus;
```

#### RedisStore

```
const config = require('cconfig')();
const servicebus = require('servicebus');
const retry = require('@servicebus/retry');

const bus = servicebus.bus({
  url: config.RABBITMQ_URL
});

bus.use(retry({
  store: new retry.RedisStore({
    host: config.REDIS.HOST,
    port: config.REDIS.PORT
  })
}));

module.exports = bus;
```

#### message.handle

@servicebus/retry causes incoming messages to have a .handle property with three available functions: `acknowledge(fn)`, `ack(fn)` (shorthand for acknowledge), and `reject(fn)`. The callback parameter in all methods is optional. 

### usage

```
bus.listen('queue.name', { ack: true /* making this queue and messages persistent */ }, function (msg) {
  msg.handle.ack(function () {
    console.log('acked message ' + msg.cid);
  });
});

bus.subscribe('routing.key2', { ack: true /* making this queue and messages persistent */ }, function (msg) {
  msg.handle.reject(function () {
    throw new Error('message ' + msg.cid + ' was rejected. let's crash and retry');
  });
});
```
