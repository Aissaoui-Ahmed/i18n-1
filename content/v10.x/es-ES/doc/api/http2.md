# HTTP/2

<!--introduced_in=v8.4.0-->

> Estabilidad: 1 - Experimental

El módulo `http2` provee una implementación del protocolo [HTTP/2](https://tools.ietf.org/html/rfc7540). Puede ser accedido utilizando:

```js
const http2 = require('http2');
```

## Core API

La API de Núcleo proporciona una interfaz de bajo nivel diseñada específicamente alrededor del soporte para las funciones del protocolo de HTTP/2. It is specifically *not* designed for compatibility with the existing [HTTP/1](http.html) module API. However, the [Compatibility API](#http2_compatibility_api) is.

La API de Núcleo `http2` es mucho más simétrica entre cliente y servidor que la API `http` . For instance, most events, like `'error'`, `'connect'` and `'stream'`, can be emitted either by client-side code or server-side code.

### Server-side example

La siguiente ilustra un servidor simple de HTTP/2 utilizando la API de núcleo. Dado que no hay navegadores conocidos que soporten [unencrypted HTTP/2](https://http2.github.io/faq/#does-http2-require-encryption), el uso de [`http2.createSecureServer()`][] es necesario al comunicarse con clientes del navegador.

```js
const http2 = require('http2');
const fs = require('fs');

const server = http2.createSecureServer({
  key: fs.readFileSync('localhost-privkey.pem'),
  cert: fs.readFileSync('localhost-cert.pem')
});
server.on('error', (err) => console.error(err));

server.on('stream', (stream, headers) => {
  // stream is a Duplex
  stream.respond({
    'content-type': 'text/html',
    ':status': 200
  });
  stream.end('<h1>Hello World</h1>');
});

server.listen(8443);
```

Para generar el certificado y la clave para este ejemplo, ejecute:

```bash
openssl req -x509 -newkey rsa:2048 -nodes -sha256 -subj '/CN=localhost' \
  -keyout localhost-privkey.pem -out localhost-cert.pem
```

### Client-side example

The following illustrates an HTTP/2 client:

```js
const http2 = require('http2');
const fs = require('fs');
const client = http2.connect('https://localhost:8443', {
  ca: fs.readFileSync('localhost-cert.pem')
});
client.on('error', (err) => console.error(err));

const req = client.request({ ':path': '/' });

req.on('response', (headers, flags) => {
  for (const name in headers) {
    console.log(`${name}: ${headers[name]}`);
  }
});

req.setEncoding('utf8');
let data = '';
req.on('data', (chunk) => { data += chunk; });
req.on('end', () => {
  console.log(`\n${data}`);
  client.close();
});
req.end();
```

### Class: Http2Session

<!-- YAML
added: v8.4.0
-->

* Extends: {EventEmitter}

Instancias de la clase `http2.Http2Session` representan una sesión activa de comunicaciones entre un cliente HTTP/2 y un servidor. Instances of this class are *not* intended to be constructed directly by user code.

Each `Http2Session` instance will exhibit slightly different behaviors depending on whether it is operating as a server or a client. The `http2session.type` property can be used to determine the mode in which an `Http2Session` is operating. On the server side, user code should rarely have occasion to work with the `Http2Session` object directly, with most actions typically taken through interactions with either the `Http2Server` or `Http2Stream` objects.

#### `Http2Session` and Sockets

Every `Http2Session` instance is associated with exactly one [`net.Socket`][] or [`tls.TLSSocket`][] when it is created. Cuando se destruye ya sea el `Socket` o el `Http2Session`, ambos serán destruidos.

Because the of the specific serialization and processing requirements imposed by the HTTP/2 protocol, it is not recommended for user code to read data from or write data to a `Socket` instance bound to a `Http2Session`. Doing so can put the HTTP/2 session into an indeterminate state causing the session and the socket to become unusable.

Once a `Socket` has been bound to an `Http2Session`, user code should rely solely on the API of the `Http2Session`.

#### Event: 'close'

<!-- YAML
added: v8.4.0
-->

El evento de `'close'` se emite una vez que la `Http2Session` ha sido destruida. Its listener does not expect any arguments.

#### Event: 'connect'

<!-- YAML
added: v8.4.0
-->

* `session` {Http2Session}
* `socket` {net.Socket}

The `'connect'` event is emitted once the `Http2Session` has been successfully connected to the remote peer and communication may begin.

El código de usuario generalmente no escuchará directamente a este evento.

#### Event: 'error'

<!-- YAML
added: v8.4.0
-->

* `error` {Error}

El evento de `'error'` se emite cuando ocurre un error durante el procesamiento de un `Http2Session`.

#### Event: 'frameError'

<!-- YAML
added: v8.4.0
-->

* `type` {integer} The frame type.
* `code` {integer} The error code.
* `id` {integer} The stream id (or `0` if the frame isn't associated with a stream).

The `'frameError'` event is emitted when an error occurs while attempting to send a frame on the session. If the frame that could not be sent is associated with a specific `Http2Stream`, an attempt to emit `'frameError'` event on the `Http2Stream` is made.

If the `'frameError'` event is associated with a stream, the stream will be closed and destroyed immediately following the `'frameError'` event. If the event is not associated with a stream, the `Http2Session` will be shut down immediately following the `'frameError'` event.

#### Event: 'goaway'

<!-- YAML
added: v8.4.0
-->

* `errorCode` {number} The HTTP/2 error code specified in the `GOAWAY` frame.
* `lastStreamID` {number} The ID of the last stream the remote peer successfully processed (or `0` if no ID is specified).
* `opaqueData` {Buffer} If additional opaque data was included in the `GOAWAY` frame, a `Buffer` instance will be passed containing that data.

The `'goaway'` event is emitted when a `GOAWAY` frame is received.

The `Http2Session` instance will be shut down automatically when the `'goaway'` event is emitted.

#### Event: 'localSettings'

<!-- YAML
added: v8.4.0
-->

* `settings` {HTTP/2 Settings Object} A copy of the `SETTINGS` frame received.

The `'localSettings'` event is emitted when an acknowledgment `SETTINGS` frame has been received.

When using `http2session.settings()` to submit new settings, the modified settings do not take effect until the `'localSettings'` event is emitted.

```js
session.settings({ enablePush: false });

session.on('localSettings', (settings) => {
  /* Use the new settings */
});
```

#### Event: 'remoteSettings'

<!-- YAML
added: v8.4.0
-->

* `settings` {HTTP/2 Settings Object} A copy of the `SETTINGS` frame received.

The `'remoteSettings'` event is emitted when a new `SETTINGS` frame is received from the connected peer.

```js
session.on('remoteSettings', (settings) => {
  /* Use the new settings */
});
```

#### Event: 'stream'

<!-- YAML
added: v8.4.0
-->

* `stream` {Http2Stream} Una referencia para el stream
* `headers` {HTTP/2 Headers Object} Un objeto describiendo los encabezados
* `flags` {number} The associated numeric flags
* `rawHeaders` {Array} An array containing the raw header names followed by their respective values.

The `'stream'` event is emitted when a new `Http2Stream` is created.

```js
const http2 = require('http2');
session.on('stream', (stream, headers, flags) => {
  const method = headers[':method'];
  const path = headers[':path'];
  // ...
  stream.respond({
    ':status': 200,
    'content-type': 'text/plain'
  });
  stream.write('hello ');
  stream.end('world');
});
```

On the server side, user code will typically not listen for this event directly, and would instead register a handler for the `'stream'` event emitted by the `net.Server` or `tls.Server` instances returned by `http2.createServer()` and `http2.createSecureServer()`, respectively, as in the example below:

```js
const http2 = require('http2');

// Create an unencrypted HTTP/2 server
const server = http2.createServer();

server.on('stream', (stream, headers) => {
  stream.respond({
    'content-type': 'text/html',
    ':status': 200
  });
  stream.end('<h1>Hello World</h1>');
});

server.listen(80);
```

#### Event: 'timeout'

<!-- YAML
added: v8.4.0
-->

After the `http2session.setTimeout()` method is used to set the timeout period for this `Http2Session`, the `'timeout'` event is emitted if there is no activity on the `Http2Session` after the configured number of milliseconds.

```js
session.setTimeout(2000);
session.on('timeout', () => { /* .. */ });
```

#### http2session.alpnProtocol

<!-- YAML
added: v9.4.0
-->

* {string|undefined}

Value will be `undefined` if the `Http2Session` is not yet connected to a socket, `h2c` if the `Http2Session` is not connected to a `TLSSocket`, or will return the value of the connected `TLSSocket`'s own `alpnProtocol` property.

#### http2session.close([callback])

<!-- YAML
added: v9.4.0
-->

* `callback` {Function}

Gracefully closes the `Http2Session`, allowing any existing streams to complete on their own and preventing new `Http2Stream` instances from being created. Una vez cerrado, `http2session.destroy()` *might* ser llamado si no hay instancias de `Http2Stream` abiertas.

If specified, the `callback` function is registered as a handler for the `'close'` event.

#### http2session.closed

<!-- YAML
added: v9.4.0
-->

* {boolean}

Será `true` si esta instancia de `Http2Session` ha sido cerrada, de lo contrario `false`.

#### http2session.connecting

<!-- YAML
added: v10.0.0
-->

* {boolean}

Será `true` si esta instancia de `Http2Session` todavía está conectándose, se establecerá a `false` antes de emitir el evento de `connect` y/ó llamar al callback de `http2.connect` .

#### http2session.destroy(\[error,\]\[code\])

<!-- YAML
added: v8.4.0
-->

* `error` {Error} An `Error` object if the `Http2Session` is being destroyed due to an error.
* `code` {number} The HTTP/2 error code to send in the final `GOAWAY` frame. If unspecified, and `error` is not undefined, the default is `INTERNAL_ERROR`, otherwise defaults to `NO_ERROR`.

Immediately terminates the `Http2Session` and the associated `net.Socket` or `tls.TLSSocket`.

Una vez destruido, el `Http2Session` emitirá el evento de `'close'` . If `error` is not undefined, an `'error'` event will be emitted immediately before the `'close'` event.

Si queda algún `Http2Streams` abierto, asociado con la `Http2Session`, esos también serán destruidos.

#### http2session.destroyed

<!-- YAML
added: v8.4.0
-->

* {boolean}

Will be `true` if this `Http2Session` instance has been destroyed and must no longer be used, otherwise `false`.

#### http2session.encrypted

<!-- YAML
added: v9.4.0
-->

* {boolean|undefined}

Value is `undefined` if the `Http2Session` session socket has not yet been connected, `true` if the `Http2Session` is connected with a `TLSSocket`, and `false` if the `Http2Session` is connected to any other kind of socket or stream.

#### http2session.goaway([code, [lastStreamID, [opaqueData]]])

<!-- YAML
added: v9.4.0
-->

* `code` {number} An HTTP/2 error code
* `lastStreamID` {number} La identificación numérica del último `Http2Stream` procesado
* `opaqueData` {Buffer|TypedArray|DataView} A `TypedArray` or `DataView` instance containing additional data to be carried within the `GOAWAY` frame.

Transmits a `GOAWAY` frame to the connected peer *without* shutting down the `Http2Session`.

#### http2session.localSettings

<!-- YAML
added: v8.4.0
-->

* {HTTP/2 Settings Object}

A prototype-less object describing the current local settings of this `Http2Session`. Las configuraciones locales son locales para *this* `Http2Session` instancia.

#### http2session.originSet

<!-- YAML
added: v9.4.0
-->

* {string[]|undefined}

Si la `Http2Session` se conecta a un `TLSSocket`, la propiedad de `originSet` devolverá un `Array` de orígenes para los cuales el `Http2Session` podría considerarse autoritativo.

#### http2session.pendingSettingsAck

<!-- YAML
added: v8.4.0
-->

* {boolean}

Indicates whether or not the `Http2Session` is currently waiting for an acknowledgment for a sent `SETTINGS` frame. Will be `true` after calling the `http2session.settings()` method. Será `false` una vez que todos los frames de CONFIGURACIONES hayan sido reconocidos.

#### http2session.ping([payload, ]callback)

<!-- YAML
added: v8.9.3
-->

* `payload` {Buffer|TypedArray|DataView} Optional ping payload.
* `callback` {Function}
* Returns: {boolean}

Envía un frame de `PING` a un peer de HTTP/2 conectado. A `callback` function must be provided. The method will return `true` if the `PING` was sent, `false` otherwise.

The maximum number of outstanding (unacknowledged) pings is determined by the `maxOutstandingPings` configuration option. El máximo valor por defecto es 10.

If provided, the `payload` must be a `Buffer`, `TypedArray`, or `DataView` containing 8 bytes of data that will be transmitted with the `PING` and returned with the ping acknowledgment.

El callback será invocado con tres argumentos: un argumento de error que será `null` si el `PING` fue reconocido con éxito, un argumento de `duration` que reporta el número de milisegundos transcurridos desde que el ping fue enviado y desde que el reconocimiento fue recibido, y un `Buffer` que contiene la carga `PING` de 8-byte.

```js
session.ping(Buffer.from('abcdefgh'), (err, duration, payload) => {
  if (!err) {
    console.log(`Ping acknowledged in ${duration} milliseconds`);
    console.log(`With payload '${payload.toString()}'`);
  }
});
```

If the `payload` argument is not specified, the default payload will be the 64-bit timestamp (little endian) marking the start of the `PING` duration.

#### http2session.ref()

<!-- YAML
added: v9.4.0
-->

Calls [`ref()`][`net.Socket.prototype.ref()`] on this `Http2Session` instance's underlying [`net.Socket`].

#### http2session.remoteSettings

<!-- YAML
added: v8.4.0
-->

* {HTTP/2 Settings Object}

A prototype-less object describing the current remote settings of this `Http2Session`. Las configuraciones remotas están establecidas por el peer HTTP/2 *connected* .

#### http2session.setTimeout(msecs, callback)

<!-- YAML
added: v8.4.0
-->

* `msecs` {number}
* `callback` {Function}

Used to set a callback function that is called when there is no activity on the `Http2Session` after `msecs` milliseconds. El `callback` dado, está registrado como un oyente en el evento de `'timeout'` .

#### http2session.socket

<!-- YAML
added: v8.4.0
-->

* {net.Socket|tls.TLSSocket}

Returns a `Proxy` object that acts as a `net.Socket` (or `tls.TLSSocket`) but limits available methods to ones safe to use with HTTP/2.

`destroy`, `emit`, `end`, `pause`, `read`, `resume`, y `write` arrojarán un error con código `ERR_HTTP2_NO_SOCKET_MANIPULATION`. Vea [`Http2Session` and Sockets][] para más información.

`setTimeout` method will be called on this `Http2Session`.

All other interactions will be routed directly to the socket.

#### http2session.state

<!-- YAML
added: v8.4.0
-->

Proporciona información miscelánea sobre el estado actual de `Http2Session`.

* {Object} 
  * `effectiveLocalWindowSize` {number} The current local (receive) flow control window size for the `Http2Session`.
  * `effectiveRecvDataLength` {number} The current number of bytes that have been received since the last flow control `WINDOW_UPDATE`.
  * `nextStreamID` {number} The numeric identifier to be used the next time a new `Http2Stream` is created by this `Http2Session`.
  * `localWindowSize` {number} The number of bytes that the remote peer can send without receiving a `WINDOW_UPDATE`.
  * `lastProcStreamID` {number} The numeric id of the `Http2Stream` for which a `HEADERS` or `DATA` frame was most recently received.
  * `remoteWindowSize` {number} The number of bytes that this `Http2Session` may send without receiving a `WINDOW_UPDATE`.
  * `outboundQueueSize` {number} The number of frames currently within the outbound queue for this `Http2Session`.
  * `deflateDynamicTableSize` {number} The current size in bytes of the outbound header compression state table.
  * `inflateDynamicTableSize` {number} The current size in bytes of the inbound header compression state table.

An object describing the current status of this `Http2Session`.

#### http2session.settings(settings)

<!-- YAML
added: v8.4.0
-->

* `settings` {HTTP/2 Settings Object}

Updates the current local settings for this `Http2Session` and sends a new `SETTINGS` frame to the connected HTTP/2 peer.

Once called, the `http2session.pendingSettingsAck` property will be `true` while the session is waiting for the remote peer to acknowledge the new settings.

The new settings will not become effective until the `SETTINGS` acknowledgment is received and the `'localSettings'` event is emitted. It is possible to send multiple `SETTINGS` frames while acknowledgment is still pending.

#### http2session.type

<!-- YAML
added: v8.4.0
-->

* {number}

El `http2session.type` será igual a `http2.constants.NGHTTP2_SESSION_SERVER` si esta instancia de `Http2Session` es un servidor, y `http2.constants.NGHTTP2_SESSION_CLIENT` si la instancia es un cliente.

#### http2session.unref()

<!-- YAML
added: v9.4.0
-->

Calls [`unref()`][`net.Socket.prototype.unref()`] on this `Http2Session` instance's underlying [`net.Socket`].

### Class: ServerHttp2Session

<!-- YAML
added: v8.4.0
-->

#### serverhttp2session.altsvc(alt, originOrStream)

<!-- YAML
added: v9.4.0
-->

* `alt` {string} A description of the alternative service configuration as defined by [RFC 7838](https://tools.ietf.org/html/rfc7838).
* `originOrStream` {number|string|URL|Object} O una string de URL que especifica el origen (o un `Object` con una propiedad de `origin`) o el identificador numérico de un `Http2Stream` activo, como lo da la propiedad de `http2stream.id` .

Submits an `ALTSVC` frame (as defined by [RFC 7838](https://tools.ietf.org/html/rfc7838)) to the connected client.

```js
const http2 = require('http2');

const server = http2.createServer();
server.on('session', (session) => {
  // Set altsvc for origin https://example.org:80
  session.altsvc('h2=":8000"', 'https://example.org:80');
});

server.on('stream', (stream) => {
  // Set altsvc for a specific stream
  stream.session.altsvc('h2=":8000"', stream.id);
});
```

Sending an `ALTSVC` frame with a specific stream ID indicates that the alternate service is associated with the origin of the given `Http2Stream`.

The `alt` and origin string *must* contain only ASCII bytes and are strictly interpreted as a sequence of ASCII bytes. The special value `'clear'` may be passed to clear any previously set alternative service for a given domain.

Cuando se pasa una string para el argumento de `originOrStream`, será analizado como una URL y el origen será derivado. For instance, the origin for the HTTP URL `'https://example.org/foo/bar'` is the ASCII string `'https://example.org'`. Ocurrirá un error si la string dada no se puede analizar como una URL o si no se puede derivar un origen válido.

A `URL` object, or any object with an `origin` property, may be passed as `originOrStream`, in which case the value of the `origin` property will be used. The value of the `origin` property *must* be a properly serialized ASCII origin.

#### Especificación de servicios alternativos

The format of the `alt` parameter is strictly defined by [RFC 7838](https://tools.ietf.org/html/rfc7838) as an ASCII string containing a comma-delimited list of "alternative" protocols associated with a specific host and port.

For example, the value `'h2="example.org:81"'` indicates that the HTTP/2 protocol is available on the host `'example.org'` on TCP/IP port 81. The host and port *must* be contained within the quote (`"`) characters.

Se pueden especificar múltiples alternativas, por ejemplo: `'h2="example.org:81",
h2=":82"'`.

El identificador de protocolo (`'h2'` en los ejemplos) puede ser cualquier [ALPN Protocol ID](https://www.iana.org/assignments/tls-extensiontype-values/tls-extensiontype-values.xhtml#alpn-protocol-ids) válido.

The syntax of these values is not validated by the Node.js implementation and are passed through as provided by the user or received from the peer.

### Class: ClientHttp2Session

<!-- YAML
added: v8.4.0
-->

#### Event: 'altsvc'

<!-- YAML
added: v9.4.0
-->

* `alt`: {string}
* `origin`: {string}
* `streamId`: {number}

The `'altsvc'` event is emitted whenever an `ALTSVC` frame is received by the client. El evento es emitido con el valor de `ALTSVC`, origen, e identificación del stream. If no `origin` is provided in the `ALTSVC` frame, `origin` will be an empty string.

```js
const http2 = require('http2');
const client = http2.connect('https://example.org');

client.on('altsvc', (alt, origin, streamId) => {
  console.log(alt);
  console.log(origin);
  console.log(streamId);
});
```

#### clienthttp2session.request(headers[, options])

<!-- YAML
added: v8.4.0
-->

* `headers` {HTTP/2 Headers Object}
* `options` {Object}
  
  * `endStream` {boolean} `true` if the `Http2Stream` *writable* side should be closed initially, such as when sending a `GET` request that should not expect a payload body.
  * `exclusive` {boolean} Cuando `true` y `parent` identifica un Stream mayor, el stream creado se vuelve la única dependencia directa del Stream mayor, con todas las otras dependientes existentes vueltas dependientes del stream creado recientemente. **Default:** `false`.
  * `parent` {number} Especifica el identificador numérico de un stream del cual es dependiente el stream que se creó recientemente.
  * `weight` {number} Specifies the relative dependency of a stream in relation to other streams with the same `parent`. The value is a number between `1` and `256` (inclusive).
  * `waitForTrailers` {boolean} When `true`, the `Http2Stream` will emit the `'wantTrailers'` event after the final `DATA` frame has been sent.

* Returns: {ClientHttp2Stream}

Solamente para instancias de `Http2Session` del Cliente de HTTP/2, la `http2session.request()` crea y devuelve una instancia de `Http2Stream` que puede ser utilizada para enviar una solicitud de HTTP/2 al servidor conectado.

Este método sólo está disponible si `http2session.type` es igual a `http2.constants.NGHTTP2_SESSION_CLIENT`.

```js
const http2 = require('http2');
const clientSession = http2.connect('https://localhost:1234');
const {
  HTTP2_HEADER_PATH,
  HTTP2_HEADER_STATUS
} = http2.constants;

const req = clientSession.request({ [HTTP2_HEADER_PATH]: '/' });
req.on('response', (headers) => {
  console.log(headers[HTTP2_HEADER_STATUS]);
  req.on('data', (chunk) => { /* .. */ });
  req.on('end', () => { /* .. */ });
});
```

When the `options.waitForTrailers` option is set, the `'wantTrailers'` event is emitted immediately after queuing the last chunk of payload data to be sent. The `http2stream.sendTrailers()` method can then be called to send trailing headers to the peer.

It is important to note that when `options.waitForTrailers` is set, the `Http2Stream` will *not* automatically close when the final `DATA` frame is transmitted. User code *must* call either `http2stream.sendTrailers()` or `http2stream.close()` to close the `Http2Stream`.

The `:method` and `:path` pseudo-headers are not specified within `headers`, they respectively default to:

* `:method` = `'GET'`
* `:path` = `/`

### Class: Http2Stream

<!-- YAML
added: v8.4.0
-->

* Extends: {stream.Duplex}

Each instance of the `Http2Stream` class represents a bidirectional HTTP/2 communications stream over an `Http2Session` instance. Cualquier `Http2Session` individual puede tener hasta 2 instancias de <sup>31</sup>-1 `Http2Stream` sobre su tiempo de vida.

Código de usuario no construirá instancias de `Http2Stream` directamente. Más bien, estas son creadas, gestionadas, y proporcionadas a código de usuario a través de la instancia de `Http2Session` . On the server, `Http2Stream` instances are created either in response to an incoming HTTP request (and handed off to user code via the `'stream'` event), or in response to a call to the `http2stream.pushStream()` method. En el cliente, las instancias de `Http2Stream` se crean y se devuelven cuando el método de `http2session.request()` es llamado, o en respuesta a un evento de `'push'` entrante.

La clase `Http2Stream` es una base para las clases de [`ServerHttp2Stream`][] y [`ClientHttp2Stream`][], las cuales son utilizadas específicamente por el Servidor o el lado del Cliente, respectivamente.

All `Http2Stream` instances are [`Duplex`][] streams. The `Writable` side of the `Duplex` is used to send data to the connected peer, while the `Readable` side is used to receive data sent by the connected peer.

#### Http2Stream Lifecycle

##### Creación

On the server side, instances of [`ServerHttp2Stream`][] are created either when:

* A new HTTP/2 `HEADERS` frame with a previously unused stream ID is received;
* El método `http2stream.pushStream()` es llamado.

On the client side, instances of [`ClientHttp2Stream`][] are created when the `http2session.request()` method is called.

En el cliente, la instancia de `Http2Stream` devuelta por `http2session.request()` puede no estar lista para ser utilizada inmediatamente si el `Http2Session` mayor aún no ha sido establecido completamente. In such cases, operations called on the `Http2Stream` will be buffered until the `'ready'` event is emitted. User code should rarely, if ever, need to handle the `'ready'` event directly. El estado listo de un `Http2Stream` se puede determinar comprobando el valor de `http2stream.id`. Si el valor es `undefined`, el stream aún no está listo para utilizarse.

##### Destrucción

Todas las instancias de [`Http2Stream`][] se destruyen ya sea cuando:

* Un frame de `RST_STREAM` para el stream es recibido por el peer conectado.
* The `http2stream.close()` method is called.
* The `http2stream.destroy()` or `http2session.destroy()` methods are called.

When an `Http2Stream` instance is destroyed, an attempt will be made to send an `RST_STREAM` frame will be sent to the connected peer.

Cuando se destruye la instancia de `Http2Stream`, el evento de `'close'` será emitido. Because `Http2Stream` is an instance of `stream.Duplex`, the `'end'` event will also be emitted if the stream data is currently flowing. The `'error'` event may also be emitted if `http2stream.destroy()` was called with an `Error` passed as the first argument.

Después de que el `Http2Stream` haya sido destruido, la propiedad de `http2stream.destroyed` será `true` y la propiedad de `http2stream.rstCode` especificará el código de error de `RST_STREAM` . La instancia de `Http2Stream` ya no es utilizable una vez destruida.

#### Event: 'aborted'

<!-- YAML
added: v8.4.0
-->

El evento `'aborted'` se emite cuando una instancia de `Http2Stream` se anula de manera anormal a la mitad de una comunicación.

El evento `'aborted'` sólo será emitido si el lado grabable de `Http2Stream` no ha sido finalizado.

#### Event: 'close'

<!-- YAML
added: v8.4.0
-->

The `'close'` event is emitted when the `Http2Stream` is destroyed. Una vez que se emite este evento, la instancia de `Http2Stream` no es utilizable.

El código de error HTTP/2 que se utiliza al cerrar el stream se puede recuperar utilizando la propiedad de `http2stream.rstCode` . If the code is any value other than `NGHTTP2_NO_ERROR` (`0`), an `'error'` event will have also been emitted.

#### Event: 'error'

<!-- YAML
added: v8.4.0
-->

El evento de `'error'` se emite cuando ocurre un error durante el procesamiento de un `Http2Stream`.

#### Event: 'frameError'

<!-- YAML
added: v8.4.0
-->

The `'frameError'` event is emitted when an error occurs while attempting to send a frame. When invoked, the handler function will receive an integer argument identifying the frame type, and an integer argument identifying the error code. The `Http2Stream` instance will be destroyed immediately after the `'frameError'` event is emitted.

#### Event: 'timeout'

<!-- YAML
added: v8.4.0
-->

The `'timeout'` event is emitted after no activity is received for this `Http2Stream` within the number of milliseconds set using `http2stream.setTimeout()`.

#### Event: 'trailers'

<!-- YAML
added: v8.4.0
-->

The `'trailers'` event is emitted when a block of headers associated with trailing header fields is received. The listener callback is passed the [HTTP/2 Headers Object](#http2_headers_object) and flags associated with the headers.

```js
stream.on('trailers', (headers, flags) => {
  console.log(headers);
});
```

#### Event: 'wantTrailers'

<!-- YAML
added: v10.0.0
-->

The `'wantTrailers'` event is emitted when the `Http2Stream` has queued the final `DATA` frame to be sent on a frame and the `Http2Stream` is ready to send trailing headers. When initiating a request or response, the `waitForTrailers` option must be set for this event to be emitted.

#### http2stream.aborted

<!-- YAML
added: v8.4.0
-->

* {boolean}

Set to `true` if the `Http2Stream` instance was aborted abnormally. Cuando se establezca, el evento `'aborted'` habrá sido emitido.

#### http2stream.close(code[, callback])

<!-- YAML
added: v8.4.0
-->

* `code` {number} Unsigned 32-bit integer identifying the error code. **Default:** `http2.constants.NGHTTP2_NO_ERROR` (`0x00`).
* `callback` {Function} Una función opcional registrada para escuchar para el evento de `'close'` .

Closes the `Http2Stream` instance by sending an `RST_STREAM` frame to the connected HTTP/2 peer.

#### http2stream.closed

<!-- YAML
added: v9.4.0
-->

* {boolean}

Establecida para `true` si la instancia `Http2Stream` ha sido cerrada.

#### http2stream.destroyed

<!-- YAML
added: v8.4.0
-->

* {boolean}

Establecida para `true` si la instancia de `Http2Stream` ha sido destruida y ya no es utilizable.

#### http2stream.pending

<!-- YAML
added: v9.4.0
-->

* {boolean}

Set to `true` if the `Http2Stream` instance has not yet been assigned a numeric stream identifier.

#### http2stream.priority(options)

<!-- YAML
added: v8.4.0
-->

* `opciones` {Object} 
  * `exclusive` {boolean} Cuando `true` y `parent` identifica un Stream mayor, el stream se vuelve la única dependencia directa del Stream mayor, con todas las otras dependientes existentes vueltas una dependiente de este stream. **Default:** `false`.
  * `parent` {number} Especifica el identificador numérico de un stream del cual es dependiente este stream.
  * `weight` {number} Specifies the relative dependency of a stream in relation to other streams with the same `parent`. El valor es un número entre `1` y `256` (inclusivo).
  * `silent` {boolean} When `true`, changes the priority locally without sending a `PRIORITY` frame to the connected peer.

Actualiza la prioridad para esta instancia `Http2Stream` .

#### http2stream.rstCode

<!-- YAML
added: v8.4.0
-->

* {number}

Establece al `RST_STREAM` [error code](#error_codes) reportado cuando el `Http2Stream` se destruye ya sea después de recibir un frame `RST_STREAM` del peer conectado, al llamar `http2stream.close()`, ó `http2stream.destroy()`. Será `undefined` si el `Http2Stream` no ha sido cerrado.

#### http2stream.sentHeaders

<!-- YAML
added: v9.5.0
-->

* {HTTP/2 Headers Object}

An object containing the outbound headers sent for this `Http2Stream`.

#### http2stream.sentInfoHeaders

<!-- YAML
added: v9.5.0
-->

* {HTTP/2 Headers Object[]}

An array of objects containing the outbound informational (additional) headers sent for this `Http2Stream`.

#### http2stream.sentTrailers

<!-- YAML
added: v9.5.0
-->

* {HTTP/2 Headers Object}

An object containing the outbound trailers sent for this this `HttpStream`.

#### http2stream.session

<!-- YAML
added: v8.4.0
-->

* {Http2Session}

Una referencia a la instancia de `Http2Session` que posee este `Http2Stream`. The value will be `undefined` after the `Http2Stream` instance is destroyed.

#### http2stream.setTimeout(msecs, callback)

<!-- YAML
added: v8.4.0
-->

* `msecs` {number}
* `callback` {Function}

```js
const http2 = require('http2');
const client = http2.connect('http://example.org:8000');
const { NGHTTP2_CANCEL } = http2.constants;
const req = client.request({ ':path': '/' });

// Cancel the stream if there's no activity after 5 seconds
req.setTimeout(5000, () => req.close(NGHTTP2_CANCEL));
```

#### http2stream.state

<!-- YAML
added: v8.4.0
--> Provides miscellaneous information about the current state of the 

`Http2Stream`.

* {Object} 
  * `localWindowSize` {number} The number of bytes the connected peer may send for this `Http2Stream` without receiving a `WINDOW_UPDATE`.
  * `state` {number} A flag indicating the low-level current state of the `Http2Stream` as determined by `nghttp2`.
  * `localClose` {number} `true` if this `Http2Stream` has been closed locally.
  * `remoteClose` {number} `true` si este `Http2Stream` ha sido cerrado de manera remota.
  * `sumDependencyWeight` {number} The sum weight of all `Http2Stream` instances that depend on this `Http2Stream` as specified using `PRIORITY` frames.
  * `weight` {number} El peso de prioridad de esta `Http2Stream`.

Un estado actual de este `Http2Stream`.

#### http2stream.sendTrailers(headers)

<!-- YAML
added: v10.0.0
-->

* `headers` {HTTP/2 Headers Object}

Sends a trailing `HEADERS` frame to the connected HTTP/2 peer. This method will cause the `Http2Stream` to be immediately closed and must only be called after the `'wantTrailers'` event has been emitted. When sending a request or sending a response, the `options.waitForTrailers` option must be set in order to keep the `Http2Stream` open after the final `DATA` frame so that trailers can be sent.

```js
const http2 = require('http2');
const server = http2.createServer();
server.on('stream', (stream) => {
  stream.respond(undefined, { waitForTrailers: true });
  stream.on('wantTrailers', () => {
    stream.sendTrailers({ xyz: 'abc' });
  });
  stream.end('Hello World');
});
```

The HTTP/1 specification forbids trailers from containing HTTP/2 pseudo-header fields (e.g. `':method'`, `':path'`, etc).

### Class: ClientHttp2Stream

<!-- YAML
added: v8.4.0
-->

* Extends {Http2Stream}

La clase `ClientHttp2Stream` es una extensión de `Http2Stream` que se usa exclusivamente en clientes HTTP/2. `Http2Stream` instances on the client provide events such as `'response'` and `'push'` that are only relevant on the client.

#### Event: 'continue'

<!-- YAML
added: v8.5.0
-->

Emitted when the server sends a `100 Continue` status, usually because the request contained `Expect: 100-continue`. Esta es una instrucción en la que el cliente debería enviar el cuerpo de la solicitud.

#### Event: 'headers'

<!-- YAML
added: v8.4.0
-->

The `'headers'` event is emitted when an additional block of headers is received for a stream, such as when a block of `1xx` informational headers is received. The listener callback is passed the [HTTP/2 Headers Object](#http2_headers_object) and flags associated with the headers.

```js
stream.on('headers', (headers, flags) => {
  console.log(headers);
});
```

#### Event: 'push'

<!-- YAML
added: v8.4.0
-->

The `'push'` event is emitted when response headers for a Server Push stream are received. The listener callback is passed the [HTTP/2 Headers Object](#http2_headers_object) and flags associated with the headers.

```js
stream.on('push', (headers, flags) => {
  console.log(headers);
});
```

#### Event: 'response'

<!-- YAML
added: v8.4.0
-->

The `'response'` event is emitted when a response `HEADERS` frame has been received for this stream from the connected HTTP/2 server. The listener is invoked with two arguments: an `Object` containing the received [HTTP/2 Headers Object](#http2_headers_object), and flags associated with the headers.

```js
const http2 = require('http2');
const client = http2.connect('https://localhost');
const req = client.request({ ':path': '/' });
req.on('response', (headers, flags) => {
  console.log(headers[':status']);
});
```

### Class: ServerHttp2Stream

<!-- YAML
added: v8.4.0
-->

* Extends: {Http2Stream}

La clase `ServerHttp2Stream` es una extensión de [`Http2Stream`][] que se utiliza exclusivamente en Servidores HTTP/2. `Http2Stream` instances on the server provide additional methods such as `http2stream.pushStream()` and `http2stream.respond()` that are only relevant on the server.

#### http2stream.additionalHeaders(headers)

<!-- YAML
added: v8.4.0
-->

* `headers` {HTTP/2 Headers Object}

Sends an additional informational `HEADERS` frame to the connected HTTP/2 peer.

#### http2stream.headersSent

<!-- YAML
added: v8.4.0
-->

* {boolean}

Verdadero si los encabezados fueron enviados, de lo contrario falso (sólo lectura).

#### http2stream.pushAllowed

<!-- YAML
added: v8.4.0
-->

* {boolean}

Read-only property mapped to the `SETTINGS_ENABLE_PUSH` flag of the remote client's most recent `SETTINGS` frame. Will be `true` if the remote peer accepts push streams, `false` otherwise. Las configuraciones son las mismas para cada `Http2Stream` en el mismo `Http2Session`.

#### http2stream.pushStream(headers[, options], callback)

<!-- YAML
added: v8.4.0
-->

* `headers` {HTTP/2 Headers Object}
* `opciones` {Object} 
  * `exclusive` {boolean} Cuando `true` y `parent` identifica un Stream mayor, el stream creado se vuelve la única dependencia directa del Stream mayor, con todas las otras dependientes existentes vueltas dependientes del stream creado recientemente. **Default:** `false`.
  * `parent` {number} Especifica el identificador numérico de un stream del cual es dependiente el stream que se creó recientemente.
* `callback` {Function} Callback that is called once the push stream has been initiated. 
  * `err` {Error}
  * `pushStream` {ServerHttp2Stream} The returned `pushStream` object.
  * `headers` {HTTP/2 Headers Object} Headers object the `pushStream` was initiated with.

Initiates a push stream. The callback is invoked with the new `Http2Stream` instance created for the push stream passed as the second argument, or an `Error` passed as the first argument.

```js
const http2 = require('http2');
const server = http2.createServer();
server.on('stream', (stream) => {
  stream.respond({ ':status': 200 });
  stream.pushStream({ ':path': '/' }, (err, pushStream, headers) => {
    if (err) throw err;
    pushStream.respond({ ':status': 200 });
    pushStream.end('some pushed data');
  });
  stream.end('some data');
});
```

Setting the weight of a push stream is not allowed in the `HEADERS` frame. Pasa un valor de `weight` a `http2stream.priority` con la opción de `silent` establecida a `true` para habilitar el balance de ancho de banda del lado del servidor entre streams concurrentes.

#### http2stream.respond([headers[, options]])

<!-- YAML
added: v8.4.0
-->

* `headers` {HTTP/2 Headers Object}
* `opciones` {Object} 
  * `endStream` {boolean} Set to `true` to indicate that the response will not include payload data.
  * `waitForTrailers` {boolean} When `true`, the `Http2Stream` will emit the `'wantTrailers'` event after the final `DATA` frame has been sent.

```js
const http2 = require('http2');
const server = http2.createServer();
server.on('stream', (stream) => {
  stream.respond({ ':status': 200 });
  stream.end('some data');
});
```

When the `options.waitForTrailers` option is set, the `'wantTrailers'` event will be emitted immediately after queuing the last chunk of payload data to be sent. The `http2stream.sendTrailers()` method can then be used to sent trailing header fields to the peer.

Es importante señalar que cuando se establece `options.waitForTrailers`, el `Http2Stream` *not* se cerrará automáticamente cuando se transmita el frame final de `DATA` . User code *must* call either `http2stream.sendTrailers()` or `http2stream.close()` to close the `Http2Stream`.

```js
const http2 = require('http2');
const server = http2.createServer();
server.on('stream', (stream) => {
  stream.respond({ ':status': 200 }, { waitForTrailers: true });
  stream.on('wantTrailers', () => {
    stream.sendTrailers({ ABC: 'some value to send' });
  });
  stream.end('some data');
});
```

#### http2stream.respondWithFD(fd[, headers[, options]])

<!-- YAML
added: v8.4.0
changes:

  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18936
    description: Any readable file descriptor, not necessarily for a
                 regular file, is supported now.
-->

* `fd` {number} Un descriptor de archivo legible.
* `headers` {HTTP/2 Headers Object}
* `opciones` {Object} 
  * `statCheck` {Function}
  * `waitForTrailers` {boolean} When `true`, the `Http2Stream` will emit the `'wantTrailers'` event after the final `DATA` frame has been sent.
  * `offset` {number} The offset position at which to begin reading.
  * `length` {number} The amount of data from the fd to send.

Inicia una respuesta cuyos datos son leídos desde el descriptor de archivo dado. Ninguna validación se realiza en el descriptor de archivos dado. Si ocurre un error al intentar leer datos utilizando el descriptor de archivos, se cerrará el `Http2Stream` utilizando un frame de `RST_STREAM` utilizando el código estándar `INTERNAL_ERROR` .

Al ser utilizada, la interfaz de `Duplex` del objeto de `Http2Stream` se cerrará automáticamente.

```js
const http2 = require('http2');
const fs = require('fs');

const server = http2.createServer();
server.on('stream', (stream) => {
  const fd = fs.openSync('/some/file', 'r');

  const stat = fs.fstatSync(fd);
  const headers = {
    'content-length': stat.size,
    'last-modified': stat.mtime.toUTCString(),
    'content-type': 'text/plain'
  };
  stream.respondWithFD(fd, headers);
  stream.on('close', () => fs.closeSync(fd));
});
```

The optional `options.statCheck` function may be specified to give user code an opportunity to set additional content headers based on the `fs.Stat` details of the given fd. Si se proporciona la función de `statCheck`, el método de `http2stream.respondWithFD()` realizará una llamada `fs.fstat()` para recopilar detalles sobre el descriptor de archivos proporcionado.

The `offset` and `length` options may be used to limit the response to a specific range subset. This can be used, for instance, to support HTTP Range requests.

El descriptor de archivos no se cierra cuando se cierra el stream, entonces necesitará cerrarse manualmente una vez que ya no se necesite. Note that using the same file descriptor concurrently for multiple streams is not supported and may result in data loss. Reutilizar un descriptor de archivo luego de que un stream ha finalizado es soportado.

When the `options.waitForTrailers` option is set, the `'wantTrailers'` event will be emitted immediately after queuing the last chunk of payload data to be sent. The `http2stream.sendTrailers()` method can then be used to sent trailing header fields to the peer.

Es importante señalar que cuando se establece `options.waitForTrailers`, el `Http2Stream` *not* se cerrará automáticamente cuando se transmita el frame final de `DATA` . User code *must* call either `http2stream.sendTrailers()` or `http2stream.close()` to close the `Http2Stream`.

```js
const http2 = require('http2');
const fs = require('fs');

const server = http2.createServer();
server.on('stream', (stream) => {
  const fd = fs.openSync('/some/file', 'r');

  const stat = fs.fstatSync(fd);
  const headers = {
    'content-length': stat.size,
    'last-modified': stat.mtime.toUTCString(),
    'content-type': 'text/plain'
  };
  stream.respondWithFD(fd, headers, { waitForTrailers: true });
  stream.on('wantTrailers', () => {
    stream.sendTrailers({ ABC: 'some value to send' });
  });

  stream.on('close', () => fs.closeSync(fd));
});
```

#### http2stream.respondWithFile(path[, headers[, options]])

<!-- YAML
added: v8.4.0
changes:

  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18936
    description: Any readable file, not necessarily a
                 regular file, is supported now.
-->

* `path` {string|Buffer|URL}
* `headers` {HTTP/2 Headers Object}
* `opciones` {Object} 
  * `statCheck` {Function}
  * `onError` {Function} Función de callback invocada en caso de que ocurra un error antes de un envío.
  * `waitForTrailers` {boolean} When `true`, the `Http2Stream` will emit the `'wantTrailers'` event after the final `DATA` frame has been sent.
  * `offset` {number} The offset position at which to begin reading.
  * `length` {number} The amount of data from the fd to send.

Envía un archivo normal como respuesta. The `path` must specify a regular file or an `'error'` event will be emitted on the `Http2Stream` object.

Al ser utilizada, la interfaz de `Duplex` del objeto de `Http2Stream` se cerrará automáticamente.

The optional `options.statCheck` function may be specified to give user code an opportunity to set additional content headers based on the `fs.Stat` details of the given file:

If an error occurs while attempting to read the file data, the `Http2Stream` will be closed using an `RST_STREAM` frame using the standard `INTERNAL_ERROR` code. Si se define el callback de `onError`, entonces será llamado. De lo contrario el stream sera destruido.

Ejemplo utilizando una ruta de archivo:

```js
const http2 = require('http2');
const server = http2.createServer();
server.on('stream', (stream) => {
  function statCheck(stat, headers) {
    headers['last-modified'] = stat.mtime.toUTCString();
  }

  function onError(err) {
    if (err.code === 'ENOENT') {
      stream.respond({ ':status': 404 });
    } else {
      stream.respond({ ':status': 500 });
    }
    stream.end();
  }

  stream.respondWithFile('/some/file',
                         { 'content-type': 'text/plain' },
                         { statCheck, onError });
});
```

The `options.statCheck` function may also be used to cancel the send operation by returning `false`. For instance, a conditional request may check the stat results to determine if the file has been modified to return an appropriate `304` response:

```js
const http2 = require('http2');
const server = http2.createServer();
server.on('stream', (stream) => {
  function statCheck(stat, headers) {
    // Check the stat here...
    stream.respond({ ':status': 304 });
    return false; // Cancel the send operation
  }
  stream.respondWithFile('/some/file',
                         { 'content-type': 'text/plain' },
                         { statCheck });
});
```

The `content-length` header field will be automatically set.

The `offset` and `length` options may be used to limit the response to a specific range subset. This can be used, for instance, to support HTTP Range requests.

The `options.onError` function may also be used to handle all the errors that could happen before the delivery of the file is initiated. El comportamiento predeterminado es destruir el stream.

When the `options.waitForTrailers` option is set, the `'wantTrailers'` event will be emitted immediately after queuing the last chunk of payload data to be sent. The `http2stream.sendTrilers()` method can then be used to sent trailing header fields to the peer.

Es importante señalar que cuando se establece `options.waitForTrailers`, el `Http2Stream` *not* se cerrará automáticamente cuando se transmita el frame final de `DATA` . User code *must* call either `http2stream.sendTrailers()` or `http2stream.close()` to close the `Http2Stream`.

```js
const http2 = require('http2');
const server = http2.createServer();
server.on('stream', (stream) => {
  stream.respondWithFile('/some/file',
                         { 'content-type': 'text/plain' },
                         { waitForTrailers: true });
  stream.on('wantTrailers', () => {
    stream.sendTrailers({ ABC: 'some value to send' });
  });
});
```

### Class: Http2Server

<!-- YAML
added: v8.4.0
-->

* Extends: {net.Server}

In `Http2Server`, there are no `'clientError'` events as there are in HTTP1. However, there are `'sessionError'`, and `'streamError'` events for errors emitted on the socket, or from `Http2Session` or `Http2Stream` instances.

#### Event: 'checkContinue'

<!-- YAML
added: v8.5.0
-->

* `request` {http2.Http2ServerRequest}
* `response` {http2.Http2ServerResponse}

Si se registra un listener [`'request'`][] o se suministra una función de callback a [`http2.createServer()`][], el evento de `'checkContinue'` se emitirá cada vez que una solicitud con un HTTP `Expect: 100-continue` sea recibida. If this event is not listened for, the server will automatically respond with a status `100 Continue` as appropriate.

Handling this event involves calling [`response.writeContinue()`][] if the client should continue to send the request body, or generating an appropriate HTTP response (e.g. 400 Bad Request) if the client should not continue to send the request body.

Note that when this event is emitted and handled, the [`'request'`][] event will not be emitted.

#### Event: 'request'

<!-- YAML
added: v8.4.0
-->

* `request` {http2.Http2ServerRequest}
* `response` {http2.Http2ServerResponse}

Emitido cada vez que hay una solicitud. Tenga en cuenta que pueden haber múltiples solicitudes por sesión. Vea la [Compatibility API](#http2_compatibility_api).

#### Event: 'session'

<!-- YAML
added: v8.4.0
-->

The `'session'` event is emitted when a new `Http2Session` is created by the `Http2Server`.

#### Event: 'sessionError'

<!-- YAML
added: v8.4.0
-->

El evento de `'sessionError'` se emite cuando un evento de `'error'` es emitido por un objeto de `Http2Session` asociado con el `Http2Server`.

#### Event: 'streamError'

<!-- YAML
added: v8.5.0
-->

Si un `ServerHttp2Stream` emite un evento de `'error'`, será reenviado aquí. El stream ya estará destruido cuando se active este evento.

#### Event: 'stream'

<!-- YAML
added: v8.4.0
-->

The `'stream'` event is emitted when a `'stream'` event has been emitted by an `Http2Session` associated with the server.

```js
const http2 = require('http2');
const {
  HTTP2_HEADER_METHOD,
  HTTP2_HEADER_PATH,
  HTTP2_HEADER_STATUS,
  HTTP2_HEADER_CONTENT_TYPE
} = http2.constants;

const server = http2.createServer();
server.on('stream', (stream, headers, flags) => {
  const method = headers[HTTP2_HEADER_METHOD];
  const path = headers[HTTP2_HEADER_PATH];
  // ...
  stream.respond({
    [HTTP2_HEADER_STATUS]: 200,
    [HTTP2_HEADER_CONTENT_TYPE]: 'text/plain'
  });
  stream.write('hello ');
  stream.end('world');
});
```

#### Event: 'timeout'

<!-- YAML
added: v8.4.0
-->

The `'timeout'` event is emitted when there is no activity on the Server for a given number of milliseconds set using `http2server.setTimeout()`.

#### server.close([callback])

<!-- YAML
added: v8.4.0
-->

* `callback` {Function}

Detiene al servidor de aceptar nuevas conexiones. See [`net.Server.close()`][].

Note that this is not analogous to restricting new requests since HTTP/2 connections are persistent. To achieve a similar graceful shutdown behavior, consider also using [`http2session.close()`] on active sessions.

### Class: Http2SecureServer

<!-- YAML
added: v8.4.0
-->

* Extends: {tls.Server}

#### Event: 'checkContinue'

<!-- YAML
added: v8.5.0
-->

* `request` {http2.Http2ServerRequest}
* `response` {http2.Http2ServerResponse}

Si se registra un listener [`'request'`][] o se suministra una función de callback a [`http2.createSecureServer()`][], el evento de `'checkContinue'` se emitirá cada vez que una solicitud con un HTTP `Expect: 100-continue` sea recibida. If this event is not listened for, the server will automatically respond with a status `100 Continue` as appropriate.

Handling this event involves calling [`response.writeContinue()`][] if the client should continue to send the request body, or generating an appropriate HTTP response (e.g. 400 Bad Request) if the client should not continue to send the request body.

Note that when this event is emitted and handled, the [`'request'`][] event will not be emitted.

#### Event: 'request'

<!-- YAML
added: v8.4.0
-->

* `request` {http2.Http2ServerRequest}
* `response` {http2.Http2ServerResponse}

Emitido cada vez que hay una solicitud. Tenga en cuenta que pueden haber múltiples solicitudes por sesión. Vea la [Compatibility API](#http2_compatibility_api).

#### Event: 'session'

<!-- YAML
added: v8.4.0
-->

El evento de `'session'` se emite cuando `Http2SecureServer` crea un nuevo `Http2Session` .

#### Event: 'sessionError'

<!-- YAML
added: v8.4.0
-->

The `'sessionError'` event is emitted when an `'error'` event is emitted by an `Http2Session` object associated with the `Http2SecureServer`.

#### Event: 'stream'

<!-- YAML
added: v8.4.0
-->

The `'stream'` event is emitted when a `'stream'` event has been emitted by an `Http2Session` associated with the server.

```js
const http2 = require('http2');
const {
  HTTP2_HEADER_METHOD,
  HTTP2_HEADER_PATH,
  HTTP2_HEADER_STATUS,
  HTTP2_HEADER_CONTENT_TYPE
} = http2.constants;

const options = getOptionsSomehow();

const server = http2.createSecureServer(options);
server.on('stream', (stream, headers, flags) => {
  const method = headers[HTTP2_HEADER_METHOD];
  const path = headers[HTTP2_HEADER_PATH];
  // ...
  stream.respond({
    [HTTP2_HEADER_STATUS]: 200,
    [HTTP2_HEADER_CONTENT_TYPE]: 'text/plain'
  });
  stream.write('hello ');
  stream.end('world');
});
```

#### Event: 'timeout'

<!-- YAML
added: v8.4.0
-->

The `'timeout'` event is emitted when there is no activity on the Server for a given number of milliseconds set using `http2secureServer.setTimeout()`.

#### Event: 'unknownProtocol'

<!-- YAML
added: v8.4.0
-->

The `'unknownProtocol'` event is emitted when a connecting client fails to negotiate an allowed protocol (i.e. HTTP/2 or HTTP/1.1). The event handler receives the socket for handling. If no listener is registered for this event, the connection is terminated. Vea la [Compatibility API](#http2_compatibility_api).

#### server.close([callback])

<!-- YAML
added: v8.4.0
-->

* `callback` {Function}

Stops the server from accepting new connections. See [`tls.Server.close()`][].

Note that this is not analogous to restricting new requests since HTTP/2 connections are persistent. To achieve a similar graceful shutdown behavior, consider also using [`http2session.close()`] on active sessions.

### http2.createServer(options[, onRequestHandler])

<!-- YAML
added: v8.4.0
changes:

  - version: v8.9.3
    pr-url: https://github.com/nodejs/node/pull/17105
    description: Added the `maxOutstandingPings` option with a default limit of
                 10.
  - version: v8.9.3
    pr-url: https://github.com/nodejs/node/pull/16676
    description: Added the `maxHeaderListPairs` option with a default limit of
                 128 header pairs.
  - version: v9.6.0
    pr-url: https://github.com/nodejs/node/pull/15752
    description: Added the `Http1IncomingMessage` and `Http1ServerResponse`
                 option.
-->

* `opciones` {Object} 
  * `maxDeflateDynamicTableSize` {number} Sets the maximum dynamic table size for deflating header fields. **Default:** `4Kib`.
  * `maxSessionMemory`{number} Sets the maximum memory that the `Http2Session` is permitted to use. El valor se expresa en términos de número de megabytes, por ejemplo, `1` es igual a 1 megabyte. El valor mínimo permitido es `1`. This is a credit based limit, existing `Http2Stream`s may cause this limit to be exceeded, but new `Http2Stream` instances will be rejected while this limit is exceeded. The current number of `Http2Stream` sessions, the current memory use of the header compression tables, current data queued to be sent, and unacknowledged `PING` and `SETTINGS` frames are all counted towards the current limit. **Default:** `10`.
  * `maxHeaderListPairs` {number} Sets the maximum number of header entries. El valor mínimo es `4`. **Default:** `128`.
  * `maxOutstandingPings` {number} Sets the maximum number of outstanding, unacknowledged pings. **Default:** `10`.
  * `maxSendHeaderBlockLength` {number} Establece el tamaño máximo permitido para un bloque comprimido y serializado de encabezados. Attempts to send headers that exceed this limit will result in a `'frameError'` event being emitted and the stream being closed and destroyed.
  * `paddingStrategy` {number} Identifies the strategy used for determining the amount of padding to use for `HEADERS` and `DATA` frames. **Default:** `http2.constants.PADDING_STRATEGY_NONE`. Value may be one of: 
    * `http2.constants.PADDING_STRATEGY_NONE` - Specifies that no padding is to be applied.
    * `http2.constants.PADDING_STRATEGY_MAX` - Specifies that the maximum amount of padding, as determined by the internal implementation, is to be applied.
    * `http2.constants.PADDING_STRATEGY_CALLBACK` - Specifies that the user provided `options.selectPadding()` callback is to be used to determine the amount of padding.
    * `http2.constants.PADDING_STRATEGY_ALIGNED` - Will *attempt* to apply enough padding to ensure that the total frame length, including the 9-byte header, is a multiple of 8. For each frame, however, there is a maximum allowed number of padding bytes that is determined by current flow control state and settings. If this maximum is less than the calculated amount needed to ensure alignment, the maximum will be used and the total frame length will *not* necessarily be aligned at 8 bytes.
  * `peerMaxConcurrentStreams` {number} Sets the maximum number of concurrent streams for the remote peer as if a `SETTINGS` frame had been received. Will be overridden if the remote peer sets its own value for `maxConcurrentStreams`. **Default:** `100`.
  * `selectPadding` {Function} Cuando `options.paddingStrategy` es igual a `http2.constants.PADDING_STRATEGY_CALLBACK`, proporciona la función de callback utilizada para determinar el relleno. See [Using `options.selectPadding()`][].
  * `settings` {HTTP/2 Settings Object} Las configuraciones iniciales para enviar al peer remoto al conectarse.
  * `Http1IncomingMessage` {http.IncomingMessage} Specifies the `IncomingMessage` class to used for HTTP/1 fallback. Útil para extender el `http.IncomingMessage` original. **Default:** `http.IncomingMessage`.
  * `Http1ServerResponse` {http.ServerResponse} Specifies the `ServerResponse` class to used for HTTP/1 fallback. Útil para extender el `http.ServerResponse` original. **Default:** `http.ServerResponse`.
  * `Http2ServerRequest` {http2.Http2ServerRequest} Specifies the `Http2ServerRequest` class to use. Útil para extender el `Http2ServerRequest` original. **Default:** `Http2ServerRequest`.
  * `Http2ServerResponse` {http2.Http2ServerResponse} Specifies the `Http2ServerResponse` class to use. Útil para extender el `Http2ServerResponse` original. **Default:** `Http2ServerResponse`.
* `onRequestHandler` {Function} See [Compatibility API](#http2_compatibility_api)
* Returns: {Http2Server}

Devuelve una instancia de `net.Server` que crea y gestiona instancias de `Http2Session` .

Dado que no hay navegadores conocidos que soporten [unencrypted HTTP/2](https://http2.github.io/faq/#does-http2-require-encryption), el uso de [`http2.createSecureServer()`][] es necesario al comunicarse con los clientes del navegador.

```js
const http2 = require('http2');

// Create an unencrypted HTTP/2 server.
// Since there are no browsers known that support
// unencrypted HTTP/2, the use of `http2.createSecureServer()`
// is necessary when communicating with browser clients.
const server = http2.createServer();

server.on('stream', (stream, headers) => {
  stream.respond({
    'content-type': 'text/html',
    ':status': 200
  });
  stream.end('<h1>Hello World</h1>');
});

server.listen(80);
```

### http2.createSecureServer(options[, onRequestHandler])

<!-- YAML
added: v8.4.0
changes:

  - version: v8.9.3
    pr-url: https://github.com/nodejs/node/pull/17105
    description: Added the `maxOutstandingPings` option with a default limit of
                 10.
  - version: v8.9.3
    pr-url: https://github.com/nodejs/node/pull/16676
    description: Added the `maxHeaderListPairs` option with a default limit of
                 128 header pairs.
-->

* `opciones` {Object} 
  * `allowHTTP1` {boolean} Incoming client connections that do not support HTTP/2 will be downgraded to HTTP/1.x when set to `true`. See the [`'unknownProtocol'`][] event. Vea [ALPN negotiation](#http2_alpn_negotiation). **Default:** `false`.
  * `maxDeflateDynamicTableSize` {number} Sets the maximum dynamic table size for deflating header fields. **Default:** `4Kib`.
  * `maxSessionMemory`{number} Sets the maximum memory that the `Http2Session` is permitted to use. El valor se expresa en términos de número de megabytes, por ejemplo, `1` es igual a 1 megabyte. El valor mínimo permitido es `1`. This is a credit based limit, existing `Http2Stream`s may cause this limit to be exceeded, but new `Http2Stream` instances will be rejected while this limit is exceeded. The current number of `Http2Stream` sessions, the current memory use of the header compression tables, current data queued to be sent, and unacknowledged `PING` and `SETTINGS` frames are all counted towards the current limit. **Default:** `10`.
  * `maxHeaderListPairs` {number} Sets the maximum number of header entries. El valor mínimo es `4`. **Default:** `128`.
  * `maxOutstandingPings` {number} Sets the maximum number of outstanding, unacknowledged pings. **Default:** `10`.
  * `maxSendHeaderBlockLength` {number} Establece el tamaño máximo permitido para un bloque comprimido y serializado de encabezados. Attempts to send headers that exceed this limit will result in a `'frameError'` event being emitted and the stream being closed and destroyed.
  * `paddingStrategy` {number} Identifies the strategy used for determining the amount of padding to use for `HEADERS` and `DATA` frames. **Default:** `http2.constants.PADDING_STRATEGY_NONE`. Value may be one of: 
    * `http2.constants.PADDING_STRATEGY_NONE` - Specifies that no padding is to be applied.
    * `http2.constants.PADDING_STRATEGY_MAX` - Specifies that the maximum amount of padding, as determined by the internal implementation, is to be applied.
    * `http2.constants.PADDING_STRATEGY_CALLBACK` - Specifies that the user provided `options.selectPadding()` callback is to be used to determine the amount of padding.
    * `http2.constants.PADDING_STRATEGY_ALIGNED` - Will *attempt* to apply enough padding to ensure that the total frame length, including the 9-byte header, is a multiple of 8. For each frame, however, there is a maximum allowed number of padding bytes that is determined by current flow control state and settings. If this maximum is less than the calculated amount needed to ensure alignment, the maximum will be used and the total frame length will *not* necessarily be aligned at 8 bytes.
  * `peerMaxConcurrentStreams` {number} Sets the maximum number of concurrent streams for the remote peer as if a `SETTINGS` frame had been received. Will be overridden if the remote peer sets its own value for `maxConcurrentStreams`. **Default:** `100`.
  * `selectPadding` {Function} Cuando `options.paddingStrategy` es igual a `http2.constants.PADDING_STRATEGY_CALLBACK`, proporciona la función de callback utilizada para determinar el relleno. See [Using `options.selectPadding()`][].
  * `settings` {HTTP/2 Settings Object} Las configuraciones iniciales para enviar al peer remoto al conectarse.
  * ...: Any [`tls.createServer()`][] options can be provided. Para los servidores, usualmente se requieren las opciones de identidad (`pfx` ó `key`/`cert`).
* `onRequestHandler` {Function} See [Compatibility API](#http2_compatibility_api)
* Returns: {Http2SecureServer}

Devuelve una instancia de `tls.Server` que crea y gestiona instancias de `Http2Session` .

```js
const http2 = require('http2');

const options = {
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem')
};

// Create a secure HTTP/2 server
const server = http2.createSecureServer(options);

server.on('stream', (stream, headers) => {
  stream.respond({
    'content-type': 'text/html',
    ':status': 200
  });
  stream.end('<h1>Hello World</h1>');
});

server.listen(80);
```

### http2.connect(authority\[, options\]\[, listener\])

<!-- YAML
added: v8.4.0
changes:

  - version: v8.9.3
    pr-url: https://github.com/nodejs/node/pull/17105
    description: Added the `maxOutstandingPings` option with a default limit of
                 10.
  - version: v8.9.3
    pr-url: https://github.com/nodejs/node/pull/16676
    description: Added the `maxHeaderListPairs` option with a default limit of
                 128 header pairs.
-->

* `authority` {string|URL}
* `opciones` {Object} 
  * `maxDeflateDynamicTableSize` {number} Sets the maximum dynamic table size for deflating header fields. **Default:** `4Kib`.
  * `maxSessionMemory`{number} Sets the maximum memory that the `Http2Session` is permitted to use. El valor se expresa en términos de número de megabytes, por ejemplo, `1` es igual a 1 megabyte. El valor mínimo permitido es `1`. This is a credit based limit, existing `Http2Stream`s may cause this limit to be exceeded, but new `Http2Stream` instances will be rejected while this limit is exceeded. The current number of `Http2Stream` sessions, the current memory use of the header compression tables, current data queued to be sent, and unacknowledged `PING` and `SETTINGS` frames are all counted towards the current limit. **Default:** `10`.
  * `maxHeaderListPairs` {number} Sets the maximum number of header entries. El valor mínimo es `1`. **Default:** `128`.
  * `maxOutstandingPings` {number} Sets the maximum number of outstanding, unacknowledged pings. **Default:** `10`.
  * `maxReservedRemoteStreams` {number} Sets the maximum number of reserved push streams the client will accept at any given time. Once the current number of currently reserved push streams exceeds reaches this limit, new push streams sent by the server will be automatically rejected.
  * `maxSendHeaderBlockLength` {number} Establece el tamaño máximo permitido para un bloque comprimido y serializado de encabezados. Attempts to send headers that exceed this limit will result in a `'frameError'` event being emitted and the stream being closed and destroyed.
  * `paddingStrategy` {number} Identifies the strategy used for determining the amount of padding to use for `HEADERS` and `DATA` frames. **Default:** `http2.constants.PADDING_STRATEGY_NONE`. Value may be one of: 
    * `http2.constants.PADDING_STRATEGY_NONE` - Specifies that no padding is to be applied.
    * `http2.constants.PADDING_STRATEGY_MAX` - Specifies that the maximum amount of padding, as determined by the internal implementation, is to be applied.
    * `http2.constants.PADDING_STRATEGY_CALLBACK` - Specifies that the user provided `options.selectPadding()` callback is to be used to determine the amount of padding.
    * `http2.constants.PADDING_STRATEGY_ALIGNED` - Will *attempt* to apply enough padding to ensure that the total frame length, including the 9-byte header, is a multiple of 8. For each frame, however, there is a maximum allowed number of padding bytes that is determined by current flow control state and settings. If this maximum is less than the calculated amount needed to ensure alignment, the maximum will be used and the total frame length will *not* necessarily be aligned at 8 bytes.
  * `peerMaxConcurrentStreams` {number} Sets the maximum number of concurrent streams for the remote peer as if a `SETTINGS` frame had been received. Will be overridden if the remote peer sets its own value for `maxConcurrentStreams`. **Default:** `100`.
  * `selectPadding` {Function} Cuando `options.paddingStrategy` es igual a `http2.constants.PADDING_STRATEGY_CALLBACK`, proporciona la función de callback utilizada para determinar el relleno. See [Using `options.selectPadding()`][].
  * `settings` {HTTP/2 Settings Object} Las configuraciones iniciales para enviar al peer remoto al conectarse.
  * `createConnection` {Function} Un callback opcional que recibe la instancia de `URL` pasada a `connect` y el objeto de `options`, y devuelve cualquier stream de [`Duplex`][] que deberá ser utilizado como la conexión para esta sesión.
  * ...: Any [`net.connect()`][] or [`tls.connect()`][] options can be provided.
* `listener` {Function}
* Returns: {ClientHttp2Session}

Returns a `ClientHttp2Session` instance.

```js
const http2 = require('http2');
const client = http2.connect('https://localhost:1234');

/* Use the client */

client.close();
```

### http2.constants

<!-- YAML
added: v8.4.0
-->

#### Error Codes for RST_STREAM and GOAWAY

<a id="error_codes"></a>

| Valor  | Nombre                    | Constant                                      |
| ------ | ------------------------- | --------------------------------------------- |
| `0x00` | Sin errores               | `http2.constants.NGHTTP2_NO_ERROR`            |
| `0x01` | Error de protocolo        | `http2.constants.NGHTTP2_PROTOCOL_ERROR`      |
| `0x02` | Error interno             | `http2.constants.NGHTTP2_INTERNAL_ERROR`      |
| `0x03` | Error de control de flujo | `http2.constants.NGHTTP2_FLOW_CONTROL_ERROR`  |
| `0x04` | Settings Timeout          | `http2.constants.NGHTTP2_SETTINGS_TIMEOUT`    |
| `0x05` | Stream cerrado            | `http2.constants.NGHTTP2_STREAM_CLOSED`       |
| `0x06` | Error de tamaño de frame  | `http2.constants.NGHTTP2_FRAME_SIZE_ERROR`    |
| `0x07` | Stream negado             | `http2.constants.NGHTTP2_REFUSED_STREAM`      |
| `0x08` | Cancelar                  | `http2.constants.NGHTTP2_CANCEL`              |
| `0x09` | Compression Error         | `http2.constants.NGHTTP2_COMPRESSION_ERROR`   |
| `0x0a` | Error de conexión         | `http2.constants.NGHTTP2_CONNECT_ERROR`       |
| `0x0b` | Enhance Your Calm         | `http2.constants.NGHTTP2_ENHANCE_YOUR_CALM`   |
| `0x0c` | Seguridad inadecuada      | `http2.constants.NGHTTP2_INADEQUATE_SECURITY` |
| `0x0d` | HTTP/1.1 Required         | `http2.constants.NGHTTP2_HTTP_1_1_REQUIRED`   |

The `'timeout'` event is emitted when there is no activity on the Server for a given number of milliseconds set using `http2server.setTimeout()`.

### http2.getDefaultSettings()

<!-- YAML
added: v8.4.0
-->

* Returns: {HTTP/2 Settings Object}

Devuelve a un objeto que contiene las configuraciones predeterminadas para una instancia de `Http2Session` . This method returns a new object instance every time it is called so instances returned may be safely modified for use.

### http2.getPackedSettings(settings)

<!-- YAML
added: v8.4.0
-->

* `settings` {HTTP/2 Settings Object}
* Returns: {Buffer}

Returns a `Buffer` instance containing serialized representation of the given HTTP/2 settings as specified in the [HTTP/2](https://tools.ietf.org/html/rfc7540) specification. This is intended for use with the `HTTP2-Settings` header field.

```js
const http2 = require('http2');

const packed = http2.getPackedSettings({ enablePush: false });

console.log(packed.toString('base64'));
// Prints: AAIAAAAA
```

### http2.getUnpackedSettings(buf)

<!-- YAML
added: v8.4.0
-->

* `buf` {Buffer|Uint8Array} Las configuraciones empaquetadas.
* Returns: {HTTP/2 Settings Object}

Returns a [HTTP/2 Settings Object](#http2_settings_object) containing the deserialized settings from the given `Buffer` as generated by `http2.getPackedSettings()`.

### Headers Object

Los encabezados están representados como propiedades propias sobre los objetos de JavaScript. The property keys will be serialized to lower-case. Property values should be strings (if they are not they will be coerced to strings) or an `Array` of strings (in order to send more than one value per header field).

```js
const headers = {
  ':status': '200',
  'content-type': 'text-plain',
  'ABC': ['has', 'more', 'than', 'one', 'value']
};

stream.respond(headers);
```

Header objects passed to callback functions will have a `null` prototype. This means that normal JavaScript object methods such as `Object.prototype.toString()` and `Object.prototype.hasOwnProperty()` will not work.

```js
const http2 = require('http2');
const server = http2.createServer();
server.on('stream', (stream, headers) => {
  console.log(headers[':path']);
  console.log(headers.ABC);
});
```

### Settings Object

<!-- YAML
added: v8.4.0
changes:

  - version: v8.9.3
    pr-url: https://github.com/nodejs/node/pull/16676
    description: The `maxHeaderListSize` setting is now strictly enforced.
--> The 

`http2.getDefaultSettings()`, `http2.getPackedSettings()`, `http2.createServer()`, `http2.createSecureServer()`, `http2session.settings()`, `http2session.localSettings`, and `http2session.remoteSettings` APIs either return or receive as input an object that defines configuration settings for an `Http2Session` object. Estos objetos son objetos ordinarios de JavaScript que contienen las siguientes propiedades.

* `headerTableSize` {number} Especifica el número máximo de bytes utilizados para la compresión de encabezado. El valor mínimo permitido es 0. The maximum allowed value is 2<sup>32</sup>-1. **Default:** `4,096 octets`.
* `enablePush` {boolean} Specifies `true` if HTTP/2 Push Streams are to be permitted on the `Http2Session` instances.
* `initialWindowSize` {number} Specifies the *senders* initial window size for stream-level flow control. El valor mínimo permitido es 0. The maximum allowed value is 2<sup>32</sup>-1. **Default:** `65,535 bytes`.
* `maxFrameSize` {number} Specifies the size of the largest frame payload. El valor mínimo permitido es 16,384. The maximum allowed value is 2<sup>24</sup>-1. **Default:** `16,384 bytes`.
* `maxConcurrentStreams` {number} Specifies the maximum number of concurrent streams permitted on an `Http2Session`. There is no default value which implies, at least theoretically, 2<sup>31</sup>-1 streams may be open concurrently at any given time in an `Http2Session`. El valor mínimo es 0. The maximum allowed value is 2<sup>31</sup>-1.
* `maxHeaderListSize` {number} Specifies the maximum size (uncompressed octets) of header list that will be accepted. El valor mínimo permitido es 0. The maximum allowed value is 2<sup>32</sup>-1. **Default:** `65535`.

Se ignoran todas las propiedades adicionales del objeto de las configuraciones.

### Using `options.selectPadding()`

Cuando `options.paddingStrategy` es igual a `http2.constants.PADDING_STRATEGY_CALLBACK`, la implementación de HTTP/2 consultará la función de callback de `options.selectPadding()`, si es proporcionada, para determinar la cantidad específica de relleno a usar por `HEADERS` y frame de `DATA` .

La función de `options.selectPadding()` recibe dos argumentos numéricos, `frameLen` y `maxFrameLen` y debe devolver un número `N` de modo que `frameLen <= N <= maxFrameLen`.

```js
const http2 = require('http2');
const server = http2.createServer({
  paddingStrategy: http2.constants.PADDING_STRATEGY_CALLBACK,
  selectPadding(frameLen, maxFrameLen) {
    return maxFrameLen;
  }
});
```

The `options.selectPadding()` function is invoked once for *every* `HEADERS` and `DATA` frame. Esto tiene un definido impacto notable sobre el rendimiento.

### Error Handling

Hay varios tipos de condiciones de error que pueden surgir al utilizar el módulo de `http2` :

Validation errors occur when an incorrect argument, option, or setting value is passed in. Estos siempre serán reportados por un `throw` sincrónico.

State errors occur when an action is attempted at an incorrect time (for instance, attempting to send data on a stream after it has closed). Estos serán reportados utilizando un `throw` sincrónico o por medio de un evento de `'error'` en el `Http2Stream`, `Http2Session` o en los objetos del Servidor de HTTP/2, dependiendo de dónde y cuándo ocurran los errores.

Los errores internos ocurren cuando una sesión HTTP/2 falla inesperadamente. Estos serán reportados por medio de un evento de `'error'` en el `Http2Session` u objetos del servidor de HTTP/2.

Protocol errors occur when various HTTP/2 protocol constraints are violated. Estos serán reportados utilizando un `throw` sincrónico o por medio de un evento de `'error'` en el `Http2Stream`, `Http2Session` o en los objetos del Servidor de HTTP/2, dependiendo de dónde y cuándo ocurran los errores.

### Invalid character handling in header names and values

The HTTP/2 implementation applies stricter handling of invalid characters in HTTP header names and values than the HTTP/1 implementation.

Header field names are *case-insensitive* and are transmitted over the wire strictly as lower-case strings. La API proporcionada por Node.js permite que los nombres de los encabezados sean establecidos como strings con mayúsculas y minúsculas (por ejemplo, `Content-Type`) pero los convertirá a minúsculas (por ejemplo, `content-type`) al ser transmitidos.

Header field-names *must only* contain one or more of the following ASCII characters: `a`-`z`, `A`-`Z`, `0`-`9`, `!`, `#`, `$`, `%`, `&`, `'`, `*`, `+`, `-`, `.`, `^`, `_`, `` ` `` (backtick), `|`, and `~`.

Using invalid characters within an HTTP header field name will cause the stream to be closed with a protocol error being reported.

Header field values are handled with more leniency but *should* not contain new-line or carriage return characters and *should* be limited to US-ASCII characters, per the requirements of the HTTP specification.

### Push streams on the client

To receive pushed streams on the client, set a listener for the `'stream'` event on the `ClientHttp2Session`:

```js
const http2 = require('http2');

const client = http2.connect('http://localhost');

client.on('stream', (pushedStream, requestHeaders) => {
  pushedStream.on('push', (responseHeaders) => {
    // process response headers
  });
  pushedStream.on('data', (chunk) => { /* handle pushed data */ });
});

const req = client.request({ ':path': '/' });
```

### Supporting the CONNECT method

The `CONNECT` method is used to allow an HTTP/2 server to be used as a proxy for TCP/IP connections.

A simple TCP Server:

```js
const net = require('net');

const server = net.createServer((socket) => {
  let name = '';
  socket.setEncoding('utf8');
  socket.on('data', (chunk) => name += chunk);
  socket.on('end', () => socket.end(`hello ${name}`));
});

server.listen(8000);
```

An HTTP/2 CONNECT proxy:

```js
const http2 = require('http2');
const { NGHTTP2_REFUSED_STREAM } = http2.constants;
const net = require('net');

const proxy = http2.createServer();
proxy.on('stream', (stream, headers) => {
  if (headers[':method'] !== 'CONNECT') {
    // Only accept CONNECT requests
    stream.close(NGHTTP2_REFUSED_STREAM);
    return;
  }
  const auth = new URL(`tcp://${headers[':authority']}`);
  // It's a very good idea to verify that hostname and port are
  // things this proxy should be connecting to.
  const socket = net.connect(auth.port, auth.hostname, () => {
    stream.respond();
    socket.pipe(stream);
    stream.pipe(socket);
  });
  socket.on('error', (error) => {
    stream.close(http2.constants.NGHTTP2_CONNECT_ERROR);
  });
});

proxy.listen(8001);
```

An HTTP/2 CONNECT client:

```js
const http2 = require('http2');

const client = http2.connect('http://localhost:8001');

// Must not specify the ':path' and ':scheme' headers
// for CONNECT requests or an error will be thrown.
const req = client.request({
  ':method': 'CONNECT',
  ':authority': `localhost:${port}`
});

req.on('response', (headers) => {
  console.log(headers[http2.constants.HTTP2_HEADER_STATUS]);
});
let data = '';
req.setEncoding('utf8');
req.on('data', (chunk) => data += chunk);
req.on('end', () => {
  console.log(`The server says: ${data}`);
  client.close();
});
req.end('Jane');
```

## API de compatibilidad

La API de Compatibilidad tiene el objetivo de proporcionar una experiencia para el desarrollador similar a HTTP/1 al utilizar HTTP/2, haciendo posible el desarrollo de aplicaciones que soporten [HTTP/1](http.html) y HTTP/2. Esta API sólo se dirige a la **public API** de [HTTP/1](http.html). However many modules use internal methods or state, and those *are not supported* as it is a completely different implementation.

The following example creates an HTTP/2 server using the compatibility API:

```js
const http2 = require('http2');
const server = http2.createServer((req, res) => {
  res.setHeader('Content-Type', 'text/html');
  res.setHeader('X-Foo', 'bar');
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('ok');
});
```

In order to create a mixed [HTTPS](https.html) and HTTP/2 server, refer to the [ALPN negotiation](#http2_alpn_negotiation) section. Upgrading from non-tls HTTP/1 servers is not supported.

The HTTP/2 compatibility API is composed of [`Http2ServerRequest`]() and [`Http2ServerResponse`](). They aim at API compatibility with HTTP/1, but they do not hide the differences between the protocols. Por ejemplo, se ignora el mensaje de estado para los códigos de HTTP.

### ALPN negotiation

ALPN negotiation allows supporting both [HTTPS](https.html) and HTTP/2 over the same socket. Los objetos `req` y `res` pueden ser HTTP/1 ó HTTP/2, y una aplicación **must** se limita a la API pública de [HTTP/1](http.html), y detecta si es posible utilizar las funciones más avanzadas de HTTP/2.

El siguiente ejemplo crea un servidor que soporta a ambos protocolos:

```js
const { createSecureServer } = require('http2');
const { readFileSync } = require('fs');

const cert = readFileSync('./cert.pem');
const key = readFileSync('./key.pem');

const server = createSecureServer(
  { cert, key, allowHTTP1: true },
  onRequest
).listen(4443);

function onRequest(req, res) {
  // detects if it is a HTTPS request or HTTP/2
  const { socket: { alpnProtocol } } = req.httpVersion === '2.0' ?
    req.stream.session : req;
  res.writeHead(200, { 'content-type': 'application/json' });
  res.end(JSON.stringify({
    alpnProtocol,
    httpVersion: req.httpVersion
  }));
}
```

El evento de `'request'` funciona de manera idéntica en [HTTPS](https.html) y HTTP/2.

### Class: http2.Http2ServerRequest

<!-- YAML
added: v8.4.0
-->

A `Http2ServerRequest` object is created by [`http2.Server`][] or [`http2.SecureServer`][] and passed as the first argument to the [`'request'`][] event. Puede ser utilizado para acceder a un estado de solicitud, encabezados, y datos.

It implements the [Readable Stream](stream.html#stream_class_stream_readable) interface, as well as the following additional events, methods, and properties.

#### Event: 'aborted'

<!-- YAML
added: v8.4.0
-->

The `'aborted'` event is emitted whenever a `Http2ServerRequest` instance is abnormally aborted in mid-communication.

The `'aborted'` event will only be emitted if the `Http2ServerRequest` writable side has not been ended.

#### Event: 'close'

<!-- YAML
added: v8.4.0
-->

Indica que el [`Http2Stream`][] subyacente fue cerrado. Just like `'end'`, this event occurs only once per response.

#### request.aborted

<!-- YAML
added: v10.1.0
-->

* {boolean}

La propiedad de `request.aborted` será `true` si la solicitud ha sido anulada.

#### request.destroy([error])

<!-- YAML
added: v8.4.0
-->

* `error` {Error}

Llama a `destroy()` en el [`Http2Stream`][] que recibió el [`Http2ServerRequest`][]. If `error` is provided, an `'error'` event is emitted and `error` is passed as an argument to any listeners on the event.

No hace nada si el stream ya fue destruido.

#### request.headers

<!-- YAML
added: v8.4.0
-->

* {Object}

The request/response headers object.

Key-value pairs of header names and values. Los nombres de los encabezados están en minúsculas. Ejemplo:

```js
// Prints something like:
//
// { 'user-agent': 'curl/7.22.0',
//   host: '127.0.0.1:8000',
//   accept: '*/*' }
console.log(request.headers);
```

Vea [HTTP/2 Headers Object](#http2_headers_object).

In HTTP/2, the request path, hostname, protocol, and method are represented as special headers prefixed with the `:` character (e.g. `':path'`). Estos encabezados especiales serán incluidos en el objeto de `request.headers` . Care must be taken not to inadvertently modify these special headers or errors may occur. Por ejemplo, remover todos los encabezados de la solicitud ocasionará que ocurran errores:

```js
removeAllHeaders(request.headers);
assert(request.url);   // Fails because the :path header has been removed
```

#### request.httpVersion

<!-- YAML
added: v8.4.0
-->

* {string}

In case of server request, the HTTP version sent by the client. En el caso de la respuesta del cliente, la versión HTTP del servidor conectado al servidor. Returns `'2.0'`.

Also `message.httpVersionMajor` is the first integer and `message.httpVersionMinor` is the second.

#### request.method

<!-- YAML
added: v8.4.0
-->

* {string}

The request method as a string. Sólo lectura. Example: `'GET'`, `'DELETE'`.

#### request.rawHeaders

<!-- YAML
added: v8.4.0
-->

* {string[]}

The raw request/response headers list exactly as they were received.

Tenga en cuenta que las claves y los valores están en la misma lista. It is *not* a list of tuples. So, the even-numbered offsets are key values, and the odd-numbered offsets are the associated values.

Los nombres de los encabezados no están en minúsculas, y los duplicados no están fusionados.

```js
// Prints something like:
//
// [ 'user-agent',
//   'this is invalid because there can be only one',
//   'User-Agent',
//   'curl/7.22.0',
//   'Host',
//   '127.0.0.1:8000',
//   'ACCEPT',
//   '*/*' ]
console.log(request.rawHeaders);
```

#### request.rawTrailers

<!-- YAML
added: v8.4.0
-->

* {string[]}

The raw request/response trailer keys and values exactly as they were received. Only populated at the `'end'` event.

#### request.setTimeout(msecs, callback)

<!-- YAML
added: v8.4.0
-->

* `msecs` {number}
* `callback` {Function}
* Returns: {http2.Http2ServerRequest}

Sets the [`Http2Stream`]()'s timeout value to `msecs`. If a callback is provided, then it is added as a listener on the `'timeout'` event on the response object.

If no `'timeout'` listener is added to the request, the response, or the server, then [`Http2Stream`]()s are destroyed when they time out. If a handler is assigned to the request, the response, or the server's `'timeout'` events, timed out sockets must be handled explicitly.

#### request.socket

<!-- YAML
added: v8.4.0
-->

* {net.Socket|tls.TLSSocket}

Returns a `Proxy` object that acts as a `net.Socket` (or `tls.TLSSocket`) but applies getters, setters, and methods based on HTTP/2 logic.

`destroyed`, `readable`, and `writable` properties will be retrieved from and set on `request.stream`.

`destroy`, `emit`, `end`, `on` and `once` methods will be called on `request.stream`.

`setTimeout` method will be called on `request.stream.session`.

`pause`, `read`, `resume`, and `write` will throw an error with code `ERR_HTTP2_NO_SOCKET_MANIPULATION`. Vea [`Http2Session` y Sockets][] para más información.

All other interactions will be routed directly to the socket. With TLS support, use [`request.socket.getPeerCertificate()`][] to obtain the client's authentication details.

#### request.stream

<!-- YAML
added: v8.4.0
-->

* {Http2Stream}

The [`Http2Stream`][] object backing the request.

#### request.trailers

<!-- YAML
added: v8.4.0
-->

* {Object}

The request/response trailers object. Only populated at the `'end'` event.

#### request.url

<!-- YAML
added: v8.4.0
-->

* {string}

Solicitar string de URL. Esto sólo contiene la URL que está presente en la solicitud de HTTP actual. Si la solicitud es:

```txt
GET /status?name=ryan HTTP/1.1\r\n
Accept: text/plain\r\n
\r\n
```

Entonces `request.url` será:

<!-- eslint-disable semi -->

```js
'/status?name=ryan'
```

To parse the url into its parts `require('url').parse(request.url)` can be used. Ejemplo:

```txt
$ node
> require('url').parse('/status?name=ryan')
Url {
  protocol: null,
  slashes: null,
  auth: null,
  host: null,
  port: null,
  hostname: null,
  hash: null,
  search: '?name=ryan',
  query: 'name=ryan',
  pathname: '/status',
  path: '/status?name=ryan',
  href: '/status?name=ryan' }
```

To extract the parameters from the query string, the `require('querystring').parse` function can be used, or `true` can be passed as the second argument to `require('url').parse`. Ejemplo:

```txt
$ node
> require('url').parse('/status?name=ryan', true)
Url {
  protocol: null,
  slashes: null,
  auth: null,
  host: null,
  port: null,
  hostname: null,
  hash: null,
  search: '?name=ryan',
  query: { name: 'ryan' },
  pathname: '/status',
  path: '/status?name=ryan',
  href: '/status?name=ryan' }
```

### Class: http2.Http2ServerResponse

<!-- YAML
added: v8.4.0
-->

This object is created internally by an HTTP server — not by the user. Se pasa como el segundo parámetro al evento de [`'request'`][].

La respuesta implementa, pero no hereda, la interfaz de [Writable Stream](stream.html#stream_writable_streams) . Esto es un [`EventEmitter`][] con los siguientes eventos:

#### Event: 'close'

<!-- YAML
added: v8.4.0
-->

Indicates that the underlying [`Http2Stream`]() was terminated before [`response.end()`][] was called or able to flush.

#### Event: 'finish'

<!-- YAML
added: v8.4.0
-->

Emitido cuando la respuesta ha sido enviada. More specifically, this event is emitted when the last segment of the response headers and body have been handed off to the HTTP/2 multiplexing for transmission over the network. Eso no implica que el cliente no ha recibido nada aún.

Después de este evento, no se emitirán más eventos en el objeto de respuesta.

#### response.addTrailers(headers)

<!-- YAML
added: v8.4.0
-->

* `headers` {Object}

This method adds HTTP trailing headers (a header but at the end of the message) to the response.

Attempting to set a header field name or value that contains invalid characters will result in a [`TypeError`][] being thrown.

#### response.connection

<!-- YAML
added: v8.4.0
-->

* {net.Socket|tls.TLSSocket}

See [`response.socket`][].

#### response.end(\[data\]\[, encoding\][, callback])

<!-- YAML
added: v8.4.0
changes:

  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18780
    description: This method now returns a reference to `ServerResponse`.
-->

* `data` {string|Buffer}
* `encoding` {string}
* `callback` {Function}
* Returns: {this}

This method signals to the server that all of the response headers and body have been sent; that server should consider this message complete. Este método, `response.end()`, DEBE ser llamado en cada respuesta.

If `data` is specified, it is equivalent to calling [`response.write(data, encoding)`][] followed by `response.end(callback)`.

Si se especifica el `callback`, será llamado cuando el stream de respuesta haya finalizado.

#### response.finished

<!-- YAML
added: v8.4.0
-->

* {boolean}

Boolean value that indicates whether the response has completed. Starts as `false`. After [`response.end()`][] executes, the value will be `true`.

#### response.getHeader(name)

<!-- YAML
added: v8.4.0
-->

* `name` {string}
* Returns: {string}

Reads out a header that has already been queued but not sent to the client. Note that the name is case insensitive.

Ejemplo:

```js
const contentType = response.getHeader('content-type');
```

#### response.getHeaderNames()

<!-- YAML
added: v8.4.0
-->

* Returns: {string[]}

Returns an array containing the unique names of the current outgoing headers. Todos los nombres de los encabezados están en minúsculas.

Ejemplo:

```js
response.setHeader('Foo', 'bar');
response.setHeader('Set-Cookie', ['foo=bar', 'bar=baz']);

const headerNames = response.getHeaderNames();
// headerNames === ['foo', 'set-cookie']
```

#### response.getHeaders()

<!-- YAML
added: v8.4.0
-->

* Returns: {Object}

Returns a shallow copy of the current outgoing headers. Since a shallow copy is used, array values may be mutated without additional calls to various header-related http module methods. Las claves del objeto devuelto son los nombres de encabezado y los valores de los respectivos valores de encabezado. Todos los nombres de los encabezados están en minúsculas.

The object returned by the `response.getHeaders()` method *does not* prototypically inherit from the JavaScript `Object`. Esto significa que métodos típicos de `Object` tales como `obj.toString()`, `obj.hasOwnProperty()`, entre otros, no están definidos y *will not work*.

Ejemplo:

```js
response.setHeader('Foo', 'bar');
response.setHeader('Set-Cookie', ['foo=bar', 'bar=baz']);

const headers = response.getHeaders();
// headers === { foo: 'bar', 'set-cookie': ['foo=bar', 'bar=baz'] }
```

#### response.hasHeader(name)

<!-- YAML
added: v8.4.0
-->

* `name` {string}
* Returns: {boolean}

Devuelve `true` si el encabezado identificado por `name` está actualmente establecido en los encabezados salientes. Note that the header name matching is case-insensitive.

Ejemplo:

```js
const hasContentType = response.hasHeader('content-type');
```

#### response.headersSent

<!-- YAML
added: v8.4.0
-->

* {boolean}

Verdadero si los encabezados fueron enviados, de lo contrario falso (sólo lectura).

#### response.removeHeader(name)

<!-- YAML
added: v8.4.0
-->

* `name` {string}

Elimina un encabezado que ha sido puesto en cola para un envío implícito.

Ejemplo:

```js
response.removeHeader('Content-Encoding');
```

#### response.sendDate

<!-- YAML
added: v8.4.0
-->

* {boolean}

Al ser verdadero, la Fecha del encabezado será generada automáticamente y enviada en la respuesta si no está presente en los encabezados. Defaults to true.

This should only be disabled for testing; HTTP requires the Date header in responses.

#### response.setHeader(name, value)

<!-- YAML
added: v8.4.0
-->

* `name` {string}
* `value` {string|string[]}

Sets a single header value for implicit headers. Si este encabezado ya existe en los envíos de encabezados pendientes, su valor será reemplazado. Use an array of strings here to send multiple headers with the same name.

Ejemplo:

```js
response.setHeader('Content-Type', 'text/html');
```

o

```js
response.setHeader('Set-Cookie', ['type=ninja', 'language=javascript']);
```

Attempting to set a header field name or value that contains invalid characters will result in a [`TypeError`][] being thrown.

When headers have been set with [`response.setHeader()`][], they will be merged with any headers passed to [`response.writeHead()`][], with the headers passed to [`response.writeHead()`][] given precedence.

```js
// returns content-type = text/plain
const server = http2.createServer((req, res) => {
  res.setHeader('Content-Type', 'text/html');
  res.setHeader('X-Foo', 'bar');
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('ok');
});
```

#### response.setTimeout(msecs[, callback])

<!-- YAML
added: v8.4.0
-->

* `msecs` {number}
* `callback` {Function}
* Returns: {http2.Http2ServerResponse}

Sets the [`Http2Stream`]()'s timeout value to `msecs`. If a callback is provided, then it is added as a listener on the `'timeout'` event on the response object.

If no `'timeout'` listener is added to the request, the response, or the server, then [`Http2Stream`]()s are destroyed when they time out. If a handler is assigned to the request, the response, or the server's `'timeout'` events, timed out sockets must be handled explicitly.

#### response.socket

<!-- YAML
added: v8.4.0
-->

* {net.Socket|tls.TLSSocket}

Returns a `Proxy` object that acts as a `net.Socket` (or `tls.TLSSocket`) but applies getters, setters, and methods based on HTTP/2 logic.

`destroyed`, `readable`, and `writable` properties will be retrieved from and set on `response.stream`.

`destroy`, `emit`, `end`, `on` and `once` methods will be called on `response.stream`.

`setTimeout` method will be called on `response.stream.session`.

`pause`, `read`, `resume`, and `write` will throw an error with code `ERR_HTTP2_NO_SOCKET_MANIPULATION`. Vea [`Http2Session` y Sockets][] para más información.

All other interactions will be routed directly to the socket.

Ejemplo:

```js
const http2 = require('http2');
const server = http2.createServer((req, res) => {
  const ip = req.socket.remoteAddress;
  const port = req.socket.remotePort;
  res.end(`Your IP address is ${ip} and your source port is ${port}.`);
}).listen(3000);
```

#### response.statusCode

<!-- YAML
added: v8.4.0
-->

* {number}

When using implicit headers (not calling [`response.writeHead()`][] explicitly), this property controls the status code that will be sent to the client when the headers get flushed.

Ejemplo:

```js
response.statusCode = 404;
```

Después de que el encabezado de respuesta fue enviado al cliente, esta propiedad indica el código de estado que fue enviado.

#### response.statusMessage

<!-- YAML
added: v8.4.0
-->

* {string}

Mensaje de estado no es soportado por HTTP/2 (RFC7540 8.1.2.4). Devuelve una string vacía.

#### response.stream

<!-- YAML
added: v8.4.0
-->

* {Http2Stream}

The [`Http2Stream`][] object backing the response.

#### response.write(chunk\[, encoding\]\[, callback\])

<!-- YAML
added: v8.4.0
-->

* `chunk` {string|Buffer}
* `encoding` {string}
* `callback` {Function}
* Returns: {boolean}

If this method is called and [`response.writeHead()`][] has not been called, it will switch to implicit header mode and flush the implicit headers.

Esto envía una parte del cuerpo de la respuesta. Este método puede ser llamado varias veces para proporcionar partes sucesivas del cuerpo.

Note that in the `http` module, the response body is omitted when the request is a HEAD request. Similarly, the `204` and `304` responses *must not* include a message body.

`chunk` can be a string or a buffer. Si `chunk` es una string, el segundo parámetro especificará cómo codificarlo dentro de un stream de bytes. Por defecto, el `encoding` es `'utf8'`. `callback` will be called when this chunk of data is flushed.

This is the raw HTTP body and has nothing to do with higher-level multi-part body encodings that may be used.

The first time [`response.write()`][] is called, it will send the buffered header information and the first chunk of the body to the client. The second time [`response.write()`][] is called, Node.js assumes data will be streamed, and sends the new data separately. That is, the response is buffered up to the first chunk of the body.

Returns `true` if the entire data was flushed successfully to the kernel buffer. Returns `false` if all or part of the data was queued in user memory. `'drain'` will be emitted when the buffer is free again.

#### response.writeContinue()

<!-- YAML
added: v8.4.0
-->

Envía un estado `100 Continue` al cliente, indicando que el cuerpo de solicitud debería ser enviado. See the [`'checkContinue'`][] event on `Http2Server` and `Http2SecureServer`.

#### response.writeHead(statusCode\[, statusMessage\]\[, headers\])

<!-- YAML
added: v8.4.0
-->

* `statusCode` {number}
* `statusMessage` {string}
* `headers` {Object}

Envía un encabezado de respuesta a la solicitud. El código de estado es un código de estado de 3 dígitos, como el `404`. El último argumento, `headers`, son los encabezados de respuesta.

For compatibility with [HTTP/1](http.html), a human-readable `statusMessage` may be passed as the second argument. However, because the `statusMessage` has no meaning within HTTP/2, the argument will have no effect and a process warning will be emitted.

Ejemplo:

```js
const body = 'hello world';
response.writeHead(200, {
  'Content-Length': Buffer.byteLength(body),
  'Content-Type': 'text/plain' });
```

Tenga en cuenta que la longitud del contenido se da en bytes, no en caracteres. La API de `Buffer.byteLength()` puede ser utilizada para determinar el número de bytes en una codificación dada. On outbound messages, Node.js does not check if Content-Length and the length of the body being transmitted are equal or not. However, when receiving messages, Node.js will automatically reject messages when the Content-Length does not match the actual payload size.

Este método puede ser llamado una vez en un mensaje, como máximo, antes de que [`response.end()`][] sea llamado.

If [`response.write()`][] or [`response.end()`][] are called before calling this, the implicit/mutable headers will be calculated and call this function.

When headers have been set with [`response.setHeader()`][], they will be merged with any headers passed to [`response.writeHead()`][], with the headers passed to [`response.writeHead()`][] given precedence.

```js
// returns content-type = text/plain
const server = http2.createServer((req, res) => {
  res.setHeader('Content-Type', 'text/html');
  res.setHeader('X-Foo', 'bar');
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('ok');
});
```

Attempting to set a header field name or value that contains invalid characters will result in a [`TypeError`][] being thrown.

#### response.createPushResponse(headers, callback)

<!-- YAML
added: v8.4.0
-->

Call [`http2stream.pushStream()`][] with the given headers, and wraps the given newly created [`Http2Stream`] on `Http2ServerRespose`.

El callback será llamado con un error con código `ERR_HTTP2_STREAM_CLOSED` si se cierra el stream.

## Collecting HTTP/2 Performance Metrics

The [Performance Observer](perf_hooks.html) API can be used to collect basic performance metrics for each `Http2Session` and `Http2Stream` instance.

```js
const { PerformanceObserver } = require('perf_hooks');

const obs = new PerformanceObserver((items) => {
  const entry = items.getEntries()[0];
  console.log(entry.entryType);  // prints 'http2'
  if (entry.name === 'Http2Session') {
    // entry contains statistics about the Http2Session
  } else if (entry.name === 'Http2Stream') {
    // entry contains statistics about the Http2Stream
  }
});
obs.observe({ entryTypes: ['http2'] });
```

La propiedad de `entryType` de la `PerformanceEntry` será igual a `'http2'`.

The `name` property of the `PerformanceEntry` will be equal to either `'Http2Stream'` or `'Http2Session'`.

If `name` is equal to `Http2Stream`, the `PerformanceEntry` will contain the following additional properties:

* `bytesRead` {number} The number of `DATA` frame bytes received for this `Http2Stream`.
* `bytesWritten` {number} The number of `DATA` frame bytes sent for this `Http2Stream`.
* `id` {number} El identificador del `Http2Stream` asociado
* `timeToFirstByte` {number} The number of milliseconds elapsed between the `PerformanceEntry` `startTime` and the reception of the first `DATA` frame.
* `timeToFirstByteSent` {number} The number of milliseconds elapsed between the `PerformanceEntry` `startTime` and sending of the first `DATA` frame.
* `timeToFirstHeader` {number} El número de milisegundos transcurridos entre el `PerformanceEntry` `startTime` y la recepción del primer encabezado.

If `name` is equal to `Http2Session`, the `PerformanceEntry` will contain the following additional properties:

* `bytesRead` {number} El número de bytes recibidos para este `Http2Session`.
* `bytesWritten` {number} El número de bytes enviados para este `Http2Session`.
* `framesReceived` {number} The number of HTTP/2 frames received by the `Http2Session`.
* `framesSent` {number} The number of HTTP/2 frames sent by the `Http2Session`.
* `maxConcurrentStreams` {number} The maximum number of streams concurrently open during the lifetime of the `Http2Session`.
* `pingRTT` {number} The number of milliseconds elapsed since the transmission of a `PING` frame and the reception of its acknowledgment. Only present if a `PING` frame has been sent on the `Http2Session`.
* `streamAverageDuration` {number} La duración promedio (en milisegundos) para todas las instancias de `Http2Stream` .
* `streamCount` {number} El número de instancias de `Http2Stream` procesadas por la `Http2Session`.
* `type` {string} Either `'server'` or `'client'` to identify the type of `Http2Session`.