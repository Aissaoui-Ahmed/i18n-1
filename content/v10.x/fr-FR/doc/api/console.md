# Console

<!--introduced_in=v0.10.13-->

> Stabilité: 2 - stable

Le module `console` fournit une console de débogage simple, similaire au mécanisme de console JavaScript fourni par les navigateurs web.

Le module exporte deux composants spécifiques :

* Une classe `Console` avec des méthodes telles que `console.log()`, `console.error()` et `console.warn()`, qui peut être utilisée pour écrire dans n’importe quel flux Node.js.
* Une instance globale `console` configurée pour écrire dans [`process.stdout`][] et [`process.stderr`][]. L'instance globale `console` peut être utilisée sans appeler `require('console')`.

***Avertissement*** : les méthodes de l’objet global console ne sont ni systématiquement synchrones comme celles de l'API de navigateur auxquelles elles ressemblent, ni systématiquement asynchrones comme tous les autres flux Node.js. Voir la [note sur les processus I/O](process.html#process_a_note_on_process_i_o) pour plus d’informations.

Exemple d’utilisation de l'instance globale `console` :

```js
console.log('hello world');
// Écrit : hello world dans la sortie standard
console.log('hello %s', 'world');
// Écrit : hello world dans la sortie standard
console.error(new Error('Whoops, something bad happened'));
 // Écrit : [Error: Whoops, something bad happened] dans la sortie standard

const name = 'Will Robinson';
console.warn(`Danger ${name}! Danger!`);
// Écrit : Danger Will Robinson! Danger! dans la sortie standard
```

Exemple d’utilisation de la classe `Console` :

```js
const out = getStreamSomehow();
const err = getStreamSomehow();
const myConsole = new console.Console(out, err);

myConsole.log('hello world');
// Écrit : hello world dans out
myConsole.log('hello %s', 'world');
// Écrit : hello world dans out
myConsole.error(new Error('Whoops, something bad happened'));
// Écrit [Error: Whoops, something bad happened] dans err

const name = 'Will Robinson';
myConsole.warn(`Danger ${name}! Danger!`);
// Écrit : Danger Will Robinson! Danger! dans err
```

## Classe : Console

<!-- YAML
changes:

  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/9744
    description: Errors that occur while writing to the underlying streams
                 will now be ignored by default.
-->

<!--type=class-->

La classe `Console` peut-être utilisée pour créer un simple logger ayant des flux de sortie configurables et est accessible en utilisant soit `require('console').Console`, soit `console.Console` (ou leurs équivalents déstructurés) :

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

* `options` {Object} 
  * `stdout` {stream.Writable}
  * `stderr` {stream.Writable}
  * `ignoreErrors` {boolean} ignore les erreurs lors de l’écriture dans les flux sous-jacents. **Par défaut :** `true`.
  * `colorMode` {boolean|string} Définit la prise en charge des couleurs pour cette instance de `Console`. Définir à `true` active le support des couleurs lors de l'inspection des valeurs, définir à `'auto'` rendra le support des couleurs dépendant de la valeur de la propriété `isTTY` et de la valeur retournée par `getColorDepth()` sur le flux respectif. **Par défaut :** `'auto'`.

Crée une nouvelle `Console` avec une ou deux instances de flux accessibles en écriture. `stdout` est un flux accessible en écriture pour écrire des sorties de log ou d'information. `stderr` est utilisé pour des sorties d'avertissement ou d'erreur. Si `stderr` n’est pas fourni, `stdout` est utilisé comme `stderr`.

```js
const output = fs.createWriteStream('./stdout.log');
const errorOutput = fs.createWriteStream('./stderr.log');
// simple logger personnalisé
const logger = new Console({ stdout: output, stderr: errorOutput });
// à utiliser comme console
const count = 5;
logger.log('count: %d', count);
// dans stdout.log: count 5
```

L'instance globale `console` est une `Console` spéciale dont la sortie est envoyée à [`process.stdout`][] et [`process.stderr`][]. Cela équivaut à l’appel :

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

* `value` {any} La valeur testée devant être vraie.
* `...message` {any} Tous les arguments en plus de `valeur` sont utilisés comme message d’erreur.

Un simple test d'assertion qui vérifie si la `valeur` est effectivement vraie. Si ce n’est pas le cas, `Assertion failed` est loggé. S'il est fourni, le `message` d’erreur est formaté via [`util.format()`][] en lui passant tous les arguments de message. La sortie est utilisée comme le message d’erreur.

```js
console.assert(true, 'does nothing');
// OK
console.assert(false, 'Whoops %s work', 'didn\'t');
// Assertion fausse : Whoops didn't work
```

Appeler `console.assert()` avec une assertion fausse ne provoquera que l'écriture du `message` dans la console, sans interrompre le code qui suit.

### console.clear()

<!-- YAML
added: v8.3.0
-->

Lorsque `stdout` est un terminal, appeler `console.clear()` tentera d’effacer le terminal. Lorsque `stdout` n’est pas un terminal, cette méthode ne fait rien.

L’opération spécifique de `console.clear()` peut varier selon les systèmes d’exploitation et les types de terminaux. Pour la plupart des systèmes d’exploitation Linux, `console.clear()` fonctionne de manière similaire à la commande shell `clear`. Sous Windows, `console.clear()` effacera uniquement la sortie dans la fenêtre courante de terminal pour l'exécutable Node.js.

### console.count([label='default'])

<!-- YAML
added: v8.3.0
-->

* `label` {string} l’étiquette d’affichage pour le compteur. **Default:** `'default'`.

Maintains an internal counter specific to `label` and outputs to `stdout` the number of times `console.count()` has been called with the given `label`.

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

* `label` {string} The display label for the counter. **Default:** `'default'`.

Resets the internal counter specific to `label`.

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

The `console.debug()` function is an alias for [`console.log()`][].

### console.dir(obj[, options])

<!-- YAML
added: v0.1.101
-->

* `obj` {any}
* `options` {Object} 
  * `showHidden` {boolean} If `true` then the object's non-enumerable and symbol properties will be shown too. **Default:** `false`.
  * `depth` {number} Tells [`util.inspect()`][] how many times to recurse while formatting the object. This is useful for inspecting large complicated objects. To make it recurse indefinitely, pass `null`. **Default:** `2`.
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