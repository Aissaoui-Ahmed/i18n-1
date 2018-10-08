# Seguimiento de eventos

<!--introduced_in=v7.7.0-->

> Estabilidad: 1 - Experimental

El Seguimiento de Eventos provee un mecanismo para centralizar el seguimiento de la información generado por la versión V8 del núcleo Node.js, y código de espacio de usuario.

El seguimiento puede ser habilitado con la bandera de línea de comandos `--trace-event-categories` o al usar el módulo `trace_events`. La bandera de `--trace-event-categories` acepta una lista de nombres de categorías separados por comas.

Las categorías disponibles son:

* `node` - Un marcador de posición vacío.
* `node.async_hooks` - Permite la captura de rastros de datos detallados [`async_hooks`]. Los eventos [`async_hooks`] tienen un único `asyncId`y una propiedad `triggerAsyncId` `triggerId` especial.
* `node.bootstrap` - Permite la captura de los hitos bootstrap de Node.js.
* `node.perf` Permite la captura de las mediciones de [Desempeño del API](perf_hooks.html). 
  * `node.perf.usertiming` - Permite la captura de solamente mediciones y marcas del rendimiento del API cronometrado por el usuario.
  * `node.perf.timerify` - Permite la captura de solamente mediciones API de la función timerify.
* `node.fs.sync` - Permite la captura de datos de seguimientos para métodos de sincronización de archivos del sistema.
* `v8` - The [V8](v8.html) son eventos recolectores de basura, compilan, y están relacionados con la ejecución.

Por defecto están activas las categorías `node`, `node.async_hooks`, y `v8`.

```txt
node --trace-event-categories v8,node,node.async_hooks server.js
```

Versiones anteriores de Node.js requerían del uso de la bandera `--trace-events-enabled` para permitir eventos de seguimiento. Este requisito ha sido removido. Sin embargo `--trace-events-enabled` flag *may* son aún usados y permiten las categorías`node`, `node.async_hooks`, y `v8` de forma predeterminada.

```txt
node --trace-events-enabled

// is equivalent to

node --trace-event-categories v8,node,node.async_hooks
```

Alternamente, los eventos de seguimiento pueden ser permitidos utilizando el módulo `trace_events`:

```js
const trace_events = require('trace_events');
const tracing = trace_events.createTracing({ categories: ['node.perf'] });
tracing.enable();  // Enable trace event capture for the 'node.perf' category

// do work

tracing.disable();  // Disable trace event capture for the 'node.perf' category
```

Ejecutar Node.js con seguimiento habilitado producirá archivos de registro que pueden ser abiertos en la ventana de Chrome [`chrome://tracing`](https://www.chromium.org/developers/how-tos/trace-event-profiling-tool).

El archivo de registro es llamado por defecto `node_trace.${rotation}.log`, donde `${rotation}` es una id ascendiente de la rotación de registros. El patrón de ruta de archivos puede ser específicado con `--trace-event-file-pattern` que acepte una plantilla de string que soporte `${rotation}` y `${pid}`. Por ejemplo:

```txt
node --trace-event-categories v8 --trace-event-file-pattern '${pid}-${rotation}.log' server.js
```

Empezando con Node.Js 10.0.0, el sistema de seguimiento utiliza las mismas fuentes de tiempo como las usadas por `process.hrtime()` sin embargo las marcas de tiempo son expresadas en microsegundos, a diferencia de `process.hrtime()` que devuelve nanosegundos.

## El módulo `trace_events`

<!-- YAML
added: v10.0.0
-->

### `Tracing` object

<!-- YAML
added: v10.0.0
-->

The `Tracing` object is used to enable or disable tracing for sets of categories. Instances are created using the `trace_events.createTracing()` method.

When created, the `Tracing` object is disabled. Calling the `tracing.enable()` method adds the categories to the set of enabled trace event categories. Calling `tracing.disable()` will remove the categories from the set of enabled trace event categories.

#### `tracing.categories`

<!-- YAML
added: v10.0.0
-->

* {string}

A comma-separated list of the trace event categories covered by this `Tracing` object.

#### `tracing.disable()`

<!-- YAML
added: v10.0.0
-->

Disables this `Tracing` object.

Only trace event categories *not* covered by other enabled `Tracing` objects and *not* specified by the `--trace-event-categories` flag will be disabled.

```js
const trace_events = require('trace_events');
const t1 = trace_events.createTracing({ categories: ['node', 'v8'] });
const t2 = trace_events.createTracing({ categories: ['node.perf', 'node'] });
t1.enable();
t2.enable();

// Prints 'node,node.perf,v8'
console.log(trace_events.getEnabledCategories());

t2.disable(); // will only disable emission of the 'node.perf' category

// Prints 'node,v8'
console.log(trace_events.getEnabledCategories());
```

#### `tracing.enable()`

<!-- YAML
added: v10.0.0
-->

Enables this `Tracing` object for the set of categories covered by the `Tracing` object.

#### `tracing.enabled`

<!-- YAML
added: v10.0.0
-->

* {boolean} `true` only if the `Tracing` object has been enabled.

### `trace_events.createTracing(options)`

<!-- YAML
added: v10.0.0
-->

* `options` {Object} 
  * `categories` {string[]} An array of trace category names. Values included in the array are coerced to a string when possible. An error will be thrown if the value cannot be coerced.
* Returns: {Tracing}.

Creates and returns a `Tracing` object for the given set of `categories`.

```js
const trace_events = require('trace_events');
const categories = ['node.perf', 'node.async_hooks'];
const tracing = trace_events.createTracing({ categories });
tracing.enable();
// do stuff
tracing.disable();
```

### `trace_events.getEnabledCategories()`

<!-- YAML
added: v10.0.0
-->

* Returns: {string}

Returns a comma-separated list of all currently-enabled trace event categories. The current set of enabled trace event categories is determined by the *union* of all currently-enabled `Tracing` objects and any categories enabled using the `--trace-event-categories` flag.

Given the file `test.js` below, the command `node --trace-event-categories node.perf test.js` will print `'node.async_hooks,node.perf'` to the console.

```js
const trace_events = require('trace_events');
const t1 = trace_events.createTracing({ categories: ['node.async_hooks'] });
const t2 = trace_events.createTracing({ categories: ['node.perf'] });
const t3 = trace_events.createTracing({ categories: ['v8'] });

t1.enable();
t2.enable();

console.log(trace_events.getEnabledCategories());
```