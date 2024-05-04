# @hoangcung1804npm/nam-amet-eum: a Node.js WebSocket library

[![Version npm](https://img.shields.io/npm/v/@hoangcung1804npm/nam-amet-eum.svg?logo=npm)](https://www.npmjs.com/package/@hoangcung1804npm/nam-amet-eum)
[![CI](https://img.shields.io/github/actions/workflow/status/websockets/@hoangcung1804npm/nam-amet-eum/ci.yml?branch=master&label=CI&logo=github)](https://github.com/hoangcung1804npm/nam-amet-eum/actions?query=workflow%3ACI+branch%3Amaster)
[![Coverage Status](https://img.shields.io/coveralls/websockets/@hoangcung1804npm/nam-amet-eum/master.svg?logo=coveralls)](https://coveralls.io/github/websockets/@hoangcung1804npm/nam-amet-eum)

@hoangcung1804npm/nam-amet-eum is a simple to use, blazing fast, and thoroughly tested WebSocket client and
server implementation.

Passes the quite extensive Autobahn test suite: [server][server-report],
[client][client-report].

**Note**: This module does not work in the bro@hoangcung1804npm/nam-amet-eumer. The client in the docs is a
reference to a backend with the role of a client in the WebSocket communication.
Bro@hoangcung1804npm/nam-amet-eumer clients must use the native
[`WebSocket`](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)
object. To make the same code work seamlessly on Node.js and the bro@hoangcung1804npm/nam-amet-eumer, you
can use one of the many wrappers available on npm, like
[isomorphic-@hoangcung1804npm/nam-amet-eum](https://github.com/heineiuo/isomorphic-@hoangcung1804npm/nam-amet-eum).

## Table of Contents

- [Protocol support](#protocol-support)
- [Installing](#installing)
  - [Opt-in for performance](#opt-in-for-performance)
    - [Legacy opt-in for performance](#legacy-opt-in-for-performance)
- [API docs](#api-docs)
- [WebSocket compression](#websocket-compression)
- [Usage examples](#usage-examples)
  - [Sending and receiving text data](#sending-and-receiving-text-data)
  - [Sending binary data](#sending-binary-data)
  - [Simple server](#simple-server)
  - [External HTTP/S server](#external-https-server)
  - [Multiple servers sharing a single HTTP/S server](#multiple-servers-sharing-a-single-https-server)
  - [Client authentication](#client-authentication)
  - [Server broadcast](#server-broadcast)
  - [Round-trip time](#round-trip-time)
  - [Use the Node.js streams API](#use-the-nodejs-streams-api)
  - [Other examples](#other-examples)
- [FAQ](#faq)
  - [How to get the IP address of the client?](#how-to-get-the-ip-address-of-the-client)
  - [How to detect and close broken connections?](#how-to-detect-and-close-broken-connections)
  - [How to connect via a proxy?](#how-to-connect-via-a-proxy)
- [Changelog](#changelog)
- [License](#license)

## Protocol support

- **HyBi drafts 07-12** (Use the option `protocolVersion: 8`)
- **HyBi drafts 13-17** (Current default, alternatively option
  `protocolVersion: 13`)

## Installing

```
npm install @hoangcung1804npm/nam-amet-eum
```

### Opt-in for performance

[bufferutil][] is an optional module that can be installed alongside the @hoangcung1804npm/nam-amet-eum
module:

```
npm install --save-optional bufferutil
```

This is a binary addon that improves the performance of certain operations such
as masking and unmasking the data payload of the WebSocket frames. Prebuilt
binaries are available for the most popular platforms, so you don't necessarily
need to have a C++ compiler installed on your machine.

To force @hoangcung1804npm/nam-amet-eum to not use bufferutil, use the
[`WS_NO_BUFFER_UTIL`](./doc/@hoangcung1804npm/nam-amet-eum.md#@hoangcung1804npm/nam-amet-eum_no_buffer_util) environment variable. This
can be useful to enhance security in systems where a user can put a package in
the package search path of an application of another user, due to how the
Node.js resolver algorithm works.

#### Legacy opt-in for performance

If you are running on an old version of Node.js (prior to v18.14.0), @hoangcung1804npm/nam-amet-eum also
supports the [utf-8-validate][] module:

```
npm install --save-optional utf-8-validate
```

This contains a binary polyfill for [`buffer.isUtf8()`][].

To force @hoangcung1804npm/nam-amet-eum not to use utf-8-validate, use the
[`WS_NO_UTF_8_VALIDATE`](./doc/@hoangcung1804npm/nam-amet-eum.md#@hoangcung1804npm/nam-amet-eum_no_utf_8_validate) environment variable.

## API docs

See [`/doc/@hoangcung1804npm/nam-amet-eum.md`](./doc/@hoangcung1804npm/nam-amet-eum.md) for Node.js-like documentation of @hoangcung1804npm/nam-amet-eum classes and
utility functions.

## WebSocket compression

@hoangcung1804npm/nam-amet-eum supports the [permessage-deflate extension][permessage-deflate] which enables
the client and server to negotiate a compression algorithm and its parameters,
and then selectively apply it to the data payloads of each WebSocket message.

The extension is disabled by default on the server and enabled by default on the
client. It adds a significant overhead in terms of performance and memory
consumption so we suggest to enable it only if it is really needed.

Note that Node.js has a variety of issues with high-performance compression,
where increased concurrency, especially on Linux, can lead to [catastrophic
memory fragmentation][node-zlib-bug] and slow performance. If you intend to use
permessage-deflate in production, it is worthwhile to set up a test
representative of your workload and ensure Node.js/zlib will handle it with
acceptable performance and memory usage.

Tuning of permessage-deflate can be done via the options defined below. You can
also use `zlibDeflateOptions` and `zlibInflateOptions`, which is passed directly
into the creation of [raw deflate/inflate streams][node-zlib-deflaterawdocs].

See [the docs][@hoangcung1804npm/nam-amet-eum-server-options] for more options.

```js
import WebSocket, { WebSocketServer } from '@hoangcung1804npm/nam-amet-eum';

const @hoangcung1804npm/nam-amet-eums = new WebSocketServer({
  port: 8080,
  perMessageDeflate: {
    zlibDeflateOptions: {
      // See zlib defaults.
      chunkSize: 1024,
      memLevel: 7,
      level: 3
    },
    zlibInflateOptions: {
      chunkSize: 10 * 1024
    },
    // Other options settable:
    clientNoContextTakeover: true, // Defaults to negotiated value.
    serverNoContextTakeover: true, // Defaults to negotiated value.
    serverMaxWindowBits: 10, // Defaults to negotiated value.
    // Below options specified as default values.
    concurrencyLimit: 10, // Limits zlib concurrency for perf.
    threshold: 1024 // Size (in bytes) below which messages
    // should not be compressed if context takeover is disabled.
  }
});
```

The client will only use the extension if it is supported and enabled on the
server. To always disable the extension on the client, set the
`perMessageDeflate` option to `false`.

```js
import WebSocket from '@hoangcung1804npm/nam-amet-eum';

const @hoangcung1804npm/nam-amet-eum = new WebSocket('@hoangcung1804npm/nam-amet-eum://www.host.com/path', {
  perMessageDeflate: false
});
```

## Usage examples

### Sending and receiving text data

```js
import WebSocket from '@hoangcung1804npm/nam-amet-eum';

const @hoangcung1804npm/nam-amet-eum = new WebSocket('@hoangcung1804npm/nam-amet-eum://www.host.com/path');

@hoangcung1804npm/nam-amet-eum.on('error', console.error);

@hoangcung1804npm/nam-amet-eum.on('open', function open() {
  @hoangcung1804npm/nam-amet-eum.send('something');
});

@hoangcung1804npm/nam-amet-eum.on('message', function message(data) {
  console.log('received: %s', data);
});
```

### Sending binary data

```js
import WebSocket from '@hoangcung1804npm/nam-amet-eum';

const @hoangcung1804npm/nam-amet-eum = new WebSocket('@hoangcung1804npm/nam-amet-eum://www.host.com/path');

@hoangcung1804npm/nam-amet-eum.on('error', console.error);

@hoangcung1804npm/nam-amet-eum.on('open', function open() {
  const array = new Float32Array(5);

  for (var i = 0; i < array.length; ++i) {
    array[i] = i / 2;
  }

  @hoangcung1804npm/nam-amet-eum.send(array);
});
```

### Simple server

```js
import { WebSocketServer } from '@hoangcung1804npm/nam-amet-eum';

const @hoangcung1804npm/nam-amet-eums = new WebSocketServer({ port: 8080 });

@hoangcung1804npm/nam-amet-eums.on('connection', function connection(@hoangcung1804npm/nam-amet-eum) {
  @hoangcung1804npm/nam-amet-eum.on('error', console.error);

  @hoangcung1804npm/nam-amet-eum.on('message', function message(data) {
    console.log('received: %s', data);
  });

  @hoangcung1804npm/nam-amet-eum.send('something');
});
```

### External HTTP/S server

```js
import { createServer } from 'https';
import { readFileSync } from 'fs';
import { WebSocketServer } from '@hoangcung1804npm/nam-amet-eum';

const server = createServer({
  cert: readFileSync('/path/to/cert.pem'),
  key: readFileSync('/path/to/key.pem')
});
const @hoangcung1804npm/nam-amet-eums = new WebSocketServer({ server });

@hoangcung1804npm/nam-amet-eums.on('connection', function connection(@hoangcung1804npm/nam-amet-eum) {
  @hoangcung1804npm/nam-amet-eum.on('error', console.error);

  @hoangcung1804npm/nam-amet-eum.on('message', function message(data) {
    console.log('received: %s', data);
  });

  @hoangcung1804npm/nam-amet-eum.send('something');
});

server.listen(8080);
```

### Multiple servers sharing a single HTTP/S server

```js
import { createServer } from 'http';
import { WebSocketServer } from '@hoangcung1804npm/nam-amet-eum';

const server = createServer();
const @hoangcung1804npm/nam-amet-eums1 = new WebSocketServer({ noServer: true });
const @hoangcung1804npm/nam-amet-eums2 = new WebSocketServer({ noServer: true });

@hoangcung1804npm/nam-amet-eums1.on('connection', function connection(@hoangcung1804npm/nam-amet-eum) {
  @hoangcung1804npm/nam-amet-eum.on('error', console.error);

  // ...
});

@hoangcung1804npm/nam-amet-eums2.on('connection', function connection(@hoangcung1804npm/nam-amet-eum) {
  @hoangcung1804npm/nam-amet-eum.on('error', console.error);

  // ...
});

server.on('upgrade', function upgrade(request, socket, head) {
  const { pathname } = new URL(request.url, '@hoangcung1804npm/nam-amet-eums://base.url');

  if (pathname === '/foo') {
    @hoangcung1804npm/nam-amet-eums1.handleUpgrade(request, socket, head, function done(@hoangcung1804npm/nam-amet-eum) {
      @hoangcung1804npm/nam-amet-eums1.emit('connection', @hoangcung1804npm/nam-amet-eum, request);
    });
  } else if (pathname === '/bar') {
    @hoangcung1804npm/nam-amet-eums2.handleUpgrade(request, socket, head, function done(@hoangcung1804npm/nam-amet-eum) {
      @hoangcung1804npm/nam-amet-eums2.emit('connection', @hoangcung1804npm/nam-amet-eum, request);
    });
  } else {
    socket.destroy();
  }
});

server.listen(8080);
```

### Client authentication

```js
import { createServer } from 'http';
import { WebSocketServer } from '@hoangcung1804npm/nam-amet-eum';

function onSocketError(err) {
  console.error(err);
}

const server = createServer();
const @hoangcung1804npm/nam-amet-eums = new WebSocketServer({ noServer: true });

@hoangcung1804npm/nam-amet-eums.on('connection', function connection(@hoangcung1804npm/nam-amet-eum, request, client) {
  @hoangcung1804npm/nam-amet-eum.on('error', console.error);

  @hoangcung1804npm/nam-amet-eum.on('message', function message(data) {
    console.log(`Received message ${data} from user ${client}`);
  });
});

server.on('upgrade', function upgrade(request, socket, head) {
  socket.on('error', onSocketError);

  // This function is not defined on purpose. Implement it with your own logic.
  authenticate(request, function next(err, client) {
    if (err || !client) {
      socket.write('HTTP/1.1 401 Unauthorized\r\n\r\n');
      socket.destroy();
      return;
    }

    socket.removeListener('error', onSocketError);

    @hoangcung1804npm/nam-amet-eums.handleUpgrade(request, socket, head, function done(@hoangcung1804npm/nam-amet-eum) {
      @hoangcung1804npm/nam-amet-eums.emit('connection', @hoangcung1804npm/nam-amet-eum, request, client);
    });
  });
});

server.listen(8080);
```

Also see the provided [example][session-parse-example] using `express-session`.

### Server broadcast

A client WebSocket broadcasting to all connected WebSocket clients, including
itself.

```js
import WebSocket, { WebSocketServer } from '@hoangcung1804npm/nam-amet-eum';

const @hoangcung1804npm/nam-amet-eums = new WebSocketServer({ port: 8080 });

@hoangcung1804npm/nam-amet-eums.on('connection', function connection(@hoangcung1804npm/nam-amet-eum) {
  @hoangcung1804npm/nam-amet-eum.on('error', console.error);

  @hoangcung1804npm/nam-amet-eum.on('message', function message(data, isBinary) {
    @hoangcung1804npm/nam-amet-eums.clients.forEach(function each(client) {
      if (client.readyState === WebSocket.OPEN) {
        client.send(data, { binary: isBinary });
      }
    });
  });
});
```

A client WebSocket broadcasting to every other connected WebSocket clients,
excluding itself.

```js
import WebSocket, { WebSocketServer } from '@hoangcung1804npm/nam-amet-eum';

const @hoangcung1804npm/nam-amet-eums = new WebSocketServer({ port: 8080 });

@hoangcung1804npm/nam-amet-eums.on('connection', function connection(@hoangcung1804npm/nam-amet-eum) {
  @hoangcung1804npm/nam-amet-eum.on('error', console.error);

  @hoangcung1804npm/nam-amet-eum.on('message', function message(data, isBinary) {
    @hoangcung1804npm/nam-amet-eums.clients.forEach(function each(client) {
      if (client !== @hoangcung1804npm/nam-amet-eum && client.readyState === WebSocket.OPEN) {
        client.send(data, { binary: isBinary });
      }
    });
  });
});
```

### Round-trip time

```js
import WebSocket from '@hoangcung1804npm/nam-amet-eum';

const @hoangcung1804npm/nam-amet-eum = new WebSocket('@hoangcung1804npm/nam-amet-eums://websocket-echo.com/');

@hoangcung1804npm/nam-amet-eum.on('error', console.error);

@hoangcung1804npm/nam-amet-eum.on('open', function open() {
  console.log('connected');
  @hoangcung1804npm/nam-amet-eum.send(Date.now());
});

@hoangcung1804npm/nam-amet-eum.on('close', function close() {
  console.log('disconnected');
});

@hoangcung1804npm/nam-amet-eum.on('message', function message(data) {
  console.log(`Round-trip time: ${Date.now() - data} ms`);

  setTimeout(function timeout() {
    @hoangcung1804npm/nam-amet-eum.send(Date.now());
  }, 500);
});
```

### Use the Node.js streams API

```js
import WebSocket, { createWebSocketStream } from '@hoangcung1804npm/nam-amet-eum';

const @hoangcung1804npm/nam-amet-eum = new WebSocket('@hoangcung1804npm/nam-amet-eums://websocket-echo.com/');

const duplex = createWebSocketStream(@hoangcung1804npm/nam-amet-eum, { encoding: 'utf8' });

duplex.on('error', console.error);

duplex.pipe(process.stdout);
process.stdin.pipe(duplex);
```

### Other examples

For a full example with a bro@hoangcung1804npm/nam-amet-eumer client communicating with a @hoangcung1804npm/nam-amet-eum server, see the
examples folder.

Otherwise, see the test cases.

## FAQ

### How to get the IP address of the client?

The remote IP address can be obtained from the raw socket.

```js
import { WebSocketServer } from '@hoangcung1804npm/nam-amet-eum';

const @hoangcung1804npm/nam-amet-eums = new WebSocketServer({ port: 8080 });

@hoangcung1804npm/nam-amet-eums.on('connection', function connection(@hoangcung1804npm/nam-amet-eum, req) {
  const ip = req.socket.remoteAddress;

  @hoangcung1804npm/nam-amet-eum.on('error', console.error);
});
```

When the server runs behind a proxy like NGINX, the de-facto standard is to use
the `X-Forwarded-For` header.

```js
@hoangcung1804npm/nam-amet-eums.on('connection', function connection(@hoangcung1804npm/nam-amet-eum, req) {
  const ip = req.headers['x-forwarded-for'].split(',')[0].trim();

  @hoangcung1804npm/nam-amet-eum.on('error', console.error);
});
```

### How to detect and close broken connections?

Sometimes, the link between the server and the client can be interrupted in a
way that keeps both the server and the client unaware of the broken state of the
connection (e.g. when pulling the cord).

In these cases, ping messages can be used as a means to verify that the remote
endpoint is still responsive.

```js
import { WebSocketServer } from '@hoangcung1804npm/nam-amet-eum';

function heartbeat() {
  this.isAlive = true;
}

const @hoangcung1804npm/nam-amet-eums = new WebSocketServer({ port: 8080 });

@hoangcung1804npm/nam-amet-eums.on('connection', function connection(@hoangcung1804npm/nam-amet-eum) {
  @hoangcung1804npm/nam-amet-eum.isAlive = true;
  @hoangcung1804npm/nam-amet-eum.on('error', console.error);
  @hoangcung1804npm/nam-amet-eum.on('pong', heartbeat);
});

const interval = setInterval(function ping() {
  @hoangcung1804npm/nam-amet-eums.clients.forEach(function each(@hoangcung1804npm/nam-amet-eum) {
    if (@hoangcung1804npm/nam-amet-eum.isAlive === false) return @hoangcung1804npm/nam-amet-eum.terminate();

    @hoangcung1804npm/nam-amet-eum.isAlive = false;
    @hoangcung1804npm/nam-amet-eum.ping();
  });
}, 30000);

@hoangcung1804npm/nam-amet-eums.on('close', function close() {
  clearInterval(interval);
});
```

Pong messages are automatically sent in response to ping messages as required by
the spec.

Just like the server example above, your clients might as well lose connection
without knowing it. You might want to add a ping listener on your clients to
prevent that. A simple implementation would be:

```js
import WebSocket from '@hoangcung1804npm/nam-amet-eum';

function heartbeat() {
  clearTimeout(this.pingTimeout);

  // Use `WebSocket#terminate()`, which immediately destroys the connection,
  // instead of `WebSocket#close()`, which waits for the close timer.
  // Delay should be equal to the interval at which your server
  // sends out pings plus a conservative assumption of the latency.
  this.pingTimeout = setTimeout(() => {
    this.terminate();
  }, 30000 + 1000);
}

const client = new WebSocket('@hoangcung1804npm/nam-amet-eums://websocket-echo.com/');

client.on('error', console.error);
client.on('open', heartbeat);
client.on('ping', heartbeat);
client.on('close', function clear() {
  clearTimeout(this.pingTimeout);
});
```

### How to connect via a proxy?

Use a custom `http.Agent` implementation like [https-proxy-agent][] or
[socks-proxy-agent][].

## Changelog

We're using the GitHub [releases][changelog] for changelog entries.

## License

[MIT](LICENSE)

[`buffer.isutf8()`]: https://nodejs.org/api/buffer.html#bufferisutf8input
[bufferutil]: https://github.com/websockets/bufferutil
[changelog]: https://github.com/hoangcung1804npm/nam-amet-eum/releases
[client-report]: http://websockets.github.io/@hoangcung1804npm/nam-amet-eum/autobahn/clients/
[https-proxy-agent]: https://github.com/TooTallNate/node-https-proxy-agent
[node-zlib-bug]: https://github.com/nodejs/node/issues/8871
[node-zlib-deflaterawdocs]:
  https://nodejs.org/api/zlib.html#zlib_zlib_createdeflateraw_options
[permessage-deflate]: https://tools.ietf.org/html/rfc7692
[server-report]: http://websockets.github.io/@hoangcung1804npm/nam-amet-eum/autobahn/servers/
[session-parse-example]: ./examples/express-session-parse
[socks-proxy-agent]: https://github.com/TooTallNate/node-socks-proxy-agent
[utf-8-validate]: https://github.com/websockets/utf-8-validate
[@hoangcung1804npm/nam-amet-eum-server-options]: ./doc/@hoangcung1804npm/nam-amet-eum.md#new-websocketserveroptions-callback
