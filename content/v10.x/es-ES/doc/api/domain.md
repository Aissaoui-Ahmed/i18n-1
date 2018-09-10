# Dominio

<!-- YAML
changes:

  - version: v8.8.0
    description: Any `Promise`s created in VM contexts no longer have a
                 `.domain` property. Their handlers are still executed in the
                 proper domain, however, and `Promise`s created in the main
                 context still possess a `.domain` property.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12489
    description: Handlers for `Promise`s are now invoked in the domain in which
                 the first promise of a chain was created.
-->

<!--introduced_in=v0.10.0-->

> Estabilidad: 0 - Desactualización

**Este módulo esta por convertirse en obsoleto**. Será completamente inútil una vez que el reemplazo de API haya finalizado. La mayoría de los usuarios finales ** no** tienen porqué usarlo. Lo que si deben saber es la funcionalidad que los dominios ofrecen pueden recaer en ella para el momento de uso, pero deben esperar a tener o migrar a una solución diferente en el futuro.

Los dominios ofrecen una forma de manejar múltiples y diversas operaciones IO como una unidad. Si cualquiera de los eventos emisores o callbacks registrados en un dominio produce un evento de `'error'`, o arroja un error.

## Advertencia: ¡No ignore los errores!

<!-- type=misc -->

Los controladores del dominio de error no son un substituto para el cierre de un proceso cuando se produce un error.

Por la naturaleza misma de cómo [`arroja`] [] funciona en JavaScript, casi nunca hay alguna forma segura de "regresar a donde se quedó", sin perdidas de referencias o crear algún otro tipo de estado frágil e indefinido.

Sin embargo, cerrar el proceso es la forma más segura de responde a un error arrojado. Pueden haber muchas conexiones abiertas en un servidor de web normal y, no es recomendable cerrarlos abruptamente solo porque un error fue provocado pro alguien más.

La mejor solución es enviar una respuesta de error a la solicitud que produjo el error, dejando que las otras terminen a su tiempo habitual y deteniendo las emisiones de nuevas solicitudes en ese trabajador.

Así, el uso del `dominio` se hace en conjunto al modulo cluster debido a que el proceso principal puede bifurcar un nuevo trabajador cuando un trabajador encuentra un error. Para los programas de Node.js que escalan en múltiples máquinas, el proxy final o servicio de registro pude registrar la falla y reaccionar de acuerdo a su naturaleza.

Por ejemplo, no es una buena idea:

```js
// XXX ¡ADVERTENCIA! ¡MALA IDEA!

const d = require('domain').create();
d.on('error', (er) => {
  // ¡El error no colisionará el proceso, hará algo peor!
  // Aunque hemos prevenido el proceso de reinicio abrupto, aún estamos filtrando // recursos como locos por si esto llegase a suceder.
  // ¡Esto no es mejor que process.on('uncaughtException')!
  console.log(`error, pero oh bueno ${er.message}`);
});
d.run(() => {
  requiere('http').createServer((req, res) => {
    handleRequest(req, res);
  }).listen(PORT);
});
```

Al usar el contexto de un dominio y la elasticidad al separar nuestros programas en procesos de múltiples de trabajo, podemos reaccionar adecuadamente y manejar los errores con mayor seguridad.

