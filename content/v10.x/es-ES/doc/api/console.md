# Consola

<!--introduced_in=v0.10.13-->

> Estabilidad: 2 - Estable

El módulo `console` proporciona una simple consola de depuración que es similar al mecanismo de consola de JavaScript proporcionado por los navegadores web.

El módulo exporta dos componentes específicos:

* Una clase `Console` con métodos como `console.log()`, `console.error()` y `console.warn()` que pueden utilizarse para escribir en cualquier Node.js stream.
* Una instancia global `console` configurada para escribir en [`process.stdout`][] y [`process.stderr`][]. Puede utilizar la global `console` sin llamar a `require('console')`.

***ADVERTENCIA***: Los métodos del objeto consola global no son siempre sincronizados como las APIs de los navegadores a las que se asemejan, ni son siempre asíncrono como todos los otros streams de Node.js. Vea la [nota del proceso I/O](process.html#process_a_note_on_process_i_o) para obtener más información.

Ejemplo de uso de la global `console`:

```js
console.log('hello world');
// Imprime: hello world, en stdout
console.log('hello %s', 'world');
// Imprime: hello world, en stdout
console.error(new Error('Whoops, something bad happened'));
// Imprime: [Error: Whoops, something bad happened], en stderr

const name = 'Will Robinson';
console.warn(`Danger ${name}! Danger!`);
// Imprime: Danger Will Robinson! Danger!, en stderr
```

Ejemplo utilizando la clase `consola`:

```js
const out = getStreamSomehow();
const err = getStreamSomehow();
const myConsole = new console.Console(out, err);

myConsole.log('hello world');
// Prints: hello world, to out
myConsole.log('hello %s', 'world');
// Prints: hello world, to out
myConsole.error(new Error('Whoops, something bad happened'));
// Prints: [Error: Whoops, something bad happened], to err

const name = 'Will Robinson';
myConsole.warn(`Danger ${name}! Danger!`);
// Imprime: Peligro Will Robinson! Danger!, to err
```

## Clase: Consola

<!-- YAML
changes:

  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/9744
    description: Errors that occur while writing to the underlying streams
                 will now be ignored by default.
-->

<!--type=class-->

La clase `console` puede utilizarse para crear un logger sencillo con flujos de salida configurable y se puede acceder ya sea usando `require('console').Console` ó `console.Console` (o sus contrapartes desestructuradas):

```js
const { Console } = require('console');
```

```js
const { Console } = console;
```

### new Console(stdout\[, stderr\]\[, ignoreErrors\])

### new Console(options)

<!-- YAML
changes:

  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/9744
    description: The `ignoreErrors` option was introduced.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19372
    description: The `Console` constructor now supports an `options` argument,
                 and the `colorMode` option was introduced.
-->

* `opciones` {Object} 
  * `stdout` {stream.Writable}
  * `stderr` {stream.Writable}
  * `ignoreErrors` {boolean} Ignorar errores al escribir en el subyacente                           corrientes. **Predeterminado:** `true`.
  * `colorMode` {boolean|string} Establezca el soporte de color para esta instancia de `Console`. El ajuste a `true` permite colorear mientras se inspeccionan los valores, configurando `'auto'` hará que el soporte de color dependa del valor de la propiedad `isTTY` y el valor devuelto por `getColorDepth()` en la secuencia respectiva. **Predeterminado:** `'auto'`.

Crea una nueva `Consola` con una o dos instancias de flujo modificables. `stdout` es un secuencia de escritura para imprimir salida de registro o información. `stderr` se usa para advertencias o salida de error. Si `stderr` no se proporciona, `stdout ` se usa para `stderr`.

```js
const output = fs.createWriteStream('./stdout.log');
const errorOutput = fs.createWriteStream('./stderr.log');
// registrador simple personalizado
const logger = new Console({ stdout: output, stderr: errorOutput });
// usar como consola
const count = 5;
logger.log('count: %d', count);
// in stdout.log: count 5
```

La `consola global` es una `consola` especial a la que se envía la salida [`process.stdout`] [] y [` process.stderr`] []. Es equivalente a llamar:

```js
new Console({ stdout: process.stdout, stderr: process.stderr });
```

### console.assert(value[, ...message])

<!-- YAML
added: v0.1.101
changes:

  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/17706
    description: The implementation is now spec compliant and does not throw
                 anymore.
-->

* `value` {any} El valor probado para ser verdad.
* `... mensaje` {any} Todos los argumentos además del `valor` se utilizan como mensaje de error.

Una prueba de afirmación simple que verifica si el `value` es verdadero. Si no es, `Assertion failed` el loggueado. Si se proporciona, el error de `message` está formateado usando [`util.format()`][] pasando a lo largo de todos los argumentos del mensaje. El resultado es usado como mensaje de error.

```js
console.assert (true, 'does nothing');
// OKAY
console.assert (false, 'Whoops %s work', 'didn\' t');
// Falló la aserción: Whoops no funcionó
```

Llamar a `console.assert()` con una aserción de falsy solo causará el `mensaje` para imprimir en la consola sin interrumpir la ejecución del código subsiguiente.

### console.clear()

<!-- YAML
added: v8.3.0
-->

Cuando `stdout` es un TTY, llamando a `console.clear()` intentará borrar el TTY. Cuando `stdout` no es un TTY, este método no hace nada.

La operación específica de `console.clear()` puede variar entre los sistemas operativos y tipos de terminales. Para la mayoría de los sistemas operativos Linux, `console.clear()` funciona de manera similar al comando de shell `borrar`. En Windows, `console.clear()` borrará solo la salida en la ventana del terminal actual para Node.js binario.

### console.count([label='default'])

<!-- YAML
added: v8.3.0
-->

* `label` {string} La etiqueta de visualización para el contador. **Default:** `'default'`.

Mantiene un contador interno específico para la etiqueta `label` y las salidas a `stdout` número de veces que se ha llamado a `console.count()` con la etiqueta `label`.

<!-- eslint-skip -->

```js
> console.count()
default: 1
undefined
> console.count('default')
default: 2
undefined
> console.count('abc')
abc: 1
undefined
> console.count('xyz')
xyz: 1
undefined
> console.count('abc')
abc: 2
undefined
> console.count()
default: 3
undefined
>
```

### console.countReset([label='default'])

<!-- YAML
added: v8.3.0
-->

* `label` {string} La etiqueta de visualización para el contador. **Default:** `'default'`.

Restablece el contador interno específico de la `etiqueta`.

<!-- eslint-skip -->

```js
> console.count('abc');
abc: 1
undefined
> console.countReset('abc');
undefined
> console.count('abc');
abc: 1
undefined
>
```

### console.debug(data[, ...args])

<!-- YAML
added: v8.0.0
changes:

  - version: v9.3.0
    pr-url: https://github.com/nodejs/node/pull/17033
    description: "`console.debug` is now an alias for `console.log`."
-->

* `data` {any}
* `...args` {any}

La función `console.debug()` es un alias para [`console.log()`][].

### console.dir(obj[, options])

<!-- YAML
added: v0.1.101
-->

* `obj` {any}
* `options` {Object} 
  * `showHidden` {boolean} If `true` luego el objeto no enumerable y símbolo las propiedades se mostrarán también. **Default:** `false`.
  * `depth` {number} Indica [`util.inspect()`][] cuántas veces se repite mientras formatear el objeto. This is useful for inspecting large complicated objects. To make it recurse indefinitely, pass `null`. **Default:** `2`.
  * `colors` {boolean} If `true`, then the output will be styled with ANSI color codes. Colors are customizable; see [customizing `util.inspect()` colors][]. **Default:** `false`.

Uses [`util.inspect()`][] on `obj` and prints the resulting string to `stdout`. This function bypasses any custom `inspect()` function defined on `obj`.

### console.dirxml(...data)

<!-- YAML
added: v8.0.0
changes:

  - version: v9.3.0
    pr-url: https://github.com/nodejs/node/pull/17152
    description: "`console.dirxml` now calls `console.log` for its arguments."
-->

* `...data` {any}

This method calls `console.log()` passing it the arguments received. Please note that this method does not produce any XML formatting.

### console.error(\[data\]\[, ...args\])

<!-- YAML
added: v0.1.100
-->

* `data` {any}
* `...args` {any}

Prints to `stderr` with newline. Multiple arguments can be passed, with the first used as the primary message and all additional used as substitution values similar to printf(3) (the arguments are all passed to [`util.format()`][]).

```js
const code = 5;
console.error('error #%d', code);
// Prints: error #5, to stderr
console.error('error', code);
// Prints: error 5, to stderr
```

If formatting elements (e.g. `%d`) are not found in the first string then [`util.inspect()`][] is called on each argument and the resulting string values are concatenated. See [`util.format()`][] for more information.

### console.group([...label])

<!-- YAML
added: v8.5.0
-->

* `...label` {any}

Increases indentation of subsequent lines by two spaces.

If one or more `label`s are provided, those are printed first without the additional indentation.

### console.groupCollapsed()

<!-- YAML
  added: v8.5.0
-->

An alias for [`console.group()`][].

### console.groupEnd()

<!-- YAML
added: v8.5.0
-->

Decreases indentation of subsequent lines by two spaces.

### console.info(\[data\]\[, ...args\])

<!-- YAML
added: v0.1.100
-->

* `data` {any}
* `...args` {any}

The `console.info()` function is an alias for [`console.log()`][].

### console.log(\[data\]\[, ...args\])

<!-- YAML
added: v0.1.100
-->

* `data` {any}
* `...args` {any}

Prints to `stdout` with newline. Multiple arguments can be passed, with the first used as the primary message and all additional used as substitution values similar to printf(3) (the arguments are all passed to [`util.format()`][]).

```js
const count = 5;
console.log('count: %d', count);
// Prints: count: 5, to stdout
console.log('count:', count);
// Prints: count: 5, to stdout
```

See [`util.format()`][] for more information.

### console.table(tabularData[, properties])

<!-- YAML
added: v10.0.0
-->

* `tabularData` {any}
* `properties` {string[]} Alternate properties for constructing the table.

Try to construct a table with the columns of the properties of `tabularData` (or use `properties`) and rows of `tabularData` and log it. Falls back to just logging the argument if it can’t be parsed as tabular.

```js
// These can't be parsed as tabular data
console.table(Symbol());
// Symbol()

console.table(undefined);
// undefined

console.table([{ a: 1, b: 'Y' }, { a: 'Z', b: 2 }]);
// ┌─────────┬─────┬─────┐
// │ (index) │  a  │  b  │
// ├─────────┼─────┼─────┤
// │    0    │  1  │ 'Y' │
// │    1    │ 'Z' │  2  │
// └─────────┴─────┴─────┘

console.table([{ a: 1, b: 'Y' }, { a: 'Z', b: 2 }], ['a']);
// ┌─────────┬─────┐
// │ (index) │  a  │
// ├─────────┼─────┤
// │    0    │  1  │
// │    1    │ 'Z' │
// └─────────┴─────┘
```

### console.time(label)

<!-- YAML
added: v0.1.104
-->

* `label` {string} **Default:** `'default'`

Starts a timer that can be used to compute the duration of an operation. Timers are identified by a unique `label`. Use the same `label` when calling [`console.timeEnd()`][] to stop the timer and output the elapsed time in milliseconds to `stdout`. Timer durations are accurate to the sub-millisecond.

### console.timeEnd(label)

<!-- YAML
added: v0.1.104
changes:

  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5901
    description: This method no longer supports multiple calls that don’t map
                 to individual `console.time()` calls; see below for details.
-->

* `label` {string} **Default:** `'default'`

Stops a timer that was previously started by calling [`console.time()`][] and prints the result to `stdout`:

```js
console.time('100-elements');
for (let i = 0; i < 100; i++) {}
console.timeEnd('100-elements');
// prints 100-elements: 225.438ms
```

### console.trace(\[message\]\[, ...args\])

<!-- YAML
added: v0.1.104
-->

* `message` {any}
* `...args` {any}

Prints to `stderr` the string `'Trace: '`, followed by the [`util.format()`][] formatted message and stack trace to the current position in the code.

```js
console.trace('Show me');
// Prints: (stack trace will vary based on where trace is called)
//  Trace: Show me
//    at repl:2:9
//    at REPLServer.defaultEval (repl.js:248:27)
//    at bound (domain.js:287:14)
//    at REPLServer.runBound [as eval] (domain.js:300:12)
//    at REPLServer.<anonymous> (repl.js:412:12)
//    at emitOne (events.js:82:20)
//    at REPLServer.emit (events.js:169:7)
//    at REPLServer.Interface._onLine (readline.js:210:10)
//    at REPLServer.Interface._line (readline.js:549:8)
//    at REPLServer.Interface._ttyWrite (readline.js:826:14)
```

### console.warn(\[data\]\[, ...args\])

<!-- YAML
added: v0.1.100
-->

* `data` {any}
* `...args` {any}

The `console.warn()` function is an alias for [`console.error()`][].

## Inspector only methods

The following methods are exposed by the V8 engine in the general API but do not display anything unless used in conjunction with the [inspector](debugger.html) (`--inspect` flag).

### console.markTimeline(label)

<!-- YAML
added: v8.0.0
-->

* `label` {string} **Default:** `'default'`

This method does not display anything unless used in the inspector. The `console.markTimeline()` method is the deprecated form of [`console.timeStamp()`][].

### console.profile([label])

<!-- YAML
added: v8.0.0
-->

* `label` {string}

This method does not display anything unless used in the inspector. The `console.profile()` method starts a JavaScript CPU profile with an optional label until [`console.profileEnd()`][] is called. The profile is then added to the **Profile** panel of the inspector.

```js
console.profile('MyLabel');
// Some code
console.profileEnd();
// Adds the profile 'MyLabel' to the Profiles panel of the inspector.
```

### console.profileEnd()

<!-- YAML
added: v8.0.0
-->

This method does not display anything unless used in the inspector. Stops the current JavaScript CPU profiling session if one has been started and prints the report to the **Profiles** panel of the inspector. See [`console.profile()`][] for an example.

### console.timeStamp([label])

<!-- YAML
added: v8.0.0
-->

* `label` {string}

This method does not display anything unless used in the inspector. The `console.timeStamp()` method adds an event with the label `'label'` to the **Timeline** panel of the inspector.

### console.timeline([label])

<!-- YAML
added: v8.0.0
-->

* `label` {string} **Default:** `'default'`

This method does not display anything unless used in the inspector. The `console.timeline()` method is the deprecated form of [`console.time()`][].

### console.timelineEnd([label])

<!-- YAML
added: v8.0.0
-->

* `label` {string} **Default:** `'default'`

This method does not display anything unless used in the inspector. The `console.timelineEnd()` method is the deprecated form of [`console.timeEnd()`][].