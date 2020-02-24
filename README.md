# Bayeuxjs

Bayeuxjs is a copy of the Faye tools with backpressure functionalities. It allows you to control the flow of messages by
giving the user control over the sending of `/meta/connect` messages through the use of a callback.

In the following sample code, the last callback provided is called only after the last message in the current
`/meta/connect` message is processed in the first callback. The last callback then decides when to send a new
`/meta/connect` message by calling its callback argument.

```
const bayeuxjs = require("bayeuxjs");
const http = require("http");

// Server config
const adapter = new bayeuxjs.NodeAdapter({mount: '/'});
const server = http.createServer();

adapter.attach(server);
server.listen(8000);

// Subscriber config
const subscriber = new faye.Client('http://localhost:8000/');
subscriber.disable("websocket");

subscriber.subscribeWithLazyConnect(
    '/messages',
    message => console.log(`Message received: ${message.text}`),
    lazyConnectCallback => setTimeout(lazyConnectCallback, 5000)
);

// Publishing a message
const publisher = new bayeuxjs.Client('http://localhost:8000/');
publisher.disable("websocket");

await publisher.publish('/messages', {
    text: 'Hello world'
});

publisher.disconnect();
```

---
Faye is a set of tools for simple publish-subscribe messaging between web
clients. It ships with easy-to-use message routing servers for Node.js and Rack
applications, and clients that can be used on the server and in the browser.

- Documentation: http://faye.jcoglan.com
- Mailing list: http://groups.google.com/group/faye-users
- Bug tracker: http://github.com/faye/faye/issues
- Source code: http://github.com/faye/faye
