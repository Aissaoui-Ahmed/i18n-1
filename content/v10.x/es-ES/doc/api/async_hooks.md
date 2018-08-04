# Async Hooks

<!--introduced_in=v8.1.0-->

> Estabilidad: 1 - Experimental

El módulo de `async_hooks` proporciona una API para registrar los callbacks que rastrean el tiempo de vida de recursos asíncronos creados dentro de una aplicación de Node.js. Puede ser accedido utilizando:

```js
const async_hooks = require('async_hooks');
```

## Terminología

Un recurso asíncrono representa un objeto con un callback asociado. Este callback se puede llamar varias veces, por ejemplo, el evento `'connection'` en `net.createServer()`, o simplemente una sóla vez como en `fs.open()`. Un recurso también puede cerrarse antes de que se llame un callback. `AsyncHook` no distingue explícitamente entre estos casos diferentes, pero los representará como el concepto abstracto que es un recurso.

## API Pública

### Resumen

A continuación, está un simple resumen de la API pública.

```js
const async_hooks = require('async_hooks');

// Return the ID of the current execution context.
const eid = async_hooks.executionAsyncId();

// Return the ID of the handle responsible for triggering the callback of the
// current execution scope to call.
const tid = async_hooks.triggerAsyncId();

// Create a new AsyncHook instance. Todos estos callbacks son opcionales.
const asyncHook =
    async_hooks.createHook({ init, before, after, destroy, promiseResolve });

// Allow callbacks of this AsyncHook instance to call. This is not an implicit
// action after running the constructor, and must be explicitly run to begin
// executing callbacks.
asyncHook.enable();

// Disable listening for new asynchronous events.
asyncHook.disable();

//
// The following are the callbacks that can be passed to createHook().
//

// init is called during object construction. The resource may not have
// completed construction when this callback runs, therefore all fields of the
// resource referenced by "asyncId" may not have been populated.
function init(asyncId, type, triggerAsyncId, resource) { }

// before is called just before the resource's callback is called. It can be
// called 0-N times for handles (e.g. TCPWrap), and will be called exactly 1
// time for requests (e.g. FSReqWrap).
function before(asyncId) { }

// after is called just after the resource's callback has finished.
function after(asyncId) { }

// destroy is called when an AsyncWrap instance is destroyed.
function destroy(asyncId) { }

// promiseResolve is called only for promise resources, when the
// `resolve` function passed to the `Promise` constructor is invoked
// (either directly or through other means of resolving a promise).
function promiseResolve(asyncId) { }
```

#### async_hooks.createHook(callbacks)

<!-- YAML
added: v8.1.0
-->

