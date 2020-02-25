# Bayeuxjs

Bayeuxjs is a copy of the Faye tools with backpressure functionality. It allows you to control the flow of messages by
giving the user control over the sending of `/meta/connect` messages through the use of a callback.

In the following example, the `afterLastMessage` callback will only be called after the last message in a
`/meta/connect` message is processed. Messages received in a `/meta/connect` message, are processed in the `onMessage`
callback. After receiving the first message with `first: true`, the sending of a `/meta/connect` message is delayed by 5
seconds. You'll notice that the output of the second message is always at least 5 seconds apart from the first one.

```js
const bayeuxjs = require("bayeuxjs");
const http = require("http");

// Server setup
const adapter = new bayeuxjs.NodeAdapter({mount: '/'});
const server = http.createServer();

adapter.attach(server);
server.listen(8000);

// Subscriber setup
const subscriber = new bayeuxjs.Client('http://localhost:8000/');
subscriber.disable("websocket");

let receivedFirstMessage = false;

subscriber.subscribeWithLazyConnect(
    '/messages',
    function onMessage(message) {
        console.log(`[${(new Date).toISOString()}] Message received: ${message.text}`)
        receivedFirstMessage = message.first;
    },
    function afterLastMessage(lazyConnectCallback) {
        setTimeout(lazyConnectCallback, receivedFirstMessage ? 5000 : 0);
    }
);

// Publishing messages
const publisher = new bayeuxjs.Client('http://localhost:8000/');
publisher.disable("websocket");

publisher.publish('/messages', {
    first: true,
    text: 'First Message'
});

setTimeout(async () => {
    await publisher.publish('/messages', {
        text: 'Secong Message after delayed /meta/connect'
    });

    publisher.disconnect();
}, 1000);
```

- Documentation: http://faye.jcoglan.com
- Mailing list: http://groups.google.com/group/faye-users
- Bug tracker: http://github.com/dhulke/bayeuxjs/issues
- Source code: http://github.com/dhulke/bayeuxjs
- Faye code: http://github.com/faye/faye
