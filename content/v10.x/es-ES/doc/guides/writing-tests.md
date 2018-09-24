# Cómo escribir una prueba para el proyecto Node.js

## ¿Qué es una prueba?

La mayoría de las pruebas en la base de Node.js son programas de JavaScript que ejercen una funcionalidad proporcionada por Node.js y verifican que se comporte como se espera. Las pruebas deben salir con el código `0` cuando tengan éxito. Una prueba falla si:

- Sale al establecer `process.exitCode` a un número distinto a cero. 
  - Esto es realizado usualmente haciendo que una aserción arroje un Error no capturado.
  - Ocasionalmente, usar `process.exit(code)` puede ser apropiado.
- Nunca sale. En este caso, el corredor de prueba terminará la prueba porque establece un límite de tiempo máximo.

Añadir pruebas cuando:

- Se añada una nueva funcionalidad.
- Se reparen regresiones y errores.
- Se amplíe la cobertura de la prueba.

## Estructura del directorio de la prueba

Vea [descripción general de la estructura del directorio](https://github.com/nodejs/node/blob/master/test/README.md#test-directories) para obtener un resumen de la prueba & ubicaciones. Al decidir si expandir un archivo de prueba existente o crear uno nuevo, considere revisar los archivos relacionados con el subsistema. Por ejemplo, busque los `test-streams` al escribir una prueba para `lib/streams.js`.

## Estructura de la prueba

Analicemos esta prueba básica de la suite de prueba de Node.js:

```javascript
'use strict';                                                          // 1
const common = require('../common');                                   // 2
const fixtures = require('../common/fixtures');                        // 3

// Esta prueba asegura que el http-parser pueda soportar caracteres UTF-8
// 5
// en el encabezado http.                                                 // 6

const assert = require('assert');                                      // 8
const http = require('http');                                          // 9

const server = http.createServer(common.mustCall((req, res) => {       // 11
  res.end('ok');                                                       // 12
}));                                                                   // 13
server.listen(0, () => {                                               // 14
  http.get({                                                           // 15
    port: server.address().port,                                       // 16
    headers: { 'Test': 'Düsseldorf' }                                  // 17
  }, common.mustCall((res) => {                                        // 18
    assert.strictEqual(res.statusCode, 200);                           // 19
    server.close();                                                    // 20
  }));                                                                 // 21
});                                                                    // 22
// ...                                                                 // 23
```

### **Líneas 1-3**

```javascript
'use strict';
const common = require('../common');
const fixtures = require('../common/fixtures');
```

La primera línea habilita el modo estricto. Todas las pruebas deben ser realizadas en modo estricto a menos que la naturaleza de la prueba requiera que la prueba sea ejecutada sin el.

La segunda línea carga el módulo `common`. El [`common` module][] es un módulo ayudante que proporciona herramientas útiles para las pruebas. Algunas funcionalidades comunes han sido extraídas en submódulos, los cuales son requeridos por separado como los accesorios del módulo de aquí.

Incluso si una prueba no usa funciones u otras propiedades exportadas por `common`, la prueba aún así debe incluir el módulo `common` antes que cualquier otro módulo. Esto es debido a que el módulo `common` incluye código que causará que la prueba falle si la misma filtra variables en el espacio global. En situaciones donde una prueba no usa funciones u otras propiedades exportadas por `common`, inclúyalo sin asignarlo a un identificador:

```javascript
require('../common');
```

### **Líneas 5-6**

```javascript
// Esta prueba asegura que el http-parser pueda soportar caracteres UTF-8
// en el encabezado http.
```

Una prueba debe iniciar con un comentario que contenga una breve descripción de lo que está diseñada a probar.

### **Líneas 8-9**

```javascript
const assert = require('assert');
const http = require('http');
```

La prueba comprueba la funcionalidad en el módulo `http`.

La mayoría de las pruebas usan el módulo `assert` para confirmar las expectativas de la prueba.

Las instrucciones require se ordenan en orden [ASCII](http://man7.org/linux/man-pages/man7/ascii.7.html) (dígitos, mayúsculas, `_`, minúsculas).

### **Líneas 11-22**

Este es el cuerpo de la prueba. Esta prueba es simple, solo prueba que un servidor HTTP acepte caracteres `non-ASCII` en las cabeceras de una solicitud entrante. Cosas interesantes que tomar en cuenta:

- Si la prueba no depende de un número de puerto específico, entonces siempre usa 0, en lugar de un valor arbitrario, ya que permite que las pruebas se ejecuten en paralelo de forma segura, ya que el sistema operativo asignará un puerto aleatorio. Si la prueba requiere un puerto específico, por ejemplo si la prueba verifica que asignar un puerto específico funciona como se espera, entonces está bien asignar un número de puerto específico.
- El uso de `common.mustCall` para verificar que algunos callbacks/listeners son llamados.
- El servidor HTTP cierra una vez que se han ejecutado todas las comprobaciones. De esta forma, la prueba puede salir con gracia. Recuerde que para que una prueba tenga éxito, debe salir con una código de estado de 0.

## Recomendaciones generales

### Temporizadores

Evite usar temporizadores a menos que la prueba esté probando específicamente los temporizadores. Existen varias razones para esto. Mayormente, son una fuente de problemas. Para una explicación completa vaya [aquí](https://github.com/nodejs/testing/issues/27).

En el evento la prueba necesita un temporizador, considere usar el método `common.platformTimeout()`. Permite configurar tiempos de espera específicos dependiendo de la plataforma:

```javascript
const timer = setTimeout(fail, common.platformTimeout(4000));
```

creará un tiempo de espera de 4 segundos en la mayoría de las plataformas, pero un tiempo de espera más largo en plataformas más lentas.

### El API *common*

Utilice los helpers del módulo `common` tanto como sea posible. Por favor consulte la [documentación del archivo common](https://github.com/nodejs/node/tree/master/test/common) para los detalles completos acerca de los helpers.

#### common.mustCall

Un caso interesante es `common.mustCall`. El uso de `common.mustCall` puede evitar el uso de variables extras y las aserciones correspondientes. Expliquemos esto con una prueba real de la suite de pruebas.

```javascript
'use strict';
require('../common');
const assert = require('assert');
const http = require('http');

let request = 0;
let response = 0;
process.on('exit', function() {
  assert.equal(request, 1, 'http server "request" callback was not called');
  assert.equal(response, 1, 'http request "response" callback was not called');
});

const server = http.createServer(function(req, res) {
  request++;
  res.end();
}).listen(0, function() {
  const options = {
    agent: null,
    port: this.address().port
  };
  http.get(options, function(res) {
    response++;
    res.resume();
    server.close();
  });
});
```

Esta prueba se puede simplificar enormemente al usar `common.mustCall` de esta forma:

```javascript
'use strict';
const common = require('../common');
const http = require('http');

const server = http.createServer(common.mustCall(function(req, res) {
  res.end();
})).listen(0, function() {
  const options = {
    agent: null,
    port: this.address().port
  };
  http.get(options, common.mustCall(function(res) {
    res.resume();
    server.close();
  }));
});

```

#### Módulo Countdown

El [módulo Countdown](https://github.com/nodejs/node/tree/master/test/common#countdown-module) common proporciona un mecanismo de cuenta regresiva simple para pruebas que requieren que se realice una acción particular después de un número determinado de tareas completadas (por ejemplo, apagar un servidor HTTP después de un número específico de solicitudes).

```javascript
const Countdown = require('../common/countdown');

const countdown = new Countdown(2, function() {
  console.log('.');
});

countdown.dec();
countdown.dec(); // La cuenta regresiva será invocada ahora.
```

### Banderas

Algunas pruebas requerirán ejecutar Node.js con banderas de línea de comando específicas establecidas. Para lograr esto, añada un comentario `// Flags:` en el preámbulo de la prueba, seguido por las banderas. Por ejemplo, para permitir que una prueba requiera algunos de los módulos `internal/*`, añada la bandera `--expose-internals`. Una prueba que requeriría `internal/freelist` podría comenzar de esta forma:

```javascript
'use strict';

// Flags: --expose-internals

require('../common');
const assert = require('assert');
const freelist = require('internal/freelist');
```

### Aserciones

Al escribir aserciones, prefiera las versiones estrictas:

- `assert.strictEqual()` sobre `assert.equal()`
- `assert.deepStrictEqual()` sobre `assert.deepEqual()`

Al usar `assert.throws()`, si es posible, proporcione el mensaje de error completo:

```js
assert.throws(
  () => {
    throw new Error('Wrong value');
  },
  /^Error: Wrong value$/ // En lugar de algo como /Valor incorrecto/
);
```

### Características ES.Next

Por razones de rendimiento, solo usamos un subconjunto seleccionado de características ES.Next en código JavaScript en el directorio `lib`. Sin embargo, al escribir pruebas, para la facilidad del backporting, se recomienda usar aquellas características ES.Next que pueden ser usadas directamente sin una bandera en [todas las ramas mantenidas](https://github.com/nodejs/lts). [node.green](http://node.green/) enumera las características disponibles en cada versión, tales como:

- `let` y `const` sobre `var`
- Template literals over string concatenation
- Arrow functions when appropriate

## Naming Test Files

Test files are named using kebab casing. The first component of the name is `test`. The second is the module or subsystem being tested. The third is usually the method or event name being tested. Subsequent components of the name add more information about what is being tested.

For example, a test for the `beforeExit` event on the `process` object might be named `test-process-before-exit.js`. If the test specifically checked that arrow functions worked correctly with the `beforeExit` event, then it might be named `test-process-before-exit-arrow-functions.js`.

## Imported Tests

### Web Platform Tests

Some of the tests for the WHATWG URL implementation (named `test-whatwg-url-*.js`) are imported from the [Web Platform Tests Project](https://github.com/w3c/web-platform-tests/tree/master/url). These imported tests will be wrapped like this:

```js
/* The following tests are copied from WPT. Modifications to them should be
   upstreamed first. Refs:
   https://github.com/w3c/web-platform-tests/blob/8791bed/url/urlsearchparams-stringifier.html
   License: http://www.w3.org/Consortium/Legal/2008/04-testsuite-copyright.html
*/
/* eslint-disable */

// Test code

/* eslint-enable */
```

To improve tests that have been imported this way, please send a PR to the upstream project first. When the proposed change is merged in the upstream project, send another PR here to update Node.js accordingly. Be sure to update the hash in the URL following `WPT Refs:`.

## C++ Unit test

C++ code can be tested using [Google Test](https://github.com/google/googletest). Most features in Node.js can be tested using the methods described previously in this document. But there are cases where these might not be enough, for example writing code for Node.js that will only be called when Node.js is embedded.

### Adding a new test

The unit test should be placed in `test/cctest` and be named with the prefix `test` followed by the name of unit being tested. For example, the code below would be placed in `test/cctest/test_env.cc`:

```c++
#include "gtest/gtest.h"
#include "node_test_fixture.h"
#include "env.h"
#include "node.h"
#include "v8.h"

static bool called_cb = false;
static void at_exit_callback(void* arg);

class EnvTest : public NodeTestFixture { };

TEST_F(EnvTest, RunAtExit) {
  v8::HandleScope handle_scope(isolate_);
  v8::Local<v8::Context> context = v8::Context::New(isolate_);
  node::IsolateData* isolateData = node::CreateIsolateData(isolate_, uv_default_loop());
  Argv argv{"node", "-e", ";"};
  auto env = node::CreateEnvironment(isolateData, context, 1, *argv, 2, *argv);
  node::AtExit(env, at_exit_callback);
  node::RunAtExit(env);
  EXPECT_TRUE(called_cb);
}

static void at_exit_callback(void* arg) {
  called_cb = true;
}
```

Next add the test to the `sources` in the `cctest` target in node.gyp:

```console
'sources': [
  'test/cctest/test_env.cc',
  ...
],
```

Note that the only sources that should be included in the cctest target are actual test or helper source files. There might be a need to include specific object files that are compiled by the `node` target and this can be done by adding them to the `libraries` section in the cctest target.

The test can be executed by running the `cctest` target:

```console
$ make cctest
```

### Node.js test fixture

There is a [test fixture](https://github.com/google/googletest/blob/master/googletest/docs/Primer.md#test-fixtures-using-the-same-data-configuration-for-multiple-tests) named `node_test_fixture.h` which can be included by unit tests. The fixture takes care of setting up the Node.js environment and tearing it down after the tests have finished.

It also contains a helper to create arguments to be passed into Node.js. It will depend on what is being tested if this is required or not.

### Test Coverage

To generate a test coverage report, see the [Test Coverage section of the Pull Requests guide](https://github.com/nodejs/node/blob/master/doc/guides/contributing/pull-requests.md#test-coverage).