# Assert

<!--introduced_in=v0.1.21-->

> Stability: 2 - Stable

El módulo `assert` proporciona un conjunto simple de pruebas de afirmación que pueden ser utilizadas para probar invariantes.

Existe un modo `strict` y un modo `legacy`, pero se recomiendo utilizar solo el [`strict mode`][].

Para mayor información acerca de las comparaciones de igualdad utilizadas vea la [MDN's guide on equality comparisons and sameness](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness).

## Clase: assert.AssertionError

Una subclase de `Error` que indica un fallo en una afirmación. Todos los errores arrojados por el módulo `assert` serán instancias de la clase `AssertionError`.

### new assert.AssertionError(opciones)

<!-- YAML
added: v0.1.21
-->

* `opciones` {Object} 
  * `message` {string} si se proporciona, el mensaje de error se establecerá a este valor.
  * `actual` {any} La propiedad `actual` en la instancia de error va a contener este valor. Utilizado internamente para la entrada de error `actual` en el caso p. ej., [`assert.strictEqual()`] es utilizado.
  * `expected` {any} La propiedad `expected` en la instancia de error va a contener este valor. Utilizado internamente para la entrada de error `expected` en el caso p. ej., [`assert.strictEqual()`] es utilizado.
  * `operator` {string} La propiedad `operator` en la instancia de error va a contener este valor. Utilizado internamente para indicar qué operación fue utilizada para la comparación (o que función de afirmación desencadenó el error).
  * `stackStartFn` {Function} If provided, the generated stack trace is going to remove all frames up to the provided function.

Una subclase de `Error` que indica la falla de una afirmación.

Todas las instancias contienen la propiedades integradas de `Error` (`message` y `name`) y:

* `actual` {any} Establecer al valor actual en el caso, p. ej., [`assert.strictEqual()`] es utilizado.
* `expected` {any} Set to the expected value in case e.g., [`assert.strictEqual()`] is used.
* `generatedMessage` {boolean} Indica si el mensaje fue generado automáticamente (`true`) o no.
* `code` {string} Esto siempre está establecido a la string `ERR_ASSERTION` para indicar que el error es en realidad un error de afirmación.
* `operator` {string} Set to the passed in operator value.

```js
const assert = require('assert');

//Generar un AssertionError para comparar el mensaje después:
const { message } = new assert.AssertionError({
  actual: 1,
  expected: 2,
  operator: 'strictEqual'
});

//Verificar la salida del error:
try {
  assert.strictEqual(1, 2);
} catch (err) {
  assert(err instanceof assert.AssertionError);
  assert.strictEqual(err.message, message);
  assert.strictEqual(err.name, 'AssertionError [ERR_ASSERTION]');
  assert.strictEqual(err.actual, 1);
  assert.strictEqual(err.expected, 2);
  assert.strictEqual(err.code, 'ERR_ASSERTION');
  assert.strictEqual(err.operator, 'strictEqual');
  assert.strictEqual(err.generatedMessage, true);
}
```

## Modo estricto

<!-- YAML
added: v9.9.0
changes:

  - version: v9.9.0
    pr-url: https://github.com/nodejs/node/pull/17615
    description: Added error diffs to the strict mode
  - version: v9.9.0
    pr-url: https://github.com/nodejs/node/pull/17002
    description: Added strict mode to the assert module.
-->

Al utilizar el `strict mode`, cualquier función `assert` utilizará la igualdad utilizada en el modo de función estricta. Por lo tanto, [`assert.deepEqual()`][] trabajará, por ejemplo, al igual que [`assert.deepStrictEqual()`][].

Además de eso, los mensajes de error que involucran objetos, producen un error diff en lugar de mostrar ambos objetos. Ese no es el caso para el modo legado.

Puede ser accedido utilizando:

```js
const assert = require('assert').strict;
```

Ejemplo de error diff:

```js
const assert = require('assert').strict;

assert.deepEqual([[[1, 2, 3]], 4, 5], [[[1, 2, '3']], 4, 5]);
// AssertionError: Input A expected to strictly deep-equal input B:
// + expected - actual ... Lineas saltadas
//
//   [
//     [
// ...
//       2,
// -     3
// +     '3'
//     ],
// ...
//     5
//   ]
```

Para desactivar los colores, use la variable de entorno `NODE_DISABLE_COLORS`. Por favor, note que esto también desactivará los colores en el REPL.

## Modo legado

> Estabilidad: 0 - Obsoleto: Utilice el modo estricto en su lugar.

Al acceder a `assert` directamente en lugar de utilizar la propiedad `strict`, la [Abstract Equality Comparison](https://tc39.github.io/ecma262/#sec-abstract-equality-comparison) será utilizada para cualquier función sin el "strict" en su nombre, como [`assert.deepEqual()`][].

Puede ser accedido utilizando:

```js
const assert = require('assert');
```

Se recomienda usar el [`strict mode`][] en su lugar ya que la [Abstract Equality Comparison](https://tc39.github.io/ecma262/#sec-abstract-equality-comparison) a menudo puede obtener resultados sorprendentes. This is especially true for [`assert.deepEqual()`][], where the comparison rules are lax:

```js
// ADVERTENCIA: ¡Esto no arroja un AssertionError!
assert.deepEqual(/a/gi, new Date());
```

## assert(value[, message])

<!-- YAML
added: v0.5.9
-->

* `value` {any}
* `message` {any}

Un alias de [`assert.ok()`][].

## assert.deepEqual(actual, expected[, message])

<!-- YAML
added: v0.1.21
changes:

  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/15001
    description: The `Error` names and messages are now properly compared
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12142
    description: The `Set` and `Map` content is also compared
  - version: v6.4.0, v4.7.1
    pr-url: https://github.com/nodejs/node/pull/8002
    description: Typed array slices are handled correctly now.
  - version: v6.1.0, v4.5.0
    pr-url: https://github.com/nodejs/node/pull/6432
    description: Objects with circular references can be used as inputs now.
  - version: v5.10.1, v4.4.3
    pr-url: https://github.com/nodejs/node/pull/5910
    description: Handle non-`Uint8Array` typed arrays correctly.
-->

* `actual` {any}
* `expected` {any}
* `message` {any}

**Modo estricto**

Un alias de [`assert.deepStrictEqual()`][].

**Modo legado**

> Estabilidad: 0 - Obsoleto: Use [`assert.deepStrictEqual()`][] en su lugar.

Pruebas para igualdad profunda entre los parámetros `actual` y `expected`. Los valores primitivos son comparados con la [Abstract Equality Comparison](https://tc39.github.io/ecma262/#sec-abstract-equality-comparison) ( `==` ).

Only [enumerable "own" properties](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Enumerability_and_ownership_of_properties) are considered. La implementación de [`assert.deepEqual()`][] no realiza una prueba de [`[[Prototype]]`](https://tc39.github.io/ecma262/#sec-ordinary-object-internal-methods-and-internal-slots) de objetos o de propiedades [`Symbol`][] enumerables propias. For such checks, consider using [`assert.deepStrictEqual()`][] instead. [`assert.deepEqual()`][] puede obtener resultados potencialmente sorprendentes. El siguiente ejemplo no arroja un `AssertionError` porque la propiedades en el objeto [`RegExp`][] no son enumerables:

```js
// ADVERTENCIA: ¡Esto no arroja un AssertionError!
assert.deepEqual(/a/gi, new Date());
```

Se hace una excepción para [`Map`][] y para [`Set`][]. `Map`s and `Set`s have their contained items compared too, as expected.

Igualdad "profunda" significa que las propiedades enumerables "propias" de objetos secundarios también son evaluadas:

```js
const assert = require('assert');

const obj1 = {
  a: {
    b: 1
  }
};
const obj2 = {
  a: {
    b: 2
  }
};
const obj3 = {
  a: {
    b: 1
  }
};
const obj4 = Object.create(obj1);

assert.deepEqual(obj1, obj1);
// OK, el objeto es igual a si mismo

assert.deepEqual(obj1, obj2);
// AssertionError: { a: { b: 1 } } deepEqual { a: { b: 2 } }
// los valores de b son diferentes

assert.deepEqual(obj1, obj3);
// OK, los objetos son iguales

assert.deepEqual(obj1, obj4);
// AssertionError: { a: { b: 1 } } deepEqual {}
//Los prototipos son ignorados
```

Si los valores no son iguales, se arroja un `AssertionError` con una propiedad `message` establecida igual al valor del parámetro `message`. Si el parámetro `message` no está definido, un mensaje de error predeterminado es asignado. Si el parámetro `message` es una instancia de un [`Error`][], entonces se arrojará en lugar de `AssertionError`.

## assert.deepStrictEqual(actual, expected[, message])

<!-- YAML
added: v1.2.0
changes:

  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/15169
    description: Enumerable symbol properties are now compared.
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/15036
    description: The `NaN` is now compared using the
              [SameValueZero](https://tc39.github.io/ecma262/#sec-samevaluezero)
              comparison.
  - version: v8.5.0
    pr-url: https://github.com/nodejs/node/pull/15001
    description: The `Error` names and messages are now properly compared
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12142
    description: The `Set` and `Map` content is also compared
  - version: v6.4.0, v4.7.1
    pr-url: https://github.com/nodejs/node/pull/8002
    description: Typed array slices are handled correctly now.
  - version: v6.1.0
    pr-url: https://github.com/nodejs/node/pull/6432
    description: Objects with circular references can be used as inputs now.
  - version: v5.10.1, v4.4.3
    pr-url: https://github.com/nodejs/node/pull/5910
    description: Handle non-`Uint8Array` typed arrays correctly.
-->

* `actual` {any}
* `expected` {any}
* `message` {any}

Pruebas para igualdad profunda entre los parámetros `actual` y `expected`. Igualdad "profunda" significa que las propiedades enumerables "propias" de objetos secundarios son evaluadas recursivamente también por las siguientes reglas.

### Detalles de compración

* Los valor primitivos son comparados utilizando la [SameValue Comparison](https://tc39.github.io/ecma262/#sec-samevalue), usada por [`Object.is()`][].
* Las [Type tags](https://tc39.github.io/ecma262/#sec-object.prototype.tostring) de objetos deben ser las mismas.
* Los [`[[Prototype]]`](https://tc39.github.io/ecma262/#sec-ordinary-object-internal-methods-and-internal-slots) de objetos son comparados utilizando la [Strict Equality Comparison](https://tc39.github.io/ecma262/#sec-strict-equality-comparison).
* Only [enumerable "own" properties](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Enumerability_and_ownership_of_properties) are considered.
* [`Error`][] names and messages are always compared, even if these are not enumerable properties.
* Las propiedades [`Symbol`][] enumerables propias también son comparadas.
* [Object wrappers](https://developer.mozilla.org/en-US/docs/Glossary/Primitive#Primitive_wrapper_objects_in_JavaScript) are compared both as objects and unwrapped values.
* Las propiedades `Object` son comparadas sin orden.
* `Map` keys and `Set` items are compared unordered.
* Recursion stops when both sides differ or both sides encounter a circular reference.
* [`WeakMap`][] and [`WeakSet`][] comparison does not rely on their values. See below for further details.

```js
const assert = require('assert').strict;

// Esto falla porque 1 !== '1'.
assert.deepStrictEqual({ a: 1 }, { a: '1' });
// AssertionError: Input A expected to strictly deep-equal input B:
// + expected - actual
//   {
// -   a: 1
// +   a: '1'
//   }

// The following objects don't have own properties
const date = new Date();
const object = {};
const fakeDate = {};
Object.setPrototypeOf(fakeDate, Date.prototype);

// Different [[Prototype]]:
assert.deepStrictEqual(object, fakeDate);
// AssertionError: Input A expected to strictly deep-equal input B:
// + expected - actual
// - {}
// + Date {}

// Different type tags:
assert.deepStrictEqual(date, fakeDate);
// AssertionError: Input A expected to strictly deep-equal input B:
// + expected - actual
// - 2018-04-26T00:49:08.604Z
// + Date {}

assert.deepStrictEqual(NaN, NaN);
// OK, because of the SameValue comparison

// Different unwrapped numbers:
assert.deepStrictEqual(new Number(1), new Number(2));
// AssertionError: Input A expected to strictly deep-equal input B:
// + expected - actual
// - [Number: 1]
// + [Number: 2]

assert.deepStrictEqual(new String('foo'), Object('foo'));
// OK because the object and the string are identical when unwrapped.

assert.deepStrictEqual(-0, -0);
// OK

// Different zeros using the SameValue Comparison:
assert.deepStrictEqual(0, -0);
// AssertionError: Input A expected to strictly deep-equal input B:
// + expected - actual
// - 0
// + -0

const symbol1 = Symbol();
const symbol2 = Symbol();
assert.deepStrictEqual({ [symbol1]: 1 }, { [symbol1]: 1 });
// OK, because it is the same symbol on both objects.
assert.deepStrictEqual({ [symbol1]: 1 }, { [symbol2]: 1 });
// AssertionError [ERR_ASSERTION]: Input objects not identical:
// {
//   [Symbol()]: 1
// }

const weakMap1 = new WeakMap();
const weakMap2 = new WeakMap([[{}, {}]]);
const weakMap3 = new WeakMap();
weakMap3.unequal = true;

assert.deepStrictEqual(weakMap1, weakMap2);
// OK, because it is impossible to compare the entries

// Fails because weakMap3 has a property that weakMap1 does not contain:
assert.deepStrictEqual(weakMap1, weakMap3);
// AssertionError: Input A expected to strictly deep-equal input B:
// + expected - actual
//   WeakMap {
// -   [items unknown]
// +   [items unknown],
// +   unequal: true
//   }
```

Si los valores no son iguales, se arroja un `AssertionError` con una propiedad de `message` establecida igual al valor del parámetro `message`. Si el parámetro `message` no está definido, un mensaje de error predeterminado es asignado. Si el parámetro `message` es una instancia de un [`Error`][], entonces se arrojará en lugar de `AssertionError`.

## assert.doesNotReject(block\[, error\]\[, message\])

<!-- YAML
added: v10.0.0
-->

* `block` {Function|Promise}
* `error` {RegExp|Function}
* `message` {any}

Espera la promesa de `block` o, si el `block` es una función, inmediatamente llama a la función y espera la promesa de vuelta para completar. Luego comprobará que la promesa no sea rechazada.

Si el `block` es una función y arroja un error sincrónicamente `assert.doesNotReject()` devolverá una `Promise` rechazada con ese error. Si la función no devuelve una promesa, `assert.doesNotReject()` devolverá una `Promise` rechazada con un error [`ERR_INVALID_RETURN_VALUE`][]. En ambos casos se omitirá el controlador de errores.

Por favor, note: El uso de `assert.doesNotReject()` no es realmente útil porque hay muy poco beneficio por coger un rechazo y luego rechazarlo de nuevo. En su lugar, considere añadir un comentario al lado de la ruta de código específica que no deba rechazar y mantenga los mensajes de error lo más expresivos posibles.

Si se especifica, el `error` puede ser una función de [`Class`][], de [`RegExp`][] o una validación. Vea [`assert.throws()`][] para más detalles.

Besides the async nature to await the completion behaves identically to [`assert.doesNotThrow()`][].

```js
(async () => {
  await assert.doesNotReject(
    async () => {
      throw new TypeError('Wrong value');
    },
    SyntaxError
  );
})();
```

```js
assert.doesNotReject(Promise.reject(new TypeError('Wrong value')))
  .then(() => {
    // ...
  });
```

## assert.doesNotThrow(block\[, error\]\[, message\])

<!-- YAML
added: v0.1.21
changes:

  - version: v5.11.0, v4.4.5
    pr-url: https://github.com/nodejs/node/pull/2407
    description: The `message` parameter is respected now.
  - version: v4.2.0
    pr-url: https://github.com/nodejs/node/pull/3276
    description: The `error` parameter can now be an arrow function.
-->

* `block` {Function}
* `error` {RegExp|Function}
* `message` {any}

Afirma que la función `block` no arroja un error.

Por favor, note: El uso de `assert.doesNotThrow()` no es realmente útil porque hay muy poco beneficio por coger un rechazo y luego devolverlo. En su lugar, considere añadir un comentario al lado de la ruta de código específica que no deba arrojar y mantenga los mensajes de error lo más expresivos posibles.

When `assert.doesNotThrow()` is called, it will immediately call the `block` function.

Si se arroja un error y es el del mismo tipo que el especificado por el parámetro `error`, entonces se arroja un `AssertionError`. Si el error es de un tipo diferente, o si el parámetro `error` no está definido, el error se propaga de nuevo a la persona que llama.

Si se especifica, el `error` puede ser una función de [`Class`][], de [`RegExp`][] o una validación. Vea [`assert.throws()`][] para más detalles.

Lo siguiente, por ejemplo, arrojará el [`TypeError`][] porque no hay un tipo de error de coincidencia en la afirmación:

<!-- eslint-disable no-restricted-syntax -->

```js
assert.doesNotThrow(
  () => {
    throw new TypeError('Wrong value');
  },
  SyntaxError
);
```

Sin embargo, lo siguiente resultará en un `AssertionError` con el mensaje 'Se obtuvo una excepción no deseada...':

<!-- eslint-disable no-restricted-syntax -->

```js
assert.doesNotThrow(
  () => {
    throw new TypeError('Wrong value');
  },
  TypeError
);
```

Si se arroja un `AssertionError` y se provee un valor para el parámetro `message`, el valor de `message` será agregado al mensaje de `AssertionError`:

<!-- eslint-disable no-restricted-syntax -->

```js
assert.doesNotThrow(
  () => {
    throw new TypeError('Wrong value');
  },
  /Wrong value/,
  'Whoops'
);
// Arroja: AssertionError: Se obtuvo una excepción no deseada: Ups
```

## assert.equal(actual, expected[, message])

<!-- YAML
added: v0.1.21
-->

* `actual` {any}
* `expected` {any}
* `message` {any}

**Modo estricto**

Un alias de [`assert.strictEqual()`][].

**Modo legado**

> Estabilidad: 0 - Obsoleto: Use [`assert.strictEqual()`][] en su lugar.

Tests shallow, coercive equality between the `actual` and `expected` parameters using the [Abstract Equality Comparison](https://tc39.github.io/ecma262/#sec-abstract-equality-comparison) ( `==` ).

```js
const assert = require('assert');

assert.equal(1, 1);
// OK, 1 == 1
assert.equal(1, '1');
// OK, 1 == '1'

assert.equal(1, 2);
// AssertionError: 1 == 2
assert.equal({ a: { b: 1 } }, { a: { b: 1 } });
// AssertionError: { a: { b: 1 } } == { a: { b: 1 } }
```

Si los valores no son iguales, se arroja un `AssertionError` con una propiedad `message` establecida igual al valor del parámetro `message`. Si el parámetro `message` no está definido, se asigna un mensaje de error predeterminado. Si el parámetro `message` es una instancia de un [`Error`][], entonces se arrojará en lugar de `AssertionError`.

## assert.fail([message])

<!-- YAML
added: v0.1.21
-->

* `message` {any} **Default:** `'Failed'`

Arroja un `AssertionError` con el mensaje de error proporcionado o un mensaje de error predeterminado. Si el parámetro `message` es una instancia de un [`Error`][], entonces se arrojará en lugar del `AssertionError`.

```js
const assert = require('assert').strict;

assert.fail();
// AssertionError [ERR_ASSERTION]: Fallido

assert.fail('boom');
// AssertionError [ERR_ASSERTION]: boom

assert.fail(new TypeError('need array'));
// TypeError: se necesita un array
```

El uso de `assert.fail()` con más de dos argumentos es posible, pero está obsoleto. Vea abajo para más detalles.

## assert.fail(actual, expected[, message[, operator[, stackStartFunction]]])

<!-- YAML
added: v0.1.21
changes:

  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18418
    description: Calling `assert.fail()` with more than one argument is
                 deprecated and emits a warning.
-->

* `actual` {any}
* `expected` {any}
* `message` {any}
* `operator` {string} **Default:** `'!='`
* `stackStartFunction` {Function} **Default:** `assert.fail`

> Stability: 0 - Deprecated: Use `assert.fail([message])` or other assert functions instead.

If `message` is falsy, the error message is set as the values of `actual` and `expected` separated by the provided `operator`. If just the two `actual` and `expected` arguments are provided, `operator` will default to `'!='`. If `message` is provided as third argument it will be used as the error message and the other arguments will be stored as properties on the thrown object. If `stackStartFunction` is provided, all stack frames above that function will be removed from stacktrace (see [`Error.captureStackTrace`]). If no arguments are given, the default message `Failed` will be used.

```js
const assert = require('assert').strict;

assert.fail('a', 'b');
// AssertionError [ERR_ASSERTION]: 'a' != 'b'

assert.fail(1, 2, undefined, '>');
// AssertionError [ERR_ASSERTION]: 1 > 2

assert.fail(1, 2, 'fail');
// AssertionError [ERR_ASSERTION]: fail

assert.fail(1, 2, 'whoops', '>');
// AssertionError [ERR_ASSERTION]: whoops

assert.fail(1, 2, new TypeError('need array'));
// TypeError: need array
```

In the last three cases `actual`, `expected`, and `operator` have no influence on the error message.

Example use of `stackStartFunction` for truncating the exception's stacktrace:

```js
function suppressFrame() {
  assert.fail('a', 'b', undefined, '!==', suppressFrame);
}
suppressFrame();
// AssertionError [ERR_ASSERTION]: 'a' !== 'b'
//     at repl:1:1
//     at ContextifyScript.Script.runInThisContext (vm.js:44:33)
//     ...
```

## assert.ifError(value)

<!-- YAML
added: v0.1.97
changes:

  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18247
    description: Instead of throwing the original error it is now wrapped into
                 an `AssertionError` that contains the full stack trace.
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18247
    description: Value may now only be `undefined` or `null`. Before any truthy
                 input was accepted.
-->

* `value` {any}

Throws `value` if `value` is not `undefined` or `null`. This is useful when testing the `error` argument in callbacks. The stack trace contains all frames from the error passed to `ifError()` including the potential new frames for `ifError()` itself. See below for an example.

```js
const assert = require('assert').strict;

assert.ifError(null);
// OK
assert.ifError(0);
// AssertionError [ERR_ASSERTION]: ifError got unwanted exception: 0
assert.ifError('error');
// AssertionError [ERR_ASSERTION]: ifError got unwanted exception: 'error'
assert.ifError(new Error());
// AssertionError [ERR_ASSERTION]: ifError got unwanted exception: Error

// Create some random error frames.
let err;
(function errorFrame() {
  err = new Error('test error');
})();

(function ifErrorFrame() {
  assert.ifError(err);
})();
// AssertionError [ERR_ASSERTION]: ifError got unwanted exception: test error
//     at ifErrorFrame
//     at errorFrame
```

## assert.notDeepEqual(actual, expected[, message])

<!-- YAML
added: v0.1.21
changes:

  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/15001
    description: The `Error` names and messages are now properly compared
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12142
    description: The `Set` and `Map` content is also compared
  - version: v6.4.0, v4.7.1
    pr-url: https://github.com/nodejs/node/pull/8002
    description: Typed array slices are handled correctly now.
  - version: v6.1.0, v4.5.0
    pr-url: https://github.com/nodejs/node/pull/6432
    description: Objects with circular references can be used as inputs now.
  - version: v5.10.1, v4.4.3
    pr-url: https://github.com/nodejs/node/pull/5910
    description: Handle non-`Uint8Array` typed arrays correctly.
-->

* `actual` {any}
* `expected` {any}
* `message` {any}

**Modo estricto**

An alias of [`assert.notDeepStrictEqual()`][].

**Modo legado**

> Stability: 0 - Deprecated: Use [`assert.notDeepStrictEqual()`][] instead.

Tests for any deep inequality. Opposite of [`assert.deepEqual()`][].

```js
const assert = require('assert');

const obj1 = {
  a: {
    b: 1
  }
};
const obj2 = {
  a: {
    b: 2
  }
};
const obj3 = {
  a: {
    b: 1
  }
};
const obj4 = Object.create(obj1);

assert.notDeepEqual(obj1, obj1);
// AssertionError: { a: { b: 1 } } notDeepEqual { a: { b: 1 } }

assert.notDeepEqual(obj1, obj2);
// OK: obj1 and obj2 are not deeply equal

assert.notDeepEqual(obj1, obj3);
// AssertionError: { a: { b: 1 } } notDeepEqual { a: { b: 1 } }

assert.notDeepEqual(obj1, obj4);
// OK: obj1 and obj4 are not deeply equal
```

If the values are deeply equal, an `AssertionError` is thrown with a `message` property set equal to the value of the `message` parameter. If the `message` parameter is undefined, a default error message is assigned. Si el parámetro `message` es una instancia de un [`Error`][], entonces se arrojará en lugar de `AssertionError`.

## assert.notDeepStrictEqual(actual, expected[, message])

<!-- YAML
added: v1.2.0
changes:

  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/15398
    description: The `-0` and `+0` are not considered equal anymore.
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/15036
    description: The `NaN` is now compared using the
              [SameValueZero](https://tc39.github.io/ecma262/#sec-samevaluezero)
              comparison.
  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/15001
    description: The `Error` names and messages are now properly compared
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12142
    description: The `Set` and `Map` content is also compared
  - version: v6.4.0, v4.7.1
    pr-url: https://github.com/nodejs/node/pull/8002
    description: Typed array slices are handled correctly now.
  - version: v6.1.0
    pr-url: https://github.com/nodejs/node/pull/6432
    description: Objects with circular references can be used as inputs now.
  - version: v5.10.1, v4.4.3
    pr-url: https://github.com/nodejs/node/pull/5910
    description: Handle non-`Uint8Array` typed arrays correctly.
-->

* `actual` {any}
* `expected` {any}
* `message` {any}

Tests for deep strict inequality. Opposite of [`assert.deepStrictEqual()`][].

```js
const assert = require('assert').strict;

assert.notDeepStrictEqual({ a: 1 }, { a: '1' });
// OK
```

If the values are deeply and strictly equal, an `AssertionError` is thrown with a `message` property set equal to the value of the `message` parameter. If the `message` parameter is undefined, a default error message is assigned. If the `message` parameter is an instance of an [`Error`][] then it will be thrown instead of the `AssertionError`.

## assert.notEqual(actual, expected[, message])

<!-- YAML
added: v0.1.21
-->

* `actual` {any}
* `expected` {any}
* `message` {any}

**Modo estricto**

An alias of [`assert.notStrictEqual()`][].

**Modo legado**

> Stability: 0 - Deprecated: Use [`assert.notStrictEqual()`][] instead.

Tests shallow, coercive inequality with the [Abstract Equality Comparison](https://tc39.github.io/ecma262/#sec-abstract-equality-comparison) ( `!=` ).

```js
const assert = require('assert');

assert.notEqual(1, 2);
// OK

assert.notEqual(1, 1);
// AssertionError: 1 != 1

assert.notEqual(1, '1');
// AssertionError: 1 != '1'
```

If the values are equal, an `AssertionError` is thrown with a `message` property set equal to the value of the `message` parameter. If the `message` parameter is undefined, a default error message is assigned. If the `message` parameter is an instance of an [`Error`][] then it will be thrown instead of the `AssertionError`.

## assert.notStrictEqual(actual, expected[, message])

<!-- YAML
added: v0.1.21
changes:

  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/17003
    description: Used comparison changed from Strict Equality to `Object.is()`
-->

* `actual` {any}
* `expected` {any}
* `message` {any}

Tests strict inequality between the `actual` and `expected` parameters as determined by the [SameValue Comparison](https://tc39.github.io/ecma262/#sec-samevalue).

```js
const assert = require('assert').strict;

assert.notStrictEqual(1, 2);
// OK

assert.notStrictEqual(1, 1);
// AssertionError [ERR_ASSERTION]: Identical input passed to notStrictEqual: 1

assert.notStrictEqual(1, '1');
// OK
```

If the values are strictly equal, an `AssertionError` is thrown with a `message` property set equal to the value of the `message` parameter. If the `message` parameter is undefined, a default error message is assigned. Si el parámetro `message` es una instancia de un [`Error`][], entonces se arrojará en lugar de `AssertionError`.

## assert.ok(value[, message])

<!-- YAML
added: v0.1.21
changes:

  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/18319
    description: The `assert.ok()` (no arguments) will now use a predefined
                 error message.
-->

* `value` {any}
* `message` {any}

Tests if `value` is truthy. It is equivalent to `assert.equal(!!value, true, message)`.

If `value` is not truthy, an `AssertionError` is thrown with a `message` property set equal to the value of the `message` parameter. If the `message` parameter is `undefined`, a default error message is assigned. Si el parámetro `message` es una instancia de un [`Error`][], entonces se arrojará en lugar de `AssertionError`. If no arguments are passed in at all `message` will be set to the string: ``'No value argument passed to `assert.ok()`'``.

Be aware that in the `repl` the error message will be different to the one thrown in a file! Vea abajo para más detalles.

```js
const assert = require('assert').strict;

assert.ok(true);
// OK
assert.ok(1);
// OK

assert.ok();
// AssertionError: No value argument passed to `assert.ok()`

assert.ok(false, 'it\'s false');
// AssertionError: it's false

// In the repl:
assert.ok(typeof 123 === 'string');
// AssertionError: false == true

// In a file (e.g. test.js):
assert.ok(typeof 123 === 'string');
// AssertionError: The expression evaluated to a falsy value:
//
//   assert.ok(typeof 123 === 'string')

assert.ok(false);
// AssertionError: The expression evaluated to a falsy value:
//
//   assert.ok(false)

assert.ok(0);
// AssertionError: The expression evaluated to a falsy value:
//
//   assert.ok(0)

// Using `assert()` works the same:
assert(0);
// AssertionError: The expression evaluated to a falsy value:
//
//   assert(0)
```

## assert.rejects(block\[, error\]\[, message\])

<!-- YAML
added: v10.0.0
-->

* `block` {Function|Promise}
* `error` {RegExp|Function|Object|Error}
* `message` {any}

Espera la promesa de `block` o, si el `block` es una función, inmediatamente llama a la función y espera la promesa de vuelta para completar. It will then check that the promise is rejected.

If `block` is a function and it throws an error synchronously, `assert.rejects()` will return a rejected `Promise` with that error. If the function does not return a promise, `assert.rejects()` will return a rejected `Promise` with an [`ERR_INVALID_RETURN_VALUE`][] error. In both cases the error handler is skipped.

Besides the async nature to await the completion behaves identically to [`assert.throws()`][].

If specified, `error` can be a [`Class`][], [`RegExp`][], a validation function, an object where each property will be tested for, or an instance of error where each property will be tested for including the non-enumerable `message` and `name` properties.

If specified, `message` will be the message provided by the `AssertionError` if the block fails to reject.

```js
(async () => {
  await assert.rejects(
    async () => {
      throw new TypeError('Wrong value');
    },
    {
      name: 'TypeError',
      message: 'Wrong value'
    }
  );
})();
```

```js
assert.rejects(
  Promise.reject(new Error('Wrong value')),
  Error
).then(() => {
  // ...
});
```

Note that `error` cannot be a string. If a string is provided as the second argument, then `error` is assumed to be omitted and the string will be used for `message` instead. This can lead to easy-to-miss mistakes. Please read the example in [`assert.throws()`][] carefully if using a string as the second argument gets considered.

## assert.strictEqual(actual, expected[, message])

<!-- YAML
added: v0.1.21
changes:

  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/17003
    description: Used comparison changed from Strict Equality to `Object.is()`
-->

* `actual` {any}
* `expected` {any}
* `message` {any}

Tests strict equality between the `actual` and `expected` parameters as determined by the [SameValue Comparison](https://tc39.github.io/ecma262/#sec-samevalue).

```js
const assert = require('assert').strict;

assert.strictEqual(1, 2);
// AssertionError [ERR_ASSERTION]: Input A expected to strictly equal input B:
// + expected - actual
// - 1
// + 2

assert.strictEqual(1, 1);
// OK

assert.strictEqual(1, '1');
// AssertionError [ERR_ASSERTION]: Input A expected to strictly equal input B:
// + expected - actual
// - 1
// + '1'
```

If the values are not strictly equal, an `AssertionError` is thrown with a `message` property set equal to the value of the `message` parameter. If the `message` parameter is undefined, a default error message is assigned. If the `message` parameter is an instance of an [`Error`][] then it will be thrown instead of the `AssertionError`.

## assert.throws(block\[, error\]\[, message\])

<!-- YAML
added: v0.1.21
changes:

  - version: v9.9.0
    pr-url: https://github.com/nodejs/node/pull/17584
    description: The `error` parameter can now be an object as well.
  - version: v4.2.0
    pr-url: https://github.com/nodejs/node/pull/3276
    description: The `error` parameter can now be an arrow function.
-->

* `block` {Function}
* `error` {RegExp|Function|Object|Error}
* `message` {any}

Expects the function `block` to throw an error.

If specified, `error` can be a [`Class`][], [`RegExp`][], a validation function, an object where each property will be tested for, or an instance of error where each property will be tested for including the non-enumerable `message` and `name` properties.

If specified, `message` will be the message provided by the `AssertionError` if the block fails to throw.

Custom error object / error instance:

```js
const err = new TypeError('Wrong value');
err.code = 404;

assert.throws(
  () => {
    throw err;
  },
  {
    name: 'TypeError',
    message: 'Wrong value'
    // Note that only properties on the error object will be tested!
  }
);

// Fails due to the different `message` and `name` properties:
assert.throws(
  () => {
    const otherErr = new Error('Not found');
    otherErr.code = 404;
    throw otherErr;
  },
  err // This tests for `message`, `name` and `code`.
);
```

Validate instanceof using constructor:

```js
assert.throws(
  () => {
    throw new Error('Wrong value');
  },
  Error
);
```

Validate error message using [`RegExp`][]:

Using a regular expression runs `.toString` on the error object, and will therefore also include the error name.

```js
assert.throws(
  () => {
    throw new Error('Wrong value');
  },
  /^Error: Wrong value$/
);
```

Custom error validation:

```js
assert.throws(
  () => {
    throw new Error('Wrong value');
  },
  function(err) {
    if ((err instanceof Error) && /value/.test(err)) {
      return true;
    }
  },
  'unexpected error'
);
```

Note that `error` cannot be a string. If a string is provided as the second argument, then `error` is assumed to be omitted and the string will be used for `message` instead. This can lead to easy-to-miss mistakes. Using the same message as the thrown error message is going to result in an `ERR_AMBIGUOUS_ARGUMENT` error. Please read the example below carefully if using a string as the second argument gets considered:

<!-- eslint-disable no-restricted-syntax -->

```js
function throwingFirst() {
  throw new Error('First');
}
function throwingSecond() {
  throw new Error('Second');
}
function notThrowing() {}

// The second argument is a string and the input function threw an Error.
// The first case will not throw as it does not match for the error message
// thrown by the input function!
assert.throws(throwingFirst, 'Second');
// In the next example the message has no benefit over the message from the
// error and since it is not clear if the user intended to actually match
// against the error message, Node.js thrown an `ERR_AMBIGUOUS_ARGUMENT` error.
assert.throws(throwingSecond, 'Second');
// Throws an error:
// TypeError [ERR_AMBIGUOUS_ARGUMENT]

// The string is only used (as message) in case the function does not throw:
assert.throws(notThrowing, 'Second');
// AssertionError [ERR_ASSERTION]: Missing expected exception: Second

// If it was intended to match for the error message do this instead:
assert.throws(throwingSecond, /Second$/);
// Does not throw because the error messages match.
assert.throws(throwingFirst, /Second$/);
// Throws an error:
// Error: First
//     at throwingFirst (repl:2:9)
```

Due to the confusing notation, it is recommended not to use a string as the second argument. This might lead to difficult-to-spot errors.