```js
// ¡Mucho mejor!

const cluster = require('cluster');
const PORT = +process.env.PORT || 1337;

Si (cluster.isMaster) {    
// Un escenario más realista tendría más de dos trabajadores y,
// quizás, no colocaría al principal y al trabajador en la misma carpeta.
  //
  // también es posible adornar un poco el registro e 
 // implementar cualquier lógica personalizada necesaria para evitar que DoS
  // ataque y otro mal comportamiento.
  //
 // Ver las opciones en el cluster de documentación.
  //
 // Lo importante es que el proceso principal hace poco, 
 // aumentando nuestra resistencia ante errores inesperados.

  cluster.fork();
  cluster.fork();

  cluster.on('disconnect', (worker) => {
    console.error('disconnect!');
    cluster.fork();
  });

} otro {
  // el trabajador
  //
  // ¡Aquí es donde se colocan nuestros errores!

  const domain = require('domain');

 // Ver el cluster de documentación para más detalles sobre el uso de
// procesos de trabajo para atender solicitudes. Cómo funciona, advertencias, entre otras.

  const server = require('http').createServer((req, res) => {
    const d = domain.create();
    d.on('error', (er) => {
      console.error(`error ${er.stack}`);

      // Nota: ¡ Estamos en un territorio peligroso!
      Por definición, algo inesperado ocurrió,
      / / que probablemente no queríamos.
      // ¡Cualquier cosa puede suceder ahora! ¡Ten mucho cuidado!

      intenta {
        // asegurarte de cerrar en 30 segundos
        const killtimer = setTimeout(() => {
          process.exit(1);
        }, 30000);
        // ¡Pero, no mantengas el proceso abierto solo por eso!
        killtimer.unref();

        // no tomes nuevas solicitudes.
        server.close();

        // Deja que el proceso principal sepa que estamos muertos. Esto desencadenará un 
        // 'desconectar' en el cluster principal y, luego, se bifurcará
        // un nuevo trabajador.
        cluster.worker.disconnect();

        // intenta enviar un error a la solicitud que arrojó el problema
        res.statusCode = 500;
        res.setHeader('content-type', 'text/plain');
        res.end('¡Ups, hubo un problema!\n');
      } catch (er2) {
        // Bueno, no se puede hacer mucho en este punto.
        console.error(`Error sending 500! ${er2.stack}`);
      }
    });

    // Porque req y res  fueron creados antes de que existieran los dominios y
    //  necesitamos agregarlos explicitamente.
    // Mira más abajo la explicación de la vinculación implícita y explicita.
    d.add(req);
    d.add(res);

    // Now run the handler function in the domain.
    d.run(() => {
      handleRequest(req, res);
    });
  });
  server.listen(PORT);
}

// This part is not important. Just an example routing thing.
// Put fancy application logic here.
function handleRequest(req, res) {
  switch (req.url) {
    case '/error':
      // We do some async stuff, and then...
      setTimeout(() => {
        // Whoops!
        flerb.bark();
      }, timeout);
      break;
    default:
      res.end('ok');
  }
}
```

## Additions to Error objects

<!-- type=misc -->

Any time an `Error` object is routed through a domain, a few extra fields are added to it.

* `error.domain` The domain that first handled the error.
* `error.domainEmitter` The event emitter that emitted an `'error'` event with the error object.
* `error.domainBound` The callback function which was bound to the domain, and passed an error as its first argument.
* `error.domainThrown` A boolean indicating whether the error was thrown, emitted, or passed to a bound callback function.

## Implicit Binding

<!--type=misc-->

If domains are in use, then all **new** `EventEmitter` objects (including Stream objects, requests, responses, etc.) will be implicitly bound to the active domain at the time of their creation.

Additionally, callbacks passed to lowlevel event loop requests (such as to `fs.open()`, or other callback-taking methods) will automatically be bound to the active domain. If they throw, then the domain will catch the error.

In order to prevent excessive memory usage, `Domain` objects themselves are not implicitly added as children of the active domain. If they were, then it would be too easy to prevent request and response objects from being properly garbage collected.

To nest `Domain` objects as children of a parent `Domain` they must be explicitly added.

Implicit binding routes thrown errors and `'error'` events to the `Domain`'s `'error'` event, but does not register the `EventEmitter` on the `Domain`. Implicit binding only takes care of thrown errors and `'error'` events.

## Explicit Binding

<!--type=misc-->

Sometimes, the domain in use is not the one that ought to be used for a specific event emitter. Or, the event emitter could have been created in the context of one domain, but ought to instead be bound to some other domain.

For example, there could be one domain in use for an HTTP server, but perhaps we would like to have a separate domain to use for each request.

That is possible via explicit binding.

```js
// create a top-level domain for the server
const domain = require('domain');
const http = require('http');
const serverDomain = domain.create();

serverDomain.run(() => {
  // server is created in the scope of serverDomain
  http.createServer((req, res) => {
    // req and res are also created in the scope of serverDomain
    // however, we'd prefer to have a separate domain for each request.
    // create it first thing, and add req and res to it.
    const reqd = domain.create();
    reqd.add(req);
    reqd.add(res);
    reqd.on('error', (er) => {
      console.error('Error', er, req.url);
      try {
        res.writeHead(500);
        res.end('Error occurred, sorry.');
      } catch (er2) {
        console.error('Error sending 500', er2, req.url);
      }
    });
  }).listen(1337);
});
```

## domain.create()

* Returns: {Domain}

## Class: Domain

The `Domain` class encapsulates the functionality of routing errors and uncaught exceptions to the active `Domain` object.

`Domain` is a child class of [`EventEmitter`][]. To handle the errors that it catches, listen to its `'error'` event.

### domain.members

* {Array}

An array of timers and event emitters that have been explicitly added to the domain.

### domain.add(emitter)

* `emitter` {EventEmitter|Timer} emitter or timer to be added to the domain

Explicitly adds an emitter to the domain. If any event handlers called by the emitter throw an error, or if the emitter emits an `'error'` event, it will be routed to the domain's `'error'` event, just like with implicit binding.

This also works with timers that are returned from [`setInterval()`][] and [`setTimeout()`][]. If their callback function throws, it will be caught by the domain `'error'` handler.