* `callbacks` {Object} The [Hook Callbacks](#async_hooks_hook_callbacks) to register 
  * `init` {Function} The [`init` callback][].
  * `before` {Function} The [`before` callback][].
  * `after` {Function} The [`after` callback][].
  * `destroy` {Function} The [`destroy` callback][].
* Devoluciones: {AsyncHook} Instancia utilizada para inhabilitar y habilitar hooks

Registra funciones para que sean llamadas por diferentes eventos en el tiempo de vida de cada operación asincrónica.

Los callbacks `init()`/`before()`/`after()`/`destroy()` son llamados para el respectivo evento asincrónico durante el tiempo de vida de un recurso.

Todos los callbacks son opcionales. Por ejemplo, si solamente se necesita rastrear la limpieza de recursos, entonces sólo se necesitará pasar el callback `destroy` . Las especificaciones de todas las funciones que pueden ser pasadas a `callbacks` están en la sección [Hook Callbacks](#async_hooks_hook_callbacks) .

```js
const async_hooks = require('async_hooks');

const asyncHook = async_hooks.createHook({
  init(asyncId, type, triggerAsyncId, resource) { },
  destroy(asyncId) { }
});
```

Tenga en cuenta que los callbacks serán heredados por medio de la cadena del prototipo:

```js
class MyAsyncCallbacks {
  init(asyncId, type, triggerAsyncId, resource) { }
  destroy(asyncId) {}
}

class MyAddedCallbacks extends MyAsyncCallbacks {
  before(asyncId) { }
  after(asyncId) { }
}

const asyncHook = async_hooks.createHook(new MyAddedCallbacks());
```

##### Error Handling

If any `AsyncHook` callbacks throw, the application will print the stack trace and exit. The exit path does follow that of an uncaught exception, but all `'uncaughtException'` listeners are removed, thus forcing the process to exit. The `'exit'` callbacks will still be called unless the application is run with `--abort-on-uncaught-exception`, in which case a stack trace will be printed and the application exits, leaving a core file.

El motivo de este comportamiento de manejo de error es que estos callbacks se ejecutan en puntos potencialmente volátiles en el tiempo de vida de un objeto, por ejemplo, durante la construcción y destrucción de una clase. Debido a esto, se considera necesario reducir el proceso rápidamente para evitar una anulación no intencionada en el futuro. Esto está sujeto a cambio en el futuro si un análisis comprensivo se realiza para asegurar que una excepción puede seguir el flujo de control normal sin efectos secundarios no intencionados.

##### Printing in AsyncHooks callbacks

Because printing to the console is an asynchronous operation, `console.log()` will cause the AsyncHooks callbacks to be called. Using `console.log()` or similar asynchronous operations inside an AsyncHooks callback function will thus cause an infinite recursion. An easy solution to this when debugging is to use a synchronous logging operation such as `fs.writeSync(1, msg)`. This will print to stdout because `1` is the file descriptor for stdout and will not invoke AsyncHooks recursively because it is synchronous.

```js
const fs = require('fs');
const util = require('util');

function debug(...args) {
  // use a function like this one when debugging inside an AsyncHooks callback
  fs.writeSync(1, `${util.format(...args)}\n`);
}
```

Si se necesita una operación asincrónica para registrarse, es posible realizar un seguimiento de lo que causó la operación asincrónica que utiliza la información proporcionada por AsyncHooks. El registro debe omitirse cuando era el registro en sí el que causaba que el callback de AsyncHooks llamara. By doing this the otherwise infinite recursion is broken.

#### asyncHook.enable()

* Returns: {AsyncHook} A reference to `asyncHook`.

Habilita las callbacks para una instancia determinada de `AsyncHook` . If no callbacks are provided enabling is a noop.

La instancia de `AsyncHook` está inhabilitada por defecto. Si la instancia de `AsyncHook` debe ser habilitada inmediatamente después de la creación, se puede utilizar el siguiente patrón.

```js
const async_hooks = require('async_hooks');

const hook = async_hooks.createHook(callbacks).enable();
```

#### asyncHook.disable()

* Returns: {AsyncHook} A reference to `asyncHook`.

Disable the callbacks for a given `AsyncHook` instance from the global pool of `AsyncHook` callbacks to be executed. Una vez que un hook haya sido inhabilitado, no volverá a ser llamado hasta que se habilite.

For API consistency `disable()` also returns the `AsyncHook` instance.

#### Hook Callbacks

Key events in the lifetime of asynchronous events have been categorized into four areas: instantiation, before/after the callback is called, and when the instance is destroyed.

##### init(asyncId, type, triggerAsyncId, resource)

* `asyncId` {number} Una identificación única para el recurso asincrónico.
* `type` {string} The type of the async resource.
* `triggerAsyncId` {number} The unique ID of the async resource in whose execution context this async resource was created.
* `resource` {Object} Reference to the resource representing the async operation, needs to be released during *destroy*.

Called when a class is constructed that has the *possibility* to emit an asynchronous event. This *does not* mean the instance must call `before`/`after` before `destroy` is called, only that the possibility exists.

Se puede observar este comportamiento al hacer algo como abrir un recurso, y después cerrarlo antes de que el recurso pueda ser utilizado. The following snippet demonstrates this.

```js
require('net').createServer().listen(function() { this.close(); });
// OR
clearTimeout(setTimeout(() => {}, 10));
```

A cada nuevo recurso se le asigna una identificación que es única dentro del scope del proceso actual.

###### `tipo`

El `type` es una string que identifica el tipo de recurso que causó que `init` fuese llamado. Generalmente, corresponderá al nombre del constructor del recurso.

```text
FSEVENTWRAP, FSREQWRAP, GETADDRINFOREQWRAP, GETNAMEINFOREQWRAP, HTTPPARSER,
JSSTREAM, PIPECONNECTWRAP, PIPEWRAP, PROCESSWRAP, QUERYWRAP, SHUTDOWNWRAP,
SIGNALWRAP, STATWATCHER, TCPCONNECTWRAP, TCPSERVER, TCPWRAP, TIMERWRAP, TTYWRAP,
UDPSENDWRAP, UDPWRAP, WRITEWRAP, ZLIB, SSLCONNECTION, PBKDF2REQUEST,
RANDOMBYTESREQUEST, TLSWRAP, Timeout, Immediate, TickObject
```

There is also the `PROMISE` resource type, which is used to track `Promise` instances and asynchronous work scheduled by them.

Los usuarios son capaces de definir su propio `type` al usar la API pública del embebedor.

Es posible tener colisiones de nombres de tipo. A los embebedores se les anima a utilizar prefijos únicos, tales como el nombre de paquete del npm, para prevenir colisiones al escuchar a los hooks.

###### `triggerAsyncId`

`triggerAsyncId` is the `asyncId` of the resource that caused (or "triggered") the new resource to initialize and that caused `init` to call. This is different from `async_hooks.executionAsyncId()` that only shows *when* a resource was created, while `triggerAsyncId` shows *why* a resource was created.

La siguiente es una demostracion simple de `triggerAsyncId`:

```js
async_hooks.createHook({
  init(asyncId, type, triggerAsyncId) {
    const eid = async_hooks.executionAsyncId();
    fs.writeSync(
      1, `${type}(${asyncId}): trigger: ${triggerAsyncId} execution: ${eid}\n`);
  }
}).enable();

require('net').createServer((conn) => {}).listen(8080);
```

Output when hitting the server with `nc localhost 8080`:

```console
TCPSERVERWRAP(2): trigger: 1 execution: 1
TCPWRAP(4): trigger: 2 execution: 0
```

The `TCPSERVERWRAP` is the server which receives the connections.

El `TCPWRAP` es la nueva conexión desde el cliente. Cuando se crea una nueva conexión, se construye inmediatamente la instancia de `TCPWrap` . This happens outside of any JavaScript stack. (An `executionAsyncId()` of `0` means that it is being executed from C++ with no JavaScript stack above it.) With only that information, it would be impossible to link resources together in terms of what caused them to be created, so `triggerAsyncId` is given the task of propagating what resource is responsible for the new resource's existence.

###### `recurso`

`resource` es un objeto que representa el recurso asincrónico verdadero que ha sido inicializado. Esto puede contener información útil que puede variar con base en el valor de `type`. Por ejemplo, para el tipo de recurso `GETADDRINFOREQWRAP`, `resource` proporciona el nombre de host utilizado cuando se busca la dirección IP para el nombre de host en `net.Server.listen()`. La API para acceder a esta información actualmente no se considera como pública, pero al usar la API del Embebedor, los usuarios pueden proporcionar y documentar sus propios objetos de recurso. For example, such a resource object could contain the SQL query being executed.

En el caso de las Promesas, el objeto de `resource` tendrá la propiedad de `promise` que se refiere a la `Promise` que está siendo inicializada, y una propiedad de `isChainedPromise`, establecida para `true` si la promesa tiene una promesa mayor, y `false` en caso contrario. For example, in the case of `b = a.then(handler)`, `a` is considered a parent `Promise` of `b`. Aquí, `b` es considerado como una promesa encadenada.

En algunos casos, se reutiliza el objeto de recurso por motivos de rendimiento, por lo tanto, no es seguro utilizarlo como una clave en una `WeakMap` o agregarle propiedades.

###### Ejemplo de contexto asincrónico

Lo siguiente es un ejemplo con información adicional sobre las llamadas a `init` entre las llamadas `before` y `after`, específicamente cómo se verá el callback a `listen()` . The output formatting is slightly more elaborate to make calling context easier to see.

```js
let indent = 0;
async_hooks.createHook({
  init(asyncId, type, triggerAsyncId) {
    const eid = async_hooks.executionAsyncId();
    const indentStr = ' '.repeat(indent);
    fs.writeSync(
      1,
      `${indentStr}${type}(${asyncId}):` +
      ` trigger: ${triggerAsyncId} execution: ${eid}\n`);
  },
  before(asyncId) {
    const indentStr = ' '.repeat(indent);
    fs.writeSync(1, `${indentStr}before:  ${asyncId}\n`);
    indent += 2;
  },
  after(asyncId) {
    indent -= 2;
    const indentStr = ' '.repeat(indent);
    fs.writeSync(1, `${indentStr}after:   ${asyncId}\n`);
  },
  destroy(asyncId) {
    const indentStr = ' '.repeat(indent);
    fs.writeSync(1, `${indentStr}destroy: ${asyncId}\n`);
  },
}).enable();

require('net').createServer(() => {}).listen(8080, () => {
  // Let's wait 10ms before logging the server started.
  setTimeout(() => {
    console.log('>>>', async_hooks.executionAsyncId());
  }, 10);
});
```

Output from only starting the server:

```console
TCPSERVERWRAP(2): trigger: 1 execution: 1
TickObject(3): trigger: 2 execution: 1
before:  3
  Timeout(4): trigger: 3 execution: 3
  TIMERWRAP(5): trigger: 3 execution: 3
after:   3
destroy: 3
before:  5
  before:  4
    TTYWRAP(6): trigger: 4 execution: 4
    SIGNALWRAP(7): trigger: 4 execution: 4
    TTYWRAP(8): trigger: 4 execution: 4
>>> 4
    TickObject(9): trigger: 4 execution: 4
  after:   4
after:   5
before:  9
after:   9
destroy: 4
destroy: 9
destroy: 5
```

Como se ilustra en el ejemplo, `executionAsyncId()` y `execution` cada uno especifica el valor del contexto de ejecución actual; el cual está delineado por las llamadas a `before` y `after`.

Only using `execution` to graph resource allocation results in the following:

```console
TTYWRAP(6) -> Timeout(4) -> TIMERWRAP(5) -> TickObject(3) -> root(1)
```

The `TCPSERVERWRAP` is not part of this graph, even though it was the reason for `console.log()` being called. This is because binding to a port without a hostname is a *synchronous* operation, but to maintain a completely asynchronous API the user's callback is placed in a `process.nextTick()`.

The graph only shows *when* a resource was created, not *why*, so to track the *why* use `triggerAsyncId`.

##### before(asyncId)

* `asyncId` {number}

When an asynchronous operation is initiated (such as a TCP server receiving a new connection) or completes (such as writing data to disk) a callback is called to notify the user. El callback de `before` es llamado justo antes de que se ejecute dicho callback. `asyncId` es el único identificador asignado al recurso que está por ejecutar el callback.

El callback `before` será llamado de 0 a N veces. The `before` callback will typically be called 0 times if the asynchronous operation was cancelled or, for example, if no connections are received by a TCP server. Recursos asincrónicos persistentes como un servidor TCP generalmente llamarán al callback de `before` varias veces, mientras que otras operaciones como `fs.open()` llamarán sólo una vez.

##### after(asyncId)

* `asyncId` {number}

Se llama inmediatamente después que el callback especificado en `before` se completa.

If an uncaught exception occurs during execution of the callback, then `after` will run *after* the `'uncaughtException'` event is emitted or a `domain`'s handler runs.

##### destroy(asyncId)

* `asyncId` {number}

Called after the resource corresponding to `asyncId` is destroyed. También es llamado de manera asincrónica desde la API del embebedor `emitDestroy()`.

Some resources depend on garbage collection for cleanup, so if a reference is made to the `resource` object passed to `init` it is possible that `destroy` will never be called, causing a memory leak in the application. Si el recurso no depende de la recolección de basura, entonces esto no será un problema.

##### promiseResolve(asyncId)

* `asyncId` {number}

Called when the `resolve` function passed to the `Promise` constructor is invoked (either directly or through other means of resolving a promise).

Note that `resolve()` does not do any observable synchronous work.

The `Promise` is not necessarily fulfilled or rejected at this point if the `Promise` was resolved by assuming the state of another `Promise`.

```js
new Promise((resolve) => resolve(true)).then((a) => {});
```

llama a los siguientes callbacks:

```text
init for PROMISE with id 5, trigger id: 1
  promise resolve 5      # corresponds to resolve(true)
init for PROMISE with id 6, trigger id: 5  # the Promise returned by then()
  before 6               # the then() callback is entered
  promise resolve 6      # the then() callback resolves the promise by returning
  after 6
```

#### async_hooks.executionAsyncId()

<!-- YAML
added: v8.1.0
changes:

  - version: v8.2.0
    pr-url: https://github.com/nodejs/node/pull/13490
    description: Renamed from `currentId`
-->

* Returns: {number} The `asyncId` of the current execution context. Útil para rastrear cuando algo llama.

```js
const async_hooks = require('async_hooks');

console.log(async_hooks.executionAsyncId());  // 1 - bootstrap
fs.open(path, 'r', (err, fd) => {
  console.log(async_hooks.executionAsyncId());  // 6 - open()
});
```

The ID returned from `executionAsyncId()` is related to execution timing, not causality (which is covered by `triggerAsyncId()`):

```js
const server = net.createServer(function onConnection(conn) {
  // Returns the ID of the server, not of the new connection, because the
  // onConnection callback runs in the execution scope of the server's
  // MakeCallback().
  async_hooks.executionAsyncId();

}).listen(port, function onListening() {
  // Returns the ID of a TickObject (i.e. process.nextTick()) because all
  // callbacks passed to .listen() are wrapped in a nextTick().
  async_hooks.executionAsyncId();
});
```

Tenga en cuenta que los contextos de promesa no podrán recibir `executionAsyncIds` precisos por defecto. Consulte la sección sobre [promise execution tracking](#async_hooks_promise_execution_tracking).

#### async_hooks.triggerAsyncId()

* Returns: {number} The ID of the resource responsible for calling the callback that is currently being executed.

```js
const server = net.createServer((conn) => {
  // The resource that caused (or triggered) this callback to be called
  // was that of the new connection. Thus the return value of triggerAsyncId()
  // is the asyncId of "conn".
  async_hooks.triggerAsyncId();

}).listen(port, () => {
  // Even though all callbacks passed to .listen() are wrapped in a nextTick()
  // the callback itself exists because the call to the server's .listen()
  // was made. So the return value would be the ID of the server.
  async_hooks.triggerAsyncId();
});
```

Tenga en cuenta que los contextos de promesa no podrán recibir `triggerAsyncId`s válidos por defecto. Consulte la sección sobre [promise execution tracking](#async_hooks_promise_execution_tracking).

## Promise execution tracking

Por defecto, a las ejecuciones de promesas no se les asignan `asyncId`s, debido a la relativamente costosa naturaleza de la [promise introspection API](https://docs.google.com/document/d/1rda3yKGHimKIhg5YeoAmCOtyURgsbTH_qaYR79FELlk) proporcionada por V8. Esto significa que los programas que utilizan promesas ó `async`/`await` no obtendrán una ejecución correcta y activarán identificaciones para contextos de callbacks de promesas por defecto.

Aquí tiene un ejemplo:

```js
const ah = require('async_hooks');
Promise.resolve(1729).then(() => {
  console.log(`eid ${ah.executionAsyncId()} tid ${ah.triggerAsyncId()}`);
});
// produces:
// eid 1 tid 0
```

Observe that the `then()` callback claims to have executed in the context of the outer scope even though there was an asynchronous hop involved. Also note that the `triggerAsyncId` value is `0`, which means that we are missing context about the resource that caused (triggered) the `then()` callback to be executed.

Installing async hooks via `async_hooks.createHook` enables promise execution tracking. Ejemplo:

```js
const ah = require('async_hooks');
ah.createHook({ init() {} }).enable(); // forces PromiseHooks to be enabled.
Promise.resolve(1729).then(() => {
  console.log(`eid ${ah.executionAsyncId()} tid ${ah.triggerAsyncId()}`);
});
// produces:
// eid 7 tid 6
```

En este ejemplo, agregar cualquier función real de un hook habilitó el rastreo de las promesas. There are two promises in the example above; the promise created by `Promise.resolve()` and the promise returned by the call to `then()`. In the example above, the first promise got the `asyncId` `6` and the latter got `asyncId` `7`. During the execution of the `then()` callback, we are executing in the context of promise with `asyncId` `7`. Esta promesa fue activada por el recurso asincrónico `6`.

Another subtlety with promises is that `before` and `after` callbacks are run only on chained promises. That means promises not created by `then()`/`catch()` will not have the `before` and `after` callbacks fired on them. For more details see the details of the V8 [PromiseHooks](https://docs.google.com/document/d/1rda3yKGHimKIhg5YeoAmCOtyURgsbTH_qaYR79FELlk) API.

## API del Embebedor de JavaScript

Library developers that handle their own asynchronous resources performing tasks like I/O, connection pooling, or managing callback queues may use the `AsyncWrap` JavaScript API so that all the appropriate callbacks are called.

### Clase: AsyncResource

La clase `AsyncResource` está diseñada para extenderse por los registros asincrónicos del embebedor. Al usar esto, los usuarios podrán fácilmente activar los eventos del tiempo de vida de sus propios recursos.

El hook de `init` se activará cuando un `AsyncResource` sea instanciado.

A continuación, se muestra un resumen de la API `AsyncResource` .

```js
const { AsyncResource, executionAsyncId } = require('async_hooks');

// AsyncResource() is meant to be extended. Instantiating a
// new AsyncResource() also triggers init. If triggerAsyncId is omitted then
// async_hook.executionAsyncId() is used.
const asyncResource = new AsyncResource(
  type, { triggerAsyncId: executionAsyncId(), requireManualDestroy: false }
);

// Run a function in the execution context of the resource. This will
// * establish the context of the resource
// * trigger the AsyncHooks before callbacks
// * call the provided function `fn` with the supplied arguments
// * trigger the AsyncHooks after callbacks
// * restore the original execution context
asyncResource.runInAsyncScope(fn, thisArg, ...args);

// Call AsyncHooks destroy callbacks.
asyncResource.emitDestroy();

// Return the unique ID assigned to the AsyncResource instance.
asyncResource.asyncId();

// Return the trigger ID for the AsyncResource instance.
asyncResource.triggerAsyncId();

// Call AsyncHooks before callbacks.
// Deprecated: Use asyncResource.runInAsyncScope instead.
asyncResource.emitBefore();

// Call AsyncHooks after callbacks.
// Deprecated: Use asyncResource.runInAsyncScope instead.
asyncResource.emitAfter();
```

#### new AsyncResource(type[, options])

* `type` {string} El tipo de evento asincrónico.
* `opciones` {Object} 
  * `triggerAsyncId` {number} The ID of the execution context that created this async event. **Default:** `executionAsyncId()`.
  * `requireManualDestroy` {boolean} Disables automatic `emitDestroy` when the object is garbage collected. This usually does not need to be set (even if `emitDestroy` is called manually), unless the resource's `asyncId` is retrieved and the sensitive API's `emitDestroy` is called with it. **Default:** `false`.

Ejemplo de uso:

```js
class DBQuery extends AsyncResource {
  constructor(db) {
    super('DBQuery');
    this.db = db;
  }

  getInfo(query, callback) {
    this.db.get(query, (err, data) => {
      this.runInAsyncScope(callback, null, err, data);
    });
  }

  close() {
    this.db = null;
    this.emitDestroy();
  }
}
```

#### asyncResource.runInAsyncScope(fn[, thisArg, ...args])

<!-- YAML
added: v9.6.0
-->

* `fn` {Function} The function to call in the execution context of this async resource.
* `thisArg` {any} The receiver to be used for the function call.
* `...args` {any} Optional arguments to pass to the function.

Llama a la función proporcionada con los argumentos proporcionados en el contexto de ejecución del recurso asincrónico. Esto establecerá el contexto, activará el AsyncHooks antes de los callbacks, llamará la función, activará el AsyncHooks después de los callbacks, y después restaurará el contexto de ejecución original.

#### asyncResource.emitBefore()

<!-- YAML
deprecated: v9.6.0
-->

> Stability: 0 - Deprecated: Use [`asyncResource.runInAsyncScope()`][] instead.

Call all `before` callbacks to notify that a new asynchronous execution context is being entered. If nested calls to `emitBefore()` are made, the stack of `asyncId`s will be tracked and properly unwound.

`before` and `after` calls must be unwound in the same order that they are called. De lo contrario, ocurrirá una excepción irrecuperable y se anulará el proceso. For this reason, the `emitBefore` and `emitAfter` APIs are considered deprecated. Por favor utilice `runInAsyncScope`, ya que ofrece una alternativa mucha más segura.

#### asyncResource.emitAfter()

<!-- YAML
deprecated: v9.6.0
-->

> Stability: 0 - Deprecated: Use [`asyncResource.runInAsyncScope()`][] instead.

Llama a todos los callbacks `after` . If nested calls to `emitBefore()` were made, then make sure the stack is unwound properly. De otra manera ocurrirá un error.

If the user's callback throws an exception, `emitAfter()` will automatically be called for all `asyncId`s on the stack if the error is handled by a domain or `'uncaughtException'` handler.

`before` and `after` calls must be unwound in the same order that they are called. De lo contrario, ocurrirá una excepción irrecuperable y se anulará el proceso. Por este motivo, las APIs de `emitBefore` y `emitAfter` se consideran obsoletas. Por favor, utilice `runInAsyncScope`, ya que ofrece una alternativa mucho más segura.

#### asyncResource.emitDestroy()

Llama a todos los hooks `destroy`. Esto sólo se debe llamar una vez. Ocurrirá un error si se llama más de una vez. Esto **must** se debe llamar manualmente. Si se deja el recurso para que sea recolectado por el GC, entonces los hooks `destroy` nunca serán llamados.

#### asyncResource.asyncId()

* Returns: {number} The unique `asyncId` assigned to the resource.

#### asyncResource.triggerAsyncId()

* Returns: {number} The same `triggerAsyncId` that is passed to the `AsyncResource` constructor.