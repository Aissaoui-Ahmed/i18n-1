# Módulos

<!--introduced_in=v0.10.0-->

> Estabilidad: 2 - Estable

<!--name=module-->

En el sistema de módulo Node.js, cada archivo es tratado como un módulo separado. Por ejemplo, considere un archivo llamado `foo.js`:

```js
const circle = require('./circle.js');
console.log(`The area of a circle of radius 4 is ${circle.area(4)}`);
```

En la primera linea, `foo.js` carga el módulo `circle.js` que está en el mismo directorio como `foo.js`.

Aquí están los contenidos de `circle.js`:

```js
const { PI } = Math;

exports.area = (r) => PI * r ** 2;

exports.circumference = (r) => 2 * PI * r;
```

El módulo `circle.js` ha exportado las funciones `area()` y `circumference()`. Las funciones y objetos son añadidos a la raíz de un módulo especificando las propiedades adicionales en el objeto `exports` especial.

Las variables locales para el módulo serán privadas, debido a que el módulo está envuelto en una función por Node.js (vea la [envoltura del módulo](#modules_the_module_wrapper)). En este ejemplo, la variable `PI` es privada para `circle.js`.

Un nuevo valor puede ser asignado a la propiedad `module.exports` (como una función o un objeto).

Abajo, `bar.js` hace uso del módulo `square`, el cual exporta una clase Square:

```js
const Square = require('./square.js');
const mySquare = new Square(2);
console.log(`The area of mySquare is ${mySquare.area()}`);
```

El módulo `square` es definido en `square.js`:

```js
// el asignar a las exportaciones no modificará el módulo, debe usar module.exports
module.exports = class Square {
  constructor(width) {
    this.width = width;
  }

  area() {
    return this.width ** 2;
  }
};
```

El sistema de módulo es implementado en el módulo `require('module')`.

## Accediendo al módulo principal

<!-- type=misc -->

Cuando un archivo corre directamente desde Node.js, `require.main` es establecido a su `module`. Eso significa que es posible determinar si un archivo ha sido ejecutado directamente probando el `require.main === module`.

Para un archivo `foo.js`, esto será `true` si se ejecuta a través de `node foo.js`, pero será `false` si se ejecuta por `require('./foo')`.

Debido a que el `module` proporciona una propiedad `filename` (normalmente equivalente a `__filename`), el punto de entrada de la aplicación actual puede obtenerse comprobando `require.main.filename`.

## Addenda: Package Manager Tips

<!-- type=misc -->

La semántica de la función `require()` de Node.js fue diseñada para ser lo suficientemente general para soportar un número de estructuras de directorios razonable. Package manager programs such as `dpkg`, `rpm`, and `npm` will hopefully find it possible to build native packages from Node.js modules without modification.

A continuación damos una estructura de directorio sugerida que podría funcionar:

Digamos que queremos que la carpeta en `/usr/lib/node/<some-package>/<some-version>` sostenga los contenidos de una versión específica de un paquete.

Los paquetes pueden depender el uno del otro. In order to install package `foo`, it may be necessary to install a specific version of package `bar`. El paquete `bar` puede tener dependencias por sí mismo, y, en algunos casos, estas incluso pueden chocar o formar dependencias cíclicas.

Puesto que Node.js busca el `realpath` de cualquier módulo que carga (es decir, resuelve symlinks), y luego busca a sus dependencias en las carpetas `node_modules`, como se describe [aquí](#modules_loading_from_node_modules_folders), esta situación es muy sencilla de resolver con la siguiente arquitectura:

* `/usr/lib/node/foo/1.2.3/` - Contenidos del paquete `foo`, versión 1.2.3.
* `/usr/lib/node/bar/4.3.2/` - Contenidos del paquete `bar` del cual depende `foo`.
* `/usr/lib/node/foo/1.2.3/node_modules/bar` Enlace simbólico a `/usr/lib/node/bar/4.3.2/`.
* `/usr/lib/node/bar/4.3.2/node_modules/*` - Enlaces simbólicos a los paquetes de los cuales `bar` depende.

Así, incluso si se encuentra un ciclo, o si hay conflictos de dependencia, cada módulo será capaz de obtener una versión de su dependencia que puede utilizar.

When the code in the `foo` package does `require('bar')`, it will get the version that is symlinked into `/usr/lib/node/foo/1.2.3/node_modules/bar`. Luego, cuando el código en el paquete `bar` llama a `require('quux')`, obtendrá la versión que está symlinked en `/usr/lib/node/bar/4.3.2/node_modules/quux`.

Además, para hacer que el módulo busque un proceso inclusive más óptimo, antes que colocar los paquetes directamente en `/usr/lib/node`, podríamos colocarlos en `/usr/lib/node_modules/<name>/<version>`. Entonces Node.js no se preocupará en buscar dependencias faltantes en `/usr/node_modules` o `/node_modules`.

Para hacer que los módulos se encuentren disponibles para el REPL de Node.js, podría ser útil también añadir la carpeta `/usr/lib/node_modules` a la variable de entorno `$NODE_PATH`. Debido a que las búsquedas del módulo que utilizan las carpetas `node_modules` son todas relativas, y se basan en la ruta real de los archivos que hacen las llamadas a `require()`, los mismos paquetes pueden encontrarse en cualquier lugar.

## Todo Junto...

<!-- type=misc -->

Para obtener el nombre de archivo exacto que será cargado cuando se llame a `require()`, utilice la función `require.resolve()`.

Colocando todo lo de arriba junto, aquí está el algoritmo de alto nivel en pseudocódigo de lo que hace `require.resolve()`:

```txt
require(X) from module at path Y
1. If X is a core module,
   a. return the core module
   b. STOP
2. If X begins with '/'
   a. set Y to be the filesystem root
3. If X begins with './' or '/' or '../'
   a. LOAD_AS_FILE(Y + X)
   b. LOAD_AS_DIRECTORY(Y + X)
4. LOAD_NODE_MODULES(X, dirname(Y))
5. THROW "not found"

LOAD_AS_FILE(X)

1. If X is a file, load X as JavaScript text.  STOP
2. If X.js is a file, load X.js as JavaScript text.  STOP
3. If X.json is a file, parse X.json to a JavaScript Object.  STOP
4. If X.node is a file, load X.node as binary addon.  STOP

LOAD_INDEX(X)

1. If X/index.js is a file, load X/index.js as JavaScript text.  STOP
2. If X/index.json is a file, parse X/index.json to a JavaScript object. STOP
3. If X/index.node is a file, load X/index.node as binary addon.  STOP

LOAD_AS_DIRECTORY(X)

1. If X/package.json is a file,
   a. Parse X/package.json, and look for "main" field.
   b. let M = X + (json main field)
   c. LOAD_AS_FILE(M)
   d. LOAD_INDEX(M)
2. LOAD_INDEX(X)

LOAD_NODE_MODULES(X, START)

1. let DIRS=NODE_MODULES_PATHS(START)
2. for each DIR in DIRS:
   a. LOAD_AS_FILE(DIR/X)
   b. LOAD_AS_DIRECTORY(DIR/X)

NODE_MODULES_PATHS(START)

1. let PARTS = path split(START)
2. let I = count of PARTS - 1
3. let DIRS = []
4. while I >= 0,
   a. if PARTS[I] = "node_modules" CONTINUE
   b. DIR = path join(PARTS[0 .. I] + "node_modules")
   c. DIRS = DIRS + DIR
   d. let I = I - 1
5. return DIRS
```

## Almacenamiento en Caché

<!--type=misc-->

Los módulos se almacenan en caché después de que se cargan por primera vez. This means (among other things) that every call to `require('foo')` will get exactly the same object returned, if it would resolve to the same file.

Puede que múltiples llamadas a `require('foo')` no provoquen que el código del módulo se ejecute múltiples veces. Esto es una función importante. Con ella, los objetos "parcialmente hechos" pueden devolverse, permitiendo que las dependencias transitivas se carguen incluso cuando puedan causar ciclos.

Para que un módulo ejecute un código múltiples veces, exporte una función y llame a esa función.

### Module Caching Caveats

<!--type=misc-->

Los módulos se almacenan en caché basados en su nombre de archivo resuelto. Since modules may resolve to a different filename based on the location of the calling module (loading from `node_modules` folders), it is not a *guarantee* that `require('foo')` will always return the exact same object, if it would resolve to different files.

Adicionalmente, en sistemas de archivo sensibles a mayúsculas y minúsculas o sistemas operativos, nombres de archivos resueltos diferentes pueden apuntar al mismo archivo, pero el caché todavía los tratará como diferentes módulos y cargará el archivo múltiples veces. Por ejemplo, `require('./foo')` y `require('./FOO')` devuelven dos objetos diferentes, desconsiderando si `./foo` y `./FOO` son o no el mismo archivo.

## Core Modules

<!--type=misc-->

Node.js tiene varios módulos compilados en el binario. Estos módulos son descritos con mayor detalle en otras partes de esta documentación.

The core modules are defined within Node.js's source and are located in the `lib/` folder.

Core modules are always preferentially loaded if their identifier is passed to `require()`. For instance, `require('http')` will always return the built in HTTP module, even if there is a file by that name.

## Ciclos

<!--type=misc-->

Cuando hay llamadas circulares a `require()`, un módulo podría no haber finalizado la ejecución cuando es devuelto.

Considere esta situación:

`a.js`:

```js
console.log('a starting');
exports.done = false;
const b = require('./b.js');
console.log('in a, b.done = %j', b.done);
exports.done = true;
console.log('a done');
```

`b.js`:

```js
console.log('b starting');
exports.done = false;
const a = require('./a.js');
console.log('in b, a.done = %j', a.done);
exports.done = true;
console.log('b done');
```

`main.js`:

```js
console.log('main starting');
const a = require('./a.js');
const b = require('./b.js');
console.log('in main, a.done = %j, b.done = %j', a.done, b.done);
```

Cuando `main.js` carga a `a.js`, luego el `a.js` en turno carga a `b.js`. En ese punto, `b.js` intenta cargar a `a.js`. Para prevenir un bucle infinito, una **copia sin terminar** del objeto de exportaciones `a.js` es devuelto al módulo `b.js`. `b.js` luego termina de cargar, y su objeto `exports` es proporcionado al módulo `a.js`.

Al momento en el que `main.js` ha cargado ambos módulos, ambos terminaron. La salida de este programa sería:

```txt
$ node main.js
main starting
a starting
b starting
in b, a.done = false
b done
in a, b.done = true
a done
in main, a.done = true, b.done = true
```

Hace falta una planificación cuidadosa para permitir que las dependencias de módulo cíclicas trabajen correctamente dentro de una aplicación.

## Módulos de Archivo

<!--type=misc-->

Si no se encuentra el nombre de archivo exacto, entonces Node.js intentará cargar el nombre de archivo requerido con las extensiones añadidas: `.js`, `.json` y finalmente `.node`.

Los archivos `.js` son interpretados como archivos de texto de JavaScript, y los archivos `.json` son analizados como archivos de texto JSON. Los archivos `.node` son interpretados como módulos addon compilados cargados con `dlopen`.

Un módulo requerido con `'/'` como prefijo es una ruta absoluta al archivo. Por ejemplo, `require('/home/marco/foo.js')` cargará el archivo en `/home/marco/foo.js`.

Un módulo requerido con `'./'` como prefijo es relativo al archivo que llama a `require()`. Es decir, `circle.js` debe estar en el mismo directorio que `foo.js` para que `require('./circle')` lo encuentre.

Without a leading `'/'`, `'./'`, or `'../'` to indicate a file, the module must either be a core module or is loaded from a `node_modules` folder.

Si la ruta dada no existe, `require()` arrojará un [`Error`][] con su propiedad de `code` establecida a `'MODULE_NOT_FOUND'`.

## Carpetas como Módulos

<!--type=misc-->

Es conveniente organizar los programas y librerías en directorios independientes, y luego proporcionar un punto de entrada simple a esa librería. Hay tres maneras en las cuales una carpeta puede ser pasada a `require()` como un argumento.

Lo primero es crear un archivo `package.json` en la raíz de la carpeta, el cual especifica un módulo `main`. Un ejemplo de un archivo `package.json` puede lucir así:

```json
{ "name" : "some-library",
  "main" : "./lib/some-library.js" }
```

Si esto estuviese en una carpeta en `./some-library`, entonces `require('./some-library')` intentará cargar `./some-library/lib/some-library.js`.

Este es el grado de conciencia de Node.js de los archivos `package.json`.

Si el archivo especificado por la entrada `'main'` de `package.json` está perdido y no puede ser resuelto, Node.js reportará el módulo completo como perdido con el error por defecto:

```txt
Error: Cannot find module 'some-library'
```

Si no hay ningún archivo `package.json` presente en el directorio, Node.js intentará cargar un archivo `index.js` o `index.node` de ese directorio. Por ejemplo, si no hay ningún archivo `package.json` en el ejemplo anterior, entonces `require('./some-library')` intentará cargar:

* `./some-library/index.js`
* `./some-library/index.node`

## Carga desde Carpetas `node_modules`

<!--type=misc-->

Si el identificador de módulo pasado a `require()` no es un módulo [core](#modules_core_modules), y no comienza con `'/'`, `'../'` o `'./'`, Node.js empieza en el directorio primario del módulo actual, y añade `/node_modules`, e intenta cargar el módulo desde esa ubicación. Node no anexará `node_modules` a una ruta que ya termina en `node_modules`.

Si no se encuentra aquí, entonces lo mueve al directorio primario, y así, hasta que se alcance la raíz del sistema de archivo.

Por ejemplo, si el archivo en `'/home/ry/projects/foo.js'` llamó a `require('bar.js')`, entonces Node.js vería en las siguientes ubicaciones, en este orden:

* `/home/ry/projects/node_modules/bar.js`
* `/home/ry/node_modules/bar.js`
* `/home/node_modules/bar.js`
* `/node_modules/bar.js`

Esto le permite a los programas localizar sus dependencias, y así no entren en conflicto.

Es posible requerir archivos específicos o sub módulos distribuidos con un módulo incluyendo un sufijo de la ruta después del nombre del módulo. For instance `require('example-module/path/to/file')` would resolve `path/to/file` relative to where `example-module` is located. La ruta con sufijo sigue las mismas semánticas de resolución de módulo.

## Carga desde carpetas globales

<!-- type=misc -->

Si la variable de entorno `NODE_PATH` se establece a una lista delimitada por dos puntos de rutas absolutas, entonces Node.js buscará esas rutas para los módulos si no se encuentran en otros lugares.

En Windows, `NODE_PATH` es delimitada por puntos y comas (`;`) en lugar de comas.

`NODE_PATH` fue originalmente creado para soportar módulos de carga de varias rutas antes de que el algoritmo de [resolución de módulo](#modules_all_together) actual fuese congelado.

`NODE_PATH` todavía es soportado, pero es menos necesario ahora que el ecosistema de Node.js ha establecido una convención para ubicar los módulos dependientes. Algunas veces las implementaciones que dependen de `NODE_PATH` muestra un comportamiento sorprendente cuando las personas no están conscientes que `NODE_PATH` debe establecerse. Sometimes a module's dependencies change, causing a different version (or even a different module) to be loaded as the `NODE_PATH` is searched.

Additionally, Node.js will search in the following locations:

* 1: `$HOME/.node_modules`
* 2: `$HOME/.node_libraries`
* 3: `$PREFIX/lib/node`

Where `$HOME` is the user's home directory, and `$PREFIX` is Node.js's configured `node_prefix`.

These are mostly for historic reasons.

It is strongly encouraged to place dependencies in the local `node_modules` folder. These will be loaded faster, and more reliably.

## The module wrapper

<!-- type=misc -->

Before a module's code is executed, Node.js will wrap it with a function wrapper that looks like the following:

```js
(function(exports, require, module, __filename, __dirname) {
// Module code actually lives in here
});
```

By doing this, Node.js achieves a few things:

* It keeps top-level variables (defined with `var`, `const` or `let`) scoped to the module rather than the global object.
* It helps to provide some global-looking variables that are actually specific to the module, such as: 
  * The `module` and `exports` objects that the implementor can use to export values from the module.
  * The convenience variables `__filename` and `__dirname`, containing the module's absolute filename and directory path.

## The module scope

### \_\_dirname

<!-- YAML
added: v0.1.27
-->

<!-- type=var -->

* {string}

The directory name of the current module. This is the same as the [`path.dirname()`][] of the [`__filename`][].

Example: running `node example.js` from `/Users/mjr`

```js
console.log(__dirname);
// Prints: /Users/mjr
console.log(path.dirname(__filename));
// Prints: /Users/mjr
```

### \_\_filename

<!-- YAML
added: v0.0.1
-->

<!-- type=var -->

* {string}

The file name of the current module. This is the resolved absolute path of the current module file.

For a main program this is not necessarily the same as the file name used in the command line.

See [`__dirname`][] for the directory name of the current module.

Examples:

Running `node example.js` from `/Users/mjr`

```js
console.log(__filename);
// Prints: /Users/mjr/example.js
console.log(__dirname);
// Prints: /Users/mjr
```

Given two modules: `a` and `b`, where `b` is a dependency of `a` and there is a directory structure of:

* `/Users/mjr/app/a.js`
* `/Users/mjr/app/node_modules/b/b.js`

References to `__filename` within `b.js` will return `/Users/mjr/app/node_modules/b/b.js` while references to `__filename` within `a.js` will return `/Users/mjr/app/a.js`.

### exports

<!-- YAML
added: v0.1.12
-->

<!-- type=var -->

A reference to the `module.exports` that is shorter to type. See the section about the [exports shortcut](#modules_exports_shortcut) for details on when to use `exports` and when to use `module.exports`.

### module

<!-- YAML
added: v0.1.16
-->

<!-- type=var -->

* {Object}

A reference to the current module, see the section about the [`module` object][]. In particular, `module.exports` is used for defining what a module exports and makes available through `require()`.

### require()

<!-- YAML
added: v0.1.13
-->

<!-- type=var -->

* {Function}

To require modules.

#### require.cache

<!-- YAML
added: v0.3.0
-->

* {Object}

Modules are cached in this object when they are required. By deleting a key value from this object, the next `require` will reload the module. Note that this does not apply to [native addons](addons.html), for which reloading will result in an error.

#### require.extensions

<!-- YAML
added: v0.3.0
deprecated: v0.10.6
-->

> Stability: 0 - Deprecated

* {Object}

Instruct `require` on how to handle certain file extensions.

Process files with the extension `.sjs` as `.js`:

```js
require.extensions['.sjs'] = require.extensions['.js'];
```

**Deprecated** In the past, this list has been used to load non-JavaScript modules into Node.js by compiling them on-demand. However, in practice, there are much better ways to do this, such as loading modules via some other Node.js program, or compiling them to JavaScript ahead of time.

Since the module system is locked, this feature will probably never go away. However, it may have subtle bugs and complexities that are best left untouched.

Note that the number of file system operations that the module system has to perform in order to resolve a `require(...)` statement to a filename scales linearly with the number of registered extensions.

In other words, adding extensions slows down the module loader and should be discouraged.

#### require.main

<!-- YAML
added: v0.1.17
-->

* {Object}

The `Module` object representing the entry script loaded when the Node.js process launched. See ["Accessing the main module"](#modules_accessing_the_main_module).

In `entry.js` script:

```js
console.log(require.main);
```

```sh
node entry.js
```

<!-- eslint-skip -->

```js
Module {
  id: '.',
  exports: {},
  parent: null,
  filename: '/absolute/path/to/entry.js',
  loaded: false,
  children: [],
  paths:
   [ '/absolute/path/to/node_modules',
     '/absolute/path/node_modules',
     '/absolute/node_modules',
     '/node_modules' ] }
```

#### require.resolve(request[, options])

<!-- YAML
added: v0.3.0
changes:

  - version: v8.9.0
    pr-url: https://github.com/nodejs/node/pull/16397
    description: The `paths` option is now supported.
-->

* `request` {string} The module path to resolve.
* `options` {Object} 
  * `paths` {string[]} Paths to resolve module location from. If present, these paths are used instead of the default resolution paths. Note that each of these paths is used as a starting point for the module resolution algorithm, meaning that the `node_modules` hierarchy is checked from this location.
* Returns: {string}

Use the internal `require()` machinery to look up the location of a module, but rather than loading the module, just return the resolved filename.

#### require.resolve.paths(request)

<!-- YAML
added: v8.9.0
-->

* `request` {string} The module path whose lookup paths are being retrieved.
* Returns: {string[]|null}

Returns an array containing the paths searched during resolution of `request` or `null` if the `request` string references a core module, for example `http` or `fs`.

## The `module` Object

<!-- YAML
added: v0.1.16
-->

<!-- type=var -->

<!-- name=module -->

* {Object}

In each module, the `module` free variable is a reference to the object representing the current module. For convenience, `module.exports` is also accessible via the `exports` module-global. `module` is not actually a global but rather local to each module.

### module.children

<!-- YAML
added: v0.1.16
-->

* {module[]}

The module objects required by this one.

### module.exports

<!-- YAML
added: v0.1.16
-->

* {Object}

The `module.exports` object is created by the `Module` system. Sometimes this is not acceptable; many want their module to be an instance of some class. To do this, assign the desired export object to `module.exports`. Note that assigning the desired object to `exports` will simply rebind the local `exports` variable, which is probably not what is desired.

For example suppose we were making a module called `a.js`:

```js
const EventEmitter = require('events');

module.exports = new EventEmitter();

// Do some work, and after some time emit
// the 'ready' event from the module itself.
setTimeout(() => {
  module.exports.emit('ready');
}, 1000);
```

Then in another file we could do:

```js
const a = require('./a');
a.on('ready', () => {
  console.log('module "a" is ready');
});
```

Note that assignment to `module.exports` must be done immediately. It cannot be done in any callbacks. This does not work:

`x.js`:

```js
setTimeout(() => {
  module.exports = { a: 'hello' };
}, 0);
```

`y.js`:

```js
const x = require('./x');
console.log(x.a);
```

#### exports shortcut

<!-- YAML
added: v0.1.16
-->

The `exports` variable is available within a module's file-level scope, and is assigned the value of `module.exports` before the module is evaluated.

It allows a shortcut, so that `module.exports.f = ...` can be written more succinctly as `exports.f = ...`. However, be aware that like any variable, if a new value is assigned to `exports`, it is no longer bound to `module.exports`:

```js
module.exports.hello = true; // Exported from require of module
exports = { hello: false };  // Not exported, only available in the module
```

When the `module.exports` property is being completely replaced by a new object, it is common to also reassign `exports`:

<!-- eslint-disable func-name-matching -->

```js
module.exports = exports = function Constructor() {
  // ... etc.
};
```

To illustrate the behavior, imagine this hypothetical implementation of `require()`, which is quite similar to what is actually done by `require()`:

```js
function require(/* ... */) {
  const module = { exports: {} };
  ((module, exports) => {
    // Module code here. In this example, define a function.
    function someFunc() {}
    exports = someFunc;
    // At this point, exports is no longer a shortcut to module.exports, and
    // this module will still export an empty default object.
    module.exports = someFunc;
    // At this point, the module will now export someFunc, instead of the
    // default object.
  })(module, module.exports);
  return module.exports;
}
```

### module.filename

<!-- YAML
added: v0.1.16
-->

* {string}

The fully resolved filename to the module.

### module.id

<!-- YAML
added: v0.1.16
-->

* {string}

The identifier for the module. Typically this is the fully resolved filename.

### module.loaded

<!-- YAML
added: v0.1.16
-->

* {boolean}

Whether or not the module is done loading, or is in the process of loading.

### module.parent

<!-- YAML
added: v0.1.16
-->

* {module}

The module that first required this one.

### module.paths

<!-- YAML
added: v0.4.0
-->

* {string[]}

The search paths for the module.

### module.require(id)

<!-- YAML
added: v0.5.1
-->

* `id` {string}
* Returns: {Object} `module.exports` from the resolved module

The `module.require` method provides a way to load a module as if `require()` was called from the original module.

In order to do this, it is necessary to get a reference to the `module` object. Since `require()` returns the `module.exports`, and the `module` is typically *only* available within a specific module's code, it must be explicitly exported in order to be used.

## The `Module` Object

<!-- YAML
added: v0.3.7
-->

* {Object}

Provides general utility methods when interacting with instances of `Module` — the `module` variable often seen in file modules. Accessed via `require('module')`.

### module.builtinModules

<!-- YAML
added: v9.3.0
-->

* {string[]}

A list of the names of all modules provided by Node.js. Can be used to verify if a module is maintained by a third party or not.

Note that `module` in this context isn't the same object that's provided by the [module wrapper](#modules_the_module_wrapper). To access it, require the `Module` module:

```js
const builtin = require('module').builtinModules;
```