If the Timer or `EventEmitter` was already bound to a domain, it is removed from that one, and bound to this one instead.

### domain.bind(callback)

* `callback` {Function} The callback function
* Returns: {Function} The bound function

The returned function will be a wrapper around the supplied callback function. When the returned function is called, any errors that are thrown will be routed to the domain's `'error'` event.

#### Example

```js
const d = domain.create();

function readSomeFile(filename, cb) {
  fs.readFile(filename, 'utf8', d.bind((er, data) => {
    // if this throws, it will also be passed to the domain
    return cb(er, data ? JSON.parse(data) : null);
  }));
}

d.on('error', (er) => {
  // an error occurred somewhere.
  // if we throw it now, it will crash the program
  // with the normal line number and stack message.
});
```

### domain.enter()

The `enter()` method is plumbing used by the `run()`, `bind()`, and `intercept()` methods to set the active domain. It sets `domain.active` and `process.domain` to the domain, and implicitly pushes the domain onto the domain stack managed by the domain module (see [`domain.exit()`][] for details on the domain stack). The call to `enter()` delimits the beginning of a chain of asynchronous calls and I/O operations bound to a domain.

Calling `enter()` changes only the active domain, and does not alter the domain itself. `enter()` and `exit()` can be called an arbitrary number of times on a single domain.

### domain.exit()

The `exit()` method exits the current domain, popping it off the domain stack. Any time execution is going to switch to the context of a different chain of asynchronous calls, it's important to ensure that the current domain is exited. The call to `exit()` delimits either the end of or an interruption to the chain of asynchronous calls and I/O operations bound to a domain.

If there are multiple, nested domains bound to the current execution context, `exit()` will exit any domains nested within this domain.

Calling `exit()` changes only the active domain, and does not alter the domain itself. `enter()` and `exit()` can be called an arbitrary number of times on a single domain.

### domain.intercept(callback)

* `callback` {Function} The callback function
* Returns: {Function} The intercepted function

This method is almost identical to [`domain.bind(callback)`][]. However, in addition to catching thrown errors, it will also intercept [`Error`][] objects sent as the first argument to the function.

In this way, the common `if (err) return callback(err);` pattern can be replaced with a single error handler in a single place.

#### Example

```js
const d = domain.create();

function readSomeFile(filename, cb) {
  fs.readFile(filename, 'utf8', d.intercept((data) => {
    // note, the first argument is never passed to the
    // callback since it is assumed to be the 'Error' argument
    // and thus intercepted by the domain.

    // if this throws, it will also be passed to the domain
    // so the error-handling logic can be moved to the 'error'
    // event on the domain instead of being repeated throughout
    // the program.
    return cb(null, JSON.parse(data));
  }));
}

d.on('error', (er) => {
  // an error occurred somewhere.
  // if we throw it now, it will crash the program
  // with the normal line number and stack message.
});
```

### domain.remove(emitter)

* `emitter` {EventEmitter|Timer} emitter or timer to be removed from the domain

The opposite of [`domain.add(emitter)`][]. Removes domain handling from the specified emitter.

### domain.run(fn[, ...args])

* `fn` {Function}
* `...args` {any}

Run the supplied function in the context of the domain, implicitly binding all event emitters, timers, and lowlevel requests that are created in that context. Optionally, arguments can be passed to the function.

This is the most basic way to use a domain.

Example:

```js
const domain = require('domain');
const fs = require('fs');
const d = domain.create();
d.on('error', (er) => {
  console.error('Caught error!', er);
});
d.run(() => {
  process.nextTick(() => {
    setTimeout(() => { // simulating some various async stuff
      fs.open('non-existent file', 'r', (er, fd) => {
        if (er) throw er;
        // proceed...
      });
    }, 100);
  });
});
```

In this example, the `d.on('error')` handler will be triggered, rather than crashing the program.

## Domains and Promises

As of Node.js 8.0.0, the handlers of Promises are run inside the domain in which the call to `.then()` or `.catch()` itself was made:

```js
const d1 = domain.create();
const d2 = domain.create();

let p;
d1.run(() => {
  p = Promise.resolve(42);
});

d2.run(() => {
  p.then((v) => {
    // running in d2
  });
});
```

A callback may be bound to a specific domain using [`domain.bind(callback)`][]:

```js
const d1 = domain.create();
const d2 = domain.create();

let p;
d1.run(() => {
  p = Promise.resolve(42);
});

d2.run(() => {
  p.then(p.domain.bind((v) => {
    // running in d1
  }));
});
```

Note that domains will not interfere with the error handling mechanisms for Promises, i.e. no `'error'` event will be emitted for unhandled `Promise` rejections.