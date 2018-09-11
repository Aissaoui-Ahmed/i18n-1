# Path

<!--introduced_in=v0.10.0-->

> Estabilidad: 2 - Estable

El módulo `path` provee utilidades para trabajar con rutas de archivos y de directorios. Puede ser accedido usando:

```js
const path = require('path');
```

## Windows vs. POSIX

La operación predeterminada del módulo `path` varía según el sistema operativo en el que una aplicación de Node.js está ejecutándose. Específicamente, cuando se ejecuta en el sistema operativo Windows, el módulo `path` va a asumir que los paths estilo Windows están siendo usados.

Por ejemplo, usar la función `path.basename()` con la ruta de archivo Windows `C:\temp\myfile.html`, va a ocasionar diferentes resultados cuando se ejecuta en POSIX que cuando se ejecuta en Windows:

En POSIX:

```js
path.basename('C:\\temp\\myfile.html');
// Retorna: 'C:\\temp\\myfile.html'
```

En Windows:

```js
path.basename('C:\\temp\\myfile.html');
// Retorna: 'myfile.html'
```

Para alcanzar resultados consistentes cuando se trabaja con rutas de archivo Windows en cualquier sistema operativo, use [`path.win32`][]:

En POSIX y Windows:

```js
path.win32.basename('C:\\temp\\myfile.html');
// Retorna: 'myfile.html'
```

Para alcanzar resultados consistentes cuando se trabajo con rutas de archivo POSIX en cualquier sistema operativo, use [`path.posix`][]:

En POSIX y Windows:

```js
path.posix.basename('/tmp/myfile.html');
// Retorna: 'myfile.html'
```

*Nota:* En Windows, Node.js sigue el concepto de directorio de trabajo por disco. Este comportamiento puede ser observado cuando se usa una ruta de disco sin un backslash. Por ejemplo, `path.resolve('c:\\')` puede potencialmente ocasionar un resultado diferente a `path.resolve('c:')`. Para más información, vea [esta página MSDN](https://msdn.microsoft.com/en-us/library/windows/desktop/aa365247.aspx#fully_qualified_vs._relative_paths).

## path.basename(path[, ext])

<!-- YAML
added: v0.1.25
changes:

  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5348
    description: Passing a non-string as the `path` argument will throw now.
-->

* `path` {string}
* `ext` {string} Una extensión de archivo opcional
* Retorna: {string}

Los métodos `path.basename()` retornan la última porción de un `path`, similar al comando Unix `basename`. Los separadores de directorios de seguimiento son ignorados, por favor vea [`path.sep`][].

```js
path.basename('/foo/bar/baz/asdf/quux.html');
// Retorna: 'quux.html'

path.basename('/foo/bar/baz/asdf/quux.html', '.html');
// Retorna: 'quux'
```

Se produce un [`TypeError`][] si `path` no es un string o si `ext` es dado y no es un string.

## path.delimiter

<!-- YAML
added: v0.9.3
-->

* {string}

Provee el delimitador de ruta específico de la plataforma:

* `;` para Windows
* `:` para POSIX

Por ejemplo, en POSIX:

```js
console.log(process.env.PATH);
// Prints: '/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin'

process.env.PATH.split(path.delimiter);
// Returns: ['/usr/bin', '/bin', '/usr/sbin', '/sbin', '/usr/local/bin']
```

En Windows:

```js
console.log(process.env.PATH);
// Prints: 'C:\Windows\system32;C:\Windows;C:\Program Files\node\'

process.env.PATH.split(path.delimiter);
// Returns ['C:\\Windows\\system32', 'C:\\Windows', 'C:\\Program Files\\node\\']
```

## path.dirname(path)

<!-- YAML
added: v0.1.16
changes:

  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5348
    description: Passing a non-string as the `path` argument will throw now.
-->

* `path` {string}
* Retorna: {string}

El método `path.dirname()` retorna el nombre del directorio de un `path`, similar al comando `dirname` de Unix. Los separadores de directorios de seguimiento son ignorados, vea [`path.sep`][].

```js
path.dirname('/foo/bar/baz/asdf/quux');
// Retorna: '/foo/bar/baz/asdf'
```

Un [`TypeError`][] es producido si `path` no es un string.

## path.extname(path)

<!-- YAML
added: v0.1.25
changes:

  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5348
    description: Passing a non-string as the `path` argument will throw now.
-->

* `path` {string}
* Returns: {string}

El método `path.extname()` retorna la extensión del `path`, desde la última ocurrencia del carácter `.` (punto) hasta el final del string en la última porción del `path`. Si no hay ningún `.` en la última porción del `path`, o si el primer carácter del nombre base del `path` (vea `path.basename()`) es `.`, entonces un string vacío es retornado.

```js
path.extname('index.html');
// Retorna: '.html'

path.extname('index.coffee.md');
// Retorna: '.md'

path.extname('index.');
// Retorna: '.'

path.extname('index');
// Retorna: ''

path.extname('.index');
// Retorna: ''
```

Un [`TypeError`][] es producido si `path` no es un string.

## path.format(pathObject)

<!-- YAML
added: v0.11.15
-->

* `pathObject` {Object} 
  * `dir` {string}
  * `root` {string}
  * `base` {string}
  * `name` {string}
  * `ext` {string}
* Returns: {string}

El método `path.format()` retorna un string de ruta de un objeto. Este es el opuesto de [`path.parse()`][].

Al proporcionar propiedades al `pathObject`, recuerde que hay combinaciones donde una propiedad tiene prioridad sobre otra:

* `pathObject.root` es ignorado si `pathObject.dir` es provisto
* `pathObject.ext` y `pathObject.name` son ignorados si `pathObject.base` existe

Por ejemplo, en POSIX:

```js
// Si `dir`, `root` and `base` son provistos,
// `${dir}${path.sep}${base}`
// va a ser retornado. `root` es ignorado.
path.format({
  root: '/ignored',
  dir: '/home/user/dir',
  base: 'file.txt'
});
// Retorna: '/home/user/dir/file.txt'

// `root` será usado si `dir` no es especificado.
// Si solo `root` es provista o `dir` es igual a `root` entonces el
// separador de plataforma no va a ser incluido. `ext` va a ser ignorado.
path.format({
  root: '/',
  base: 'file.txt',
  ext: 'ignored'
});
// Retorna: '/file.txt'

// `name` + `ext` van a ser usados si `base` no es especificado.
path.format({
  root: '/',
  name: 'file',
  ext: '.txt'
});
// Retorna: '/file.txt'
```

En Windows:

```js
path.format({
  dir: 'C:\\path\\dir',
  base: 'file.txt'
});
// Retorna: 'C:\\path\\dir\\file.txt'
```

## path.isAbsolute(path)

<!-- YAML
added: v0.11.2
-->

* `path` {string}
* Returns: {boolean}

The `path.isAbsolute()` method determines if `path` is an absolute path.

If the given `path` is a zero-length string, `false` will be returned.

Por ejemplo, en POSIX:

```js
path.isAbsolute('/foo/bar'); // true
path.isAbsolute('/baz/..');  // true
path.isAbsolute('qux/');     // false
path.isAbsolute('.');        // false
```

En Windows:

```js
path.isAbsolute('//server');    // true
path.isAbsolute('\\\\server');  // true
path.isAbsolute('C:/foo/..');   // true
path.isAbsolute('C:\\foo\\..'); // true
path.isAbsolute('bar\\baz');    // false
path.isAbsolute('bar/baz');     // false
path.isAbsolute('.');           // false
```

Un [`TypeError`][] es producido si `path` no es un string.

## path.join([...paths])

<!-- YAML
added: v0.1.16
-->

* `...paths` {string} Una secuencia de segmentos de ruta
* Retorna: {string}

El método `path.join()` junta a todos los segmentos `path` dados usando el separador específico de plataforma como delimitador, luego normaliza la ruta resultante.

Los segmentos `path` sin extensión son ignorados. Si el string de ruta unido es un string sin extensión, entonces `'.'` va a ser retornado, representando al directorio de trabajo actual.

```js
path.join('/foo', 'bar', 'baz/asdf', 'quux', '..');
// Returns: '/foo/bar/baz/asdf'

path.join('foo', {}, 'bar');
// throws 'TypeError: Path must be a string. Received {}'
```

Un [`TypeError`][] va a ser producido si cualquiera de los segmentos de ruta no es un string.

## path.normalize(path)

<!-- YAML
added: v0.1.23
-->

* `path` {string}
* Retorna: {string}

El método `path.normalize()` normaliza el `path` dado, resolviendo los segmentos `'..'` y `'.'`.

When multiple, sequential path segment separation characters are found (e.g. `/` on POSIX and either `` or `/` on Windows), they are replaced by a single instance of the platform specific path segment separator (`/` on POSIX and `` on Windows). Trailing separators are preserved.

If the `path` is a zero-length string, `'.'` is returned, representing the current working directory.

Por ejemplo, en POSIX:

```js
path.normalize('/foo/bar//baz/asdf/quux/..');
// Retorna: '/foo/bar/baz/asdf'
```

En Windows:

```js
path.normalize('C:\\temp\\\\foo\\bar\\..\\');
// Retorna: 'C:\\temp\\foo\\'
```

Since Windows recognizes multiple path separators, both separators will be replaced by instances of the Windows preferred separator (``):

```js
path.win32.normalize('C:////temp\\\\/\\/\\/foo/bar');
// Retorna: 'C:\\temp\\foo\\bar'
```

A [`TypeError`][] is thrown if `path` is not a string.

## path.parse(path)

<!-- YAML
added: v0.11.15
-->

* `path` {string}
* Returns: {Object}

The `path.parse()` method returns an object whose properties represent significant elements of the `path`. Trailing directory separators are ignored, see [`path.sep`][].

El objeto retornado va a tener las siguientes propiedades:

* `dir` {string}
* `root` {string}
* `base` {string}
* `name` {string}
* `ext` {string}

Por ejemplo, en POSIX:

```js
path.parse('/home/user/dir/file.txt');
// Retorna:
// { root: '/',
//   dir: '/home/user/dir',
//   base: 'file.txt',
//   ext: '.txt',
//   name: 'file' }
```

```text
┌─────────────────────┬────────────┐
│          dir        │    base    │
├──────┬              ├──────┬─────┤
│ root │              │ name │ ext │
"  /    home/user/dir / file  .txt "
└──────┴──────────────┴──────┴─────┘
(todos los espacios en la línea "" deberían ser ignorados — ellos están solamente para el formato)
```

En Windows:

```js
path.parse('C:\\path\\dir\\file.txt');
// Retorna:
// { root: 'C:\\',
//   dir: 'C:\\path\\dir',
//   base: 'file.txt',
//   ext: '.txt',
//   name: 'file' }
```

```text
┌─────────────────────┬────────────┐
│          dir        │    base    │
├──────┬              ├──────┬─────┤
│ root │              │ name │ ext │
" C:\      path\dir   \ file  .txt "
└──────┴──────────────┴──────┴─────┘
(todos los espacios en la línea "" deberían ser ignorados — ellos están solamente para el formato)
```

Un [`TypeError`][] es producido si `path` no es un string.

## path.posix

<!-- YAML
added: v0.11.15
-->

* {Object}

La propiedad `path.posix` provee acceso a implementaciones específicas de POSIX de los métodos `path`.

## path.relative(from, to)

<!-- YAML
added: v0.5.0
changes:

  - version: v6.8.0
    pr-url: https://github.com/nodejs/node/pull/8523
    description: On Windows, the leading slashes for UNC paths are now included
                 in the return value.
-->

* `from` {string}
* `to` {string}
* Returns: {string}

The `path.relative()` method returns the relative path from `from` to `to` based on the current working directory. If `from` and `to` each resolve to the same path (after calling `path.resolve()` on each), a zero-length string is returned.

If a zero-length string is passed as `from` or `to`, the current working directory will be used instead of the zero-length strings.

Por ejemplo, en POSIX:

```js
path.relative('/data/orandea/test/aaa', '/data/orandea/impl/bbb');
// Returns: '../../impl/bbb'
```

En Windows:

```js
path.relative('C:\\orandea\\test\\aaa', 'C:\\orandea\\impl\\bbb');
// Returns: '..\\..\\impl\\bbb'
```

A [`TypeError`][] is thrown if either `from` or `to` is not a string.

## path.resolve([...paths])

<!-- YAML
added: v0.3.4
-->

* `...paths` {string} A sequence of paths or path segments
* Returns: {string}

The `path.resolve()` method resolves a sequence of paths or path segments into an absolute path.

The given sequence of paths is processed from right to left, with each subsequent `path` prepended until an absolute path is constructed. For instance, given the sequence of path segments: `/foo`, `/bar`, `baz`, calling `path.resolve('/foo', '/bar', 'baz')` would return `/bar/baz`.

If after processing all given `path` segments an absolute path has not yet been generated, the current working directory is used.

The resulting path is normalized and trailing slashes are removed unless the path is resolved to the root directory.

Zero-length `path` segments are ignored.

If no `path` segments are passed, `path.resolve()` will return the absolute path of the current working directory.

```js
path.resolve('/foo/bar', './baz');
// Retorna: '/foo/bar/baz'

path.resolve('/foo/bar', '/tmp/file/');
// Retorna: '/tmp/file'

path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif');
// si el directorio de trabajo actual es /home/myself/node,
// esto retorna '/home/myself/node/wwwroot/static_files/gif/image.gif'
```

Un [`TypeError`][] es producido si cualquiera de los argumentos no es un string.

## path.sep

<!-- YAML
added: v0.7.9
-->

* {string}

Provides the platform-specific path segment separator:

* `` en Windows
* `/` en POSIX

Por ejemplo, en POSIX:

```js
'foo/bar/baz'.split(path.sep);
// Retorna: ['foo', 'bar', 'baz']
```

En Windows:

```js
'foo\\bar\\baz'.split(path.sep);
// Retorna: ['foo', 'bar', 'baz']
```

En Windows, tanto el slash inclinado hacia adelante (`/`) como el slash inclinado hacia atrás (``) son aceptados como separadores de segmentos de ruta; sin embargo, los métodos `path` solo agregan slashes inclinados hacia atrás (``).

## path.toNamespacedPath(path)

<!-- YAML
added: v9.0.0
-->

* `path` {string}
* Retorna: {string}

Solo en sistemas Windows, retorna un [namespace-prefixed path](https://msdn.microsoft.com/library/windows/desktop/aa365247(v=vs.85).aspx#namespaces) equivalente por el `path` dado. Si `path` no es un string, `path` va a ser retornado sin modificaciones.

Este método es significativo solo en el sistema Windows. En sistemas posix, el método es no-operacional y siempre retorna a `path` sin modificaciones.

## path.win32

<!-- YAML
added: v0.11.15
-->

* {Object}

La propiedad `path.win32` provee acceso a implementaciones específicas para Windows de los métodos `path`.