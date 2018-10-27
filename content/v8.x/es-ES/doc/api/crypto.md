# Crypto

<!--introduced_in=v0.3.6-->

> Estabilidad 2 - Estable

El módulo `crypto` provee funcionalidad criptográfica que incluye un conjunto de empaquetadores para hash de OpenSSL, HMAC, cifrar, descifrar, firmar, y verificar funciones.

Use `require('crypto')` para acceder a este módulo.

```js
const crypto = require('crypto');

const secret = 'abcdefg';
const hash = crypto.createHmac('sha256', secret)
                   .update('I love cupcakes')
                   .digest('hex');
console.log(hash);
// Imprime:
//   c0fa1bc00531bd78ef38c628449c5102aeabd49b5dc3a2a516ea6ea959d6658e
```

## Determinar si el soporte de crypto está desactivado

Es posible que Node.js sea compilado sin incluir soporte para el módulo `crypto`. En estos casos, llamar al comando `require('crypto')` arrojará como resultado un error.

```js
let crypto;
try {
  crypto = require('crypto');
} catch (err) {
  console.log('crypto support is disabled!');
}
```

## Clase: Certificado

<!-- YAML
added: v0.11.8
-->

SPKAC es un mecanismo de Solicitud de Firma de Certificado implementado originalmente por Netscape y, en la actualidad, especificado formalmente como parte de [HTML5's `keygen` element][].

El módulo `crypto` provee la clase `Certificate` para trabajar con datos SPKAC. El uso más común es la salida handling generada por el elemento HTML5 `<keygen>`. Node.js usa [OpenSSL's SPKAC implementation](https://www.openssl.org/docs/man1.0.2/apps/spkac.html) internamente.

### nuevo crypto.Certificate()

Instancias del tipo de `Certificate` pueden ser creadas usando la palabra `new` o, llamando `crypto.Certificate()` como una función:

```js
const crypto = require('crypto');

const cert1 = new crypto.Certificate();
const cert2 = crypto.Certificate();
```

### certificate.exportChallenge(spkac)

<!-- YAML
added: v0.11.8
-->

- `spack` {string | Buffer | TypedArray | DataView}
- Devuelve al {Buffer} el componente desafío de la estructura de datos `spkac`, que incluye una clave pública y un desafío.

```js
const cert = require('crypto').Certificate();
const spkac = getSpkacSomehow();
const challenge = cert.exportChallenge(spkac);
console.log(challenge.toString('utf8'));
// Imprime: el desafío como una string en UTF8
```

### certificate.exportPublicKey(spkac)

<!-- YAML
added: v0.11.8
-->

- `spack` {string | Buffer | TypedArray | DataView}
- Devuelve al {Buffer} el componente público de la estructura de datos `spkac`, que incluye una clave pública y un desafío.

```js
const cert = require('crypto').Certificate();
const spkac = getSpkacSomehow();
const publicKey = cert.exportPublicKey(spkac);
console.log(publicKey);
// Imprime: la clave pública como <Buffer ...>
```

### certificate.verifySpkac(spkac)

<!-- YAML
added: v0.11.8
-->

- `spkac` {Buffer | TypedArray | DataView}
- Devuelve el {boolean} `true` si la estructura de datos `spkac` dada es válida, `false` si no lo es.

```js
const cert = require('crypto').Certificate();
const spkac = getSpkacSomehow();
console.log(cert.verifySpkac(Buffer.from(spkac)));
// Imprime: “true” o “false”
```

## Clase: Cipher

<!-- YAML
added: v0.1.94
-->

Las instancias de la clase `Cipher` son usadas para encriptar datos. La clase puede ser empleada en una de dos formas:

- Como un [stream](stream.html) que es tanto legible como grabable en donde los datos sencillos desencriptados son escritos para producir datos encriptados en lado legible; o,
- Usando los métodos [`cipher.update()`][] y [`cipher.final()`][] para producir los datos encriptados.

Los métodos [`crypto.createCipher()`][] o [`crypto.createCipheriv()`][] son usados para crear las instancias de `Cipher`. Los objetos `Cipher` no son creados directamente al usar la palabra clave `new`.

Ejemplo: Usar los objetos `Cipher` como streams:

```js
const crypto = require('crypto');
const cipher = crypto.createCipher('aes192', 'a password');

let encrypted = '';
cipher.on('readable', () => {
  const data = cipher.read();
  if (data)
    encrypted += data.toString('hex');
});
cipher.on('end', () => {
  console.log(encrypted);
  // Imprime: ca981be48e90867604588e75d04feabb63cc007a8f8ad89b10616ed84d815504
});

cipher.write('some clear text data');
cipher.end();
```

Ejemplo: Usando `Cipher` y los streams canalizados:

```js
const crypto = require('crypto');
const fs = require('fs');
const cipher = crypto.createCipher('aes192', 'a password');

const input = fs.createReadStream('test.js');
const output = fs.createWriteStream('test.enc');

input.pipe(cipher).pipe(output);
```

Ejemplo: Usando los métodos [`cipher.update()`][] y [`cipher.final()`][]:

```js
const crypto = require('crypto');
const cipher = crypto.createCipher('aes192', 'a password');

let encrypted = cipher.update('some clear text data', 'utf8', 'hex');
encrypted += cipher.final('hex');
console.log(encrypted);
// Imprime: ca981be48e90867604588e75d04feabb63cc007a8f8ad89b10616ed84d815504
```

### cipher.final([outputEncoding])

<!-- YAML
added: v0.1.94
-->

- `outputEncoding` {string}

Regresa cualquiera de los contenidos restantes cifrados. Un string es regresado si el parámetro `output_encoding` es uno de los ` 'latin1'`, `'base64'` o `'hex'`. Si no se provee una `outputEncoding`, un [`Buffer`][] es devuelto.

El objeto `Cipher` no puede ser utilizado para encriptar datos una vez que el método `cipher.final()` ha sido llamado. Y, arrojará un error al hacer varios intentos para llamar al `cipher.final()` más de una vez.

### cipher.setAAD(buffer)

<!-- YAML
added: v1.0.0
-->

- `buffer` {Buffer}
- Devuelve al {Cipher} como método cadena.

El método `cipher.setAAD()` establece el valor empleado por el parámetro de entrada *additional authenticated data* (AAD) cuando se use un modo autenticado de encriptación (solo el `GCM` es válido actualmente).

El método `cipher.setAAD()` debe llamarse antes que [`cipher.update()`][].

### cipher.getAuthTag()

<!-- YAML
added: v1.0.0
-->

El método `cipher.getAuthTag()` regresa a un [`Buffer`][] que contiene la *etiqueta de autenticación* que ha sido computada por los datos dados cuando es un modo de encriptación autenticado (solo `GCM` está actualmente avalado).

El método `cipher.getAuthTag()` solo debría ser llamado luego de haber completado la encriptación al usar el método [`cipher.final()`][].

### cipher.setAutoPadding([autoPadding])

<!-- YAML
added: v0.7.1
-->

- `autoPadding` {boolean} Por defecto: `true`.
- Devuelve al {Cipher} como método cadena.

El tipo de `Cipher` se añadirá como relleno automáticamente para el ingreso de datos al tamaño del bloque apropiado. Para inhabilitar el relleno por defecto, llame a `cipher.setAutoPadding(false)`.

Cuando `autoPadding` es `false`, la longitud de los datos de entrada debe ser un múltiplo del tamaño del bloque cipher o [`cipher.final()`][] arrojará un error. Desabilitar el relleno automático es útil para un relleno atípico, por lo que se puede usar `0x0` en vez del relleno PKCS.

El método `cipher.setAutoPadding()` debe llamarse antes del [`cipher.final()`][].

### cipher.update(data\[, input_encoding\]\[, output_encoding\])

<!-- YAML
added: v0.1.94
changes:

  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default `inputEncoding` changed from `binary` to `utf8`.
-->

- `data` {string | Buffer | TypedArray | DataView}
- `inputEncoding` {string}
- `outputEncoding` {string}

Actualiza el cifrado con `data`. Si el argumento `inputEncoding` es dado, su valor debe ser `'utf8'`, `'ascii'`, o `'latin1'` y el argumento `data` es una string usando el código específico. Si el argumento `inputEncoding` no es dado, `data` debe ser un [`Buffer`][], `TypedArray`, o `DataView`. Si `data` es un [`Buffer`][], `TypedArray`, o `DataView`, entonces se omite `inputEncoding`.

El `outputEncoding` especifica el formato de salida de los datos cifrados, y puede ser `'latin1'`, `'base64'` o `'hex'`. Si el `outputEncoding` es especificado, se devuelve una string que usa el código especificado. Si no se provee un `outputEncoding`, un [`Buffer`][] es devuelto.

El método `cipher.update()` puede ser llamado múltiples veces con nuevos datos hasta que se llame a [`cipher.final()`][]. Si se llama a `cipher.update()` después de [`cipher.final()`][] se producirá un error.

## Clase: Decipher

<!-- YAML
added: v0.1.94
-->

Instancias de la clase `Decipher` son usadas para descifrar datos. La clase puede ser empleada en una de dos formas:

- Como un [stream](stream.html) que es legible y escribible, donde los datos planos encriptados están escritos para producir datos no encriptados en el lado legible, o
- Usando los métodos [`decipher.update()`][] y [`decipher.final()`][] para producir datos no encriptados.

Los métodos [`crypto.createDecipher()`][] o [`crypto.createDecipheriv()`][] son usados para crear instancias `Decipher`. Los objetos `Decipher` no deben crearse directamente usando la palabra clave `new`.

Ejemplo: Usando objetos `Decipher` como streams:

```js
const crypto = require('crypto'); 
const decipher = crypto.createDecipher('aes192', 'a password'); 

let decrypted = ''; 
decipher.on('readable', () => {   
  const data = decipher.read();   
  if (data)     
    decrypted += data.toString('utf8'); 
}); 
decipher.on('end', () => {   
  console.log(decrypted);   
  // Imprime: algunos datos de texto claros 
}); 

const encrypted =                       
    'ca981be48e90867604588e75d04feabb63cc007a8f8ad89b10616ed84d815504'; decipher.write(encrypted, 'hex'); 
decipher.end();
```

Ejemplo: Usando `Cipher` y los streams canalizados:

```js
const crypto = require('crypto');
const fs = require('fs');
const decipher = crypto.createDecipher('aes192', 'a password');

const input = fs.createReadStream('test.enc');
const output = fs.createWriteStream('test.js');

input.pipe(decipher).pipe(output);
```

Ejemplo: Usando los métodos [`decipher.update()`][] y [`decipher.final()`][]:

```js
const crypto = require('crypto'); 
const decipher = crypto.createDecipher('aes192', 'a password'); 

const encrypted =
    'ca981be48e90867604588e75d04feabb63cc007a8f8ad89b10616ed84d815504'; 
let decrypted = decipher.update(encrypted, 'hex', 'utf8'); 
decrypted += decipher.final('utf8'); 
console.log(decrypted); 
// Imprime: algunos datos limpios
```

### decipher.final([outputEncoding])

<!-- YAML
added: v0.1.94
-->

- `outputEncoding` {string}

Regresa cualquiera de los contenidos restantes descifrados. Un string es regresado si el parámetro `output_encoding` es `'latin1'`, `'base64'` o `'hex'`. Si no se provee una `outputEncoding`, un [`Buffer`][] es devuelto.

Una vez que el método `decipher.final()` ha sido llamado, el objeto `Decipher` no puede ser usado para descifrar datos. Intentar llamar mas de una vez a `decipher.final()` producirá un error.

### decipher.setAAD(buffer)

<!-- YAML
added: v1.0.0
changes:

  - version: v7.2.0
    pr-url: https://github.com/nodejs/node/pull/9398
    description: This method now returns a reference to `decipher`.
-->

- `buffer` {Buffer | TypedArray | DataView}
- Devuelve al {Cipher} como método cadena.

El método `decipher.setAAD()` establece el valor empleado por el parámetro de entrada *additional authenticated data* (AAD) cuando se use un modo autenticado de encriptación (solo el `GCM` es válido actualmente).

El método `decipher.setAAD()` debe llamarse antes de [`decipher.update()`][].

### decipher.setAuthTag(buffer)

<!-- YAML
added: v1.0.0
changes:

  - version: v7.2.0
    pr-url: https://github.com/nodejs/node/pull/9398
    description: This method now returns a reference to `decipher`.
-->

- `buffer` {Buffer | TypedArray | DataView}
- Devuelve al {Cipher} como método cadena.

El método `decipher.setAuthTag()` es usado para pasar la *etiqueta de autenticación recibida* cuando se usa un modo de encriptación autenticado (solamente `GCM` y <0>CCM</0> están siendo respaldados en la actualiad). Si no se provee ninguna etiqueta o el texto cifrado ha sido manipulado, va a arrojar [`decipher.final()`][], indicando que el texto cifrado debe descartarse por una autenticación fallida.

El método `decipher.setAuthTag()` debe ser llamado antes de [`decipher.final()`][].

### decipher.setAutoPadding([autoPadding])

<!-- YAML
added: v0.7.1
-->

- `autoPadding` {boolean} Por defecto: `true`.
- Devuelve al {Cipher} como método cadena.

Cuando los datos han sido encriptados sin un llenado de bloques estándar, llamar al `decipher.setAutoPadding(false)` deshabilitará automáticamente el llenado automático para prevenir a [`decipher.final()`][] de verificar y remover el llenado.

Desactivar el llenado automático solo funcionara si la longitud de datos de entrada es un múltiplo del tamaño de bloque de los cifrados.

El método `decipher.setAutoPadding()` debe ser llamado antes de [`decipher.final()`][].

### decipher.update(data\[, inputEncoding\]\[, outputEncoding\])

<!-- YAML
added: v0.1.94
changes:

  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default `inputEncoding` changed from `binary` to `utf8`.
-->

- `data` {string | Buffer | TypedArray | DataView}
- `inputEncoding` {string}
- `outputEncoding` {string}

Actualiza el descifrado con `data`. Si el argumento `inputEncoding` es dado, su valor debe ser `'latin1'`, `'base64'`, o `'hex'` y el argumento `data` es una string que utiliza la codificación especificada. Si el argumento `inputEncoding` no es dado, `data` debe ser un [`Buffer`][]. Si `data` es un [`Buffer`][] entonces `inputEncoding` es ignorado.

El `outputEncoding` especifica el formato de salida de los datos cifrados, y puede ser `'latin1'`, `'ascii'` o `'utf8'`. Si el `outputEncoding` es especificado, se devuelve una string que usa el código especificado. Si no se provee un `outputEncoding`, un [`Buffer`][] es devuelto.

El método `decipher.update()` puede ser llamado múltiples veces con nuevos datos hasta que sea llamado [`decipher.final()`][]. Pero, si se llama a `decipher.update()` después de [`decipher.final()`][] se producirá un error.

## Clase: DiffieHellman

<!-- YAML
added: v0.5.0
-->

La clase `DiffieHellman` es útil para crear intercambios en la clave Diffie-Hellman.

Las instancias de la clase `DiffieHellman` pude ser creada usando la función [`rypto.createDiffieHellman()`][].

```js
const crypto = require('crypto');
const assert = require('assert');

// Genera las llaves de Alice...
const alice = crypto.createDiffieHellman(2048);
const aliceKey = alice.generateKeys();

// Genera las llaves de Bob...
const bob = crypto.createDiffieHellman(alice.getPrime(), alice.getGenerator());
const bobKey = bob.generateKeys();

// intercambian y generan el secreto...
const aliceSecret = alice.computeSecret(bobKey);
const bobSecret = bob.computeSecret(aliceKey);

// OK
assert.strictEqual(aliceSecret.toString('hex'), bobSecret.toString('hex'));
```

### diffieHellman.computeSecret(otherPublicKey\[, inputEncoding\]\[, outputEncoding\])

<!-- YAML
added: v0.5.0
-->

- `otherPublicKey` {string | Buffer | TypedArray | DataView}
- `inputEncoding` {string}
- `outputEncoding` {string}

Computa el secreto compartido usando `otherPublicKey` como la clave pública de la otra parte y devuelve el secreto compartido computado. The supplied key is interpreted using the specified `inputEncoding`, and secret is encoded using specified `outputEncoding`. Los códigos pueden ser `'latin1'`, `'hex'`, o `'base64'`. If the `inputEncoding` is not provided, `otherPublicKey` is expected to be a [`Buffer`][], `TypedArray`, or `DataView`.

If `outputEncoding` is given a string is returned; otherwise, a [`Buffer`][] is returned.

### diffieHellman.generateKeys([encoding])

<!-- YAML
added: v0.5.0
-->

- `encoding` {string}

Genera valores de claves Diffie-Hellman privadas y públicas, y devuelve la clave pública en el `encoding` especificado. Esta clave debe ser transferida a la otra parte. Los códigos pueden ser `'latin1'`, `'hex'`, o `'base64'`. Si `encoding` es dado, una string es devuelta; de no ser así, un [`Buffer`][] es devuelto.

### diffieHellman.getGenerator([encoding])

<!-- YAML
added: v0.5.0
-->

- `encoding` {string}

Devuelve el generador de Diffie-Hellman al `encoding` especificado, el cual puede ser `'latin1'`, `'hex'`, o `'base64'`. If `encoding` is provided a string is returned; otherwise a [`Buffer`][] is returned.

### diffieHellman.getPrime([encoding])

<!-- YAML
added: v0.5.0
-->

- `encoding` {string}

Regresa el Diffie-Hellman prime al `encoding` especificado, el cual puede ser `'latin1'`, `'hex'`, o `'base64'`. Si `encoding` es dado, una string es devuelta; de no ser así un [`Buffer`][] es devuelto.

### diffieHellman.getPrivateKey([encoding])

<!-- YAML
added: v0.5.0
-->

- `encoding` {string}

Regresa la llave privada Diffie-Hellman en el `encoding` especificado, el cual puede ser `'latin1'`, `'hex'`, o `'base64'`. Si `encoding` es dado, una string es devuelta; de no ser así un [`Buffer`][] es devuelto.

### diffieHellman.getPublicKey([encoding])

<!-- YAML
added: v0.5.0
-->

- `encoding` {string}

Regresa la llave pública Diffie-Hellman al `encoding` especificado, el cual puede ser `'latin1'`, `'hex'`, o `'base64'`. Si `encoding` es dado, una string es devuelta; de no ser así un [`Buffer`][] es devuelto.

### diffieHellman.setPrivateKey(privateKey[, encoding])

<!-- YAML
added: v0.5.0
-->

- `privateKey` {string | Buffer | TypedArray | DataView}
- `encoding` {string}

Establece la clave privada Diffie-Hellman. If the `encoding` argument is provided and is either `'latin1'`, `'hex'`, or `'base64'`, `privateKey` is expected to be a string. If no `encoding` is provided, `privateKey` is expected to be a [`Buffer`][], `TypedArray`, or `DataView`.

### diffieHellman.setPublicKey(publicKey[, encoding])

<!-- YAML
added: v0.5.0
-->

- `publicKey` {string | Buffer | TypedArray | DataView}
- `encoding` {string}

Establece la clave pública Diffie-Hellman. If the `encoding` argument is provided and is either `'latin1'`, `'hex'` or `'base64'`, `publicKey` is expected to be a string. If no `encoding` is provided, `publicKey` is expected to be a [`Buffer`][], `TypedArray`, or `DataView`.

### diffieHellman.verifyError

<!-- YAML
added: v0.11.12
-->

Un campo de bits que contiene advertencias y/o errores que resultan de un chequeo realizado durante la inicialización del objeto `DiffieHellman`.

Los valores a continuación son válidos para esta propiedad (como es definido en el módulo`constants`):

- `DH_CHECK_P_NOT_SAFE_PRIME`
- `DH_CHECK_P_NOT_PRIME`
- `DH_UNABLE_TO_CHECK_GENERATOR`
- `DH_NOT_SUITABLE_GENERATOR`

## Clase: ECDH

<!-- YAML
added: v0.11.14
-->

La clase `ECDH` es una utilidad para crear la Curva Elíptica Diffie-Hellman (ECDH) de intercambios de claves.

Las instancias de clase `ECDH` pueden ser creadas usando la función [`crypto.createECDH()`][].

```js
const crypto = require('crypto');
const assert = require('assert');

// Genera las llaves de Alice...
const alice = crypto.createDiffieHellman(2048);
const aliceKey = alice.generateKeys();

// Generar las llaves de Bob...
const bob = crypto.createECDH('secp521r1');
const bobKey = bob.generateKeys();

// Intercambiar y generar el secreto...
const aliceSecret = alice.computeSecret(bobKey); 
const bobSecret = bob.computeSecret(aliceKey);
 
assert.strictEqual(aliceSecret.toString('hex'), bobSecret.toString('hex')); 
// OK
```

### ecdh.computeSecret(otherPublicKey\[, inputEncoding\]\[, outputEncoding\])

<!-- YAML
added: v0.11.14
changes:

  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default `inputEncoding` changed from `binary` to `utf8`.
-->

- `otherPublicKey` {string | Buffer | TypedArray | DataView}
- `inputEncoding` {string}
- `outputEncoding` {string}

Computa el secreto compartido usando `otherPublicKey` como la clave pública de la otra parte y devuelve el secreto compartido computado. The supplied key is interpreted using specified `inputEncoding`, and the returned secret is encoded using the specified `outputEncoding`. Los códigos pueden ser `'latin1'`, `'hex'`, o `'base64'`. If the `inputEncoding` is not provided, `otherPublicKey` is expected to be a [`Buffer`][], `TypedArray`, or `DataView`.

If `outputEncoding` is given a string will be returned; otherwise a [`Buffer`][] is returned.

### ecdh.generateKeys([encoding[, format]])

<!-- YAML
added: v0.11.14
-->

- `encoding` {string}
- `format` {string} Defaults to `uncompressed`.

Genera los valores de la clave EC Diffie-Hellman privados y públicos, y devuelve la clave pública en el `format` y `encoding` especificado. Esta clave debe ser transferida a la otra parte.

The `format` argument specifies point encoding and can be `'compressed'` or `'uncompressed'`. If `format` is not specified, the point will be returned in `'uncompressed'` format.

El argumento `encoding` puede ser `'latin1'`, `'hex'`, o `'base64'`. Si `encoding` es dado, una string es devuelta; de no ser así un [`Buffer`][] es devuelto.

### ecdh.getPrivateKey([encoding])

<!-- YAML
added: v0.11.14
-->

- `encoding` {string}

Regresa la llave privada EC Diffie-Hellman en el `encoding` especificado, el cual puede ser `'latin1'`, `'hex'`, o `'base64'`. Si `encoding` es dado, una string es devuelta; de no ser así un [`Buffer`][] es devuelto.

### ecdh.getPublicKey(\[encoding\]\[, format\])

<!-- YAML
added: v0.11.14
-->

- `encoding` {string}
- `format` {string} Defaults to `uncompressed`.

Devuelve la clave pública EC Diffie-Hellman en el `encoding` y `format` especificado.

The `format` argument specifies point encoding and can be `'compressed'` or `'uncompressed'`. If `format` is not specified the point will be returned in `'uncompressed'` format.

El argumento `encoding` puede ser `'latin1'`, `'hex'`, o `'base64'`. Si `encoding` es especificado, una string es devuelta; de no ser así un [`Buffer`][] es devuelto.

### ecdh.setPrivateKey(privateKey[, encoding])

<!-- YAML
added: v0.11.14
-->

- `privateKey` {string | Buffer | TypedArray | DataView}
- `encoding` {string}

Establece la clave privada EC Diffie-Hellman. El `encoding` puede ser `'latin1'`, `'hex'` o `'base64'`. If `encoding` is provided, `privateKey` is expected to be a string; otherwise `privateKey` is expected to be a [`Buffer`][], `TypedArray`, or `DataView`.

If `privateKey` is not valid for the curve specified when the `ECDH` object was created, an error is thrown. Sobre la configuración de la clave privada, el punto público asociado (clave) es también generado y establecido en el objeto ECDH.

### ecdh.setPublicKey(publicKey[, encoding])

<!-- YAML
added: v0.11.14
deprecated: v5.2.0
-->

> Estabilidad: 0 - Desaprobado

- `publicKey` {string | Buffer | TypedArray | DataView}
- `encoding` {string}

Establece la clave pública EC Diffie-Hellman. La codificación de claves puede ser `'latin1'`, `'hex'` o `'base64'`. If `encoding` is provided `publicKey` is expected to be a string; otherwise a [`Buffer`][], `TypedArray`, or `DataView` is expected.

Note que no hay normalmente una razón para llamar a este método porque `ECDH` solo requiere una clave privada y la clave pública de la otra parte para computar el secreto compartido. Tipicamente, [`ecdh.generateKeys()`][] o [`ecdh.setPrivateKey()`][] serán llamados. El método [`ecdh.setPrivateKey()`][] intenta generar la clave/punto público asociado con la clave privada que se está configurando.

Ejemplo (obteniendo un secreto compartido):

```js
const crypto = require('crypto'); 
const alice = crypto.createECDH('secp256k1'); 
const bob = crypto.createECDH('secp256k1');

// Note: Esta es una forma de acceso directo para especificar una de las anteriores
// claves privadas de Alice. Sería poco inteligente usar una clave privada predecible en una 
// aplicación real.
const crypto = require('crypto');
const hash = crypto.createHash('sha256');

hash.on('readable', () => {
  const data = hash.read();
  if (data) {
    console.log(data.toString('hex'));
    // Imprime:
    //   6a2da20943931e9834fc12cfe5bb47bbd9ae43489a30726962b576f4e3993e50
  }
});

hash.write('some data to hash');
hash.end();
```

## Clase: Hash

<!-- YAML
added: v0.1.92
-->

La clase `Hash` es una utilidad para crear resúmenes hash de datos. Puede ser usado de una de las dos maneras:

- Como una [stream](stream.html) que es legible y escribible, donde los datos son escritos para producir un resumen hash computado en el lado legible, o
- Usando el método [`hash.update()`][] y [`hash.digest()`][] para producir el hash computado.

El método [`crypto.createHash()`][] es usado para crear las instancias `Hash`. Los objetos `Hash` no deben crearse directamente usando la palabra clave `new`.

Ejemplo: Usando los objetos `Hash` como streams:

```js
const crypto = require('crypto');
const hash = crypto.createHash('sha256');

hash.on('readable', () => {
  const data = hash.read();
  if (data) {
    console.log(data.toString('hex'));
    // Prints:
    //   6a2da20943931e9834fc12cfe5bb47bbd9ae43489a30726962b576f4e3993e50
  }
});

hash.write('some data to hash');
hash.end();
```

Ejemplo: Usando `Hash` y piped streams:

```js
const crypto = require('crypto');
const fs = require('fs');
const hash = crypto.createHash('sha256');

const input = fs.createReadStream('test.js');
input.pipe(hash).pipe(process.stdout);
```

Ejemplo: Usando los métodos [`hash.update()`][] y [`hash.digest()`][]:

```js
const crypto = require('crypto');
const hash = crypto.createHash('sha256');

hash.update('some data to hash');
console.log(hash.digest('hex'));
// Imprime:
//   6a2da20943931e9834fc12cfe5bb47bbd9ae43489a30726962b576f4e3993e50
```

### hash.digest([encoding])

<!-- YAML
added: v0.1.92
-->

- `encoding` {string}

Calcula el resumen de todos los datos pasados para ser hashed (usando el método [`hash.update()`][]). El `encoding` puede ser `'hex'`, `'latin1'` o `'base64'`. Si`encoding` es dado, una string será devuelta; de no ser así un [`Buffer`][] es devuelto.

El objeto `Hash` no puede ser usado nuevamente después de que el método `hash.digest()` ha sido llamado. Llamados múltiples causarán que un error sea arrojado.

### hash.update(data[, inputEncoding])

<!-- YAML
added: v0.1.92
changes:

  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default `inputEncoding` changed from `binary` to `utf8`.
-->

- `data` {string | Buffer | TypedArray | DataView}
- `inputEncoding` {string}

Updates the hash content with the given `data`, the encoding of which is given in `inputEncoding` and can be `'utf8'`, `'ascii'` or `'latin1'`. Si `encoding` no es dado, y los `data` son una string, se aplica un código de `'utf8'`. If `data` is a [`Buffer`][], `TypedArray`, or `DataView`, then `inputEncoding` is ignored.

Esto puede ser llamado muchas veces con nuevos datos a medida en que son streamed.

## Clase: Hmac

<!-- YAML
added: v0.1.94
-->

La clase `Hmac` es una utilidad para crear resúmenes criptográficos HMAC. Puede ser usado de una de las dos maneras:

- Como una [stream](stream.html) que es tanto legible como escribible, donde los datos son escritos para producir un resúmen computado HMAC en el lado legible, o
- Usando los métodos [`hmac.update()`][] y [`hmac.digest()`][] para producir el resúmen HMAC.

El método [`crypto.createHmac()`][] es usado para crear instancias `Hmac`. Los objetos `Hmac` no deben crearse directamente usando la palabra clave `new`.

Ejemplo: Usando objetos `Hmac` como streams:

```js
const crypto = require('crypto');
const hmac = crypto.createHmac('sha256', 'a secret');

hmac.on('readable', () => {
  const data = hmac.read();
  if (data) {
    console.log(data.toString('hex'));
    // Imprime:
    //   7fd04df92f636fd450bc841c9418e5825c17f33ad9c87c518115a45971f7f77e
  }
});

hmac.write('some data to hash');
hmac.end();
```

Ejemplo: Usando `Hmac` y piped streams:

```js
const crypto = require('crypto');
const fs = require('fs');
const hmac = crypto.createHmac('sha256', 'a secret');

const input = fs.createReadStream('test.js');
input.pipe(hmac).pipe(process.stdout);
```

Ejemplo: Usando los métodos [`hmac.update()`][] y [`hmac.digest()`][]:

```js
const crypto = require('crypto');
const hmac = crypto.createHmac('sha256', 'a secret');

hmac.update('some data to hash');
console.log(hmac.digest('hex'));
// Imprime:
//   7fd04df92f636fd450bc841c9418e5825c17f33ad9c87c518115a45971f7f77e
```

### hmac.digest([encoding])

<!-- YAML
added: v0.1.94
-->

- `encoding` {string}

Calcula el resúmen HMAC de todos los datos pasados usando [`hmac.update()`][]. El `encoding` puede ser `'hex'`, `'latin1'` o `'base64'`. Si `encoding` es dado, una string es devuelta; de no ser así, un [`Buffer`][] es devuelto;

El objeto `Hmac` no puede ser usado nuevamente después de que `hmac.digest()` ha sido llamado. Llamadas múltiples a `hmac.digest()` producirá un error.

### hmac.update(data[, inputEncoding])

<!-- YAML
added: v0.1.94
changes:

  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default `inputEncoding` changed from `binary` to `utf8`.
-->

- `data` {string | Buffer | TypedArray | DataView}
- `inputEncoding` {string}

Updates the `Hmac` content with the given `data`, the encoding of which is given in `inputEncoding` and can be `'utf8'`, `'ascii'` or `'latin1'`. Si `encoding` no es dado, y los `data` son una string, se aplica un código de `'utf8'`. If `data` is a [`Buffer`][], `TypedArray`, or `DataView`, then `inputEncoding` is ignored.

Esto puede ser llamado muchas veces con nuevos datos a medida en que son streamed.

## Clase: Sign

<!-- YAML
added: v0.1.92
-->

La clase `Sign` es una utilidad para generar firmas. Puede ser usada de una de las dos maneras:

- Como una [stream](stream.html) escribible, donde los datos a ser firmados están escritos y el método [`sign.sign()`][] es usado para generar y devolver la firma, o
- Usando los métodos [`sign.update()`][] y [`sign.sign()`][] para producir la firma.

El método [`crypto.createSign()`][] es usado para crear instancias `Sign`. El argumento es el nombre de la string de la función hash a utilizar. Los objetos `Sign` no son creados usando directamente la palabra clave `new`.

Ejemplo: Usando objetos`Sign` como streams:

```js
const crypto = require('crypto');
const sign = crypto.createSign('SHA256');

sign.write('some data to sign');
sign.end();

const privateKey = getPrivateKeySomehow();
console.log(sign.sign(privateKey, 'hex'));
// Imprime: la firma calculada usando la clave privada especificada y el
// SHA-256. Para las llaves RSA, el algoritmo es RSASSA-PKCS1-v1_5 (ver el parámetro
// padding a continuación para RSASSA-PSS). El algoritmo es ECDSA para las llaves EC.
```

Ejemplo: Usando los métodos [`sign.update()`][] y [`sign.sign()`][]:

```js
const crypto = require('crypto');
const sign = crypto.createSign('SHA256');

sign.update('some data to sign');

const privateKey = getPrivateKeySomehow();
console.log(sign.sign(privateKey, 'hex'));
// Imprime: la firma calculada
```

En algunos casos, una instancia `Sign` también puede ser creada pasando un nombre de algoritmo de firma, como lo es 'RSA-SHA256'. Esto va a usar el algoritmo de resumen correspondiente. Esto no funciona para todos los algoritmos de firmas, tales como 'ecdsa-with-SHA256'. En su lugar, use nombres resumidos.

Ejemplo: firma usando el nombre del algoritmo de firma heredado

```js
const crypto = require('crypto');
const sign = crypto.createSign('RSA-SHA256');

sign.update('some data to sign');

const privateKey = getPrivateKeySomehow();
console.log(sign.sign(privateKey, 'hex'));
// Imprime: la firma calculada
```

### sign.sign(privateKey[, outputFormat])

<!-- YAML
added: v0.1.92
changes:

  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11705
    description: Support for RSASSA-PSS and additional options was added.
-->

- `privateKey` {string | Object} 
  - `key` {string}
  - `passphrase` {string}
- `outputFormat` {string}

Calcula la firma de todos los datos pasados usando [`sign.update()`][] o [`sign.write()`](stream.html#stream_writable_write_chunk_encoding_callback).

The `privateKey` argument can be an object or a string. If `privateKey` is a string, it is treated as a raw key with no passphrase. If `privateKey` is an object, it must contain one or more of the following properties:

- `key`: {string} - Clave privada con codificación PEM (requerida)
- `passphrase`: {string} - frase de contraseña para la clave privada
- `padding`: {integer} - Valor de padding opcional para RSA, uno de los siguientes:
  
  - `crypto.constants.RSA_PKCS1_PADDING` (por defecto)
  - `crypto.constants.RSA_PKCS1_PSS_PADDING`
  
  Tenga en cuenta que `RSA_PKCS1_PSS_PADDING` va a usar MGF1 con la misma función hash usada para firmar el mensaje como se especifica en la sección 3.1 de [RFC 4055](https://www.rfc-editor.org/rfc/rfc4055.txt).

- `saltLength`: {integer} - longitud de la sal para cuando el relleno es `RSA_PKCS1_PSS_PADDING`. El valor especial `crypto.constants.RSA_PSS_SALTLEN_DIGEST` establece la longitud de la sal al tamaño resumido, `crypto.constants.RSA_PSS_SALTLEN_MAX_SIGN` (por defecto) lo establece en el valor máximo permitido.

The `outputFormat` can specify one of `'latin1'`, `'hex'` or `'base64'`. If `outputFormat` is provided a string is returned; otherwise a [`Buffer`][] is returned.

El objeto `Sign` no puede ser usado nuevamente después de que el método `sign.sign()` ha sido llamado. Llamadas múltiples a `sign.sign()` van a resultar en un error.

### sign.update(data[, inputEncoding])

<!-- YAML
added: v0.1.92
changes:

  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default `inputEncoding` changed from `binary` to `utf8`.
-->

- `data` {string | Buffer | TypedArray | DataView}
- `inputEncoding` {string}

Updates the `Sign` content with the given `data`, the encoding of which is given in `inputEncoding` and can be `'utf8'`, `'ascii'` or `'latin1'`. Si `encoding` no es dado, y los `data` son una string, se aplica un código de `'utf8'`. If `data` is a [`Buffer`][], `TypedArray`, or `DataView`, then `inputEncoding` is ignored.

Esto puede ser llamado muchas veces con nuevos datos a medida en que son streamed.

## Clase: Verify

<!-- YAML
added: v0.1.92
-->

La clase `Verify` es una utilidad para verificar firmas. Puede ser usada de una de las dos maneras:

- As a writable [stream](stream.html) where written data is used to validate against the supplied signature, or
- Using the [`verify.update()`][] and [`verify.verify()`][] methods to verify the signature.

The [`crypto.createVerify()`][] method is used to create `Verify` instances. `Verify` objects are not to be created directly using the `new` keyword.

Example: Using `Verify` objects as streams:

```js
const crypto = require('crypto');
const verify = crypto.createVerify('SHA256');

verify.write('some data to sign');
verify.end();

const publicKey = getPublicKeySomehow();
const signature = getSignatureToVerify();
console.log(verify.verify(publicKey, signature));
// Prints: true or false
```

Example: Using the [`verify.update()`][] and [`verify.verify()`][] methods:

```js
const crypto = require('crypto');
const verify = crypto.createVerify('SHA256');

verify.update('some data to sign');

const publicKey = getPublicKeySomehow();
const signature = getSignatureToVerify();
console.log(verify.verify(publicKey, signature));
// Prints: true or false
```

### verify.update(data[, inputEncoding])

<!-- YAML
added: v0.1.92
changes:

  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default `inputEncoding` changed from `binary` to `utf8`.
-->

- `data` {string | Buffer | TypedArray | DataView}
- `inputEncoding` {string}

Updates the `Verify` content with the given `data`, the encoding of which is given in `inputEncoding` and can be `'utf8'`, `'ascii'` or `'latin1'`. Si `encoding` no es dado, y los `data` son una string, se aplica un código de `'utf8'`. If `data` is a [`Buffer`][], `TypedArray`, or `DataView`, then `inputEncoding` is ignored.

Esto puede ser llamado muchas veces con nuevos datos a medida en que son streamed.

### verify.verify(object, signature[, signatureFormat])

<!-- YAML
added: v0.1.92
changes:

  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11705
    description: Support for RSASSA-PSS and additional options was added.
-->

- `object` {string | Object}
- `signature` {string | Buffer | TypedArray | DataView}
- `signatureFormat` {string}

Verifies the provided data using the given `object` and `signature`. The `object` argument can be either a string containing a PEM encoded object, which can be an RSA public key, a DSA public key, or an X.509 certificate, or an object with one or more of the following properties:

- `key`: {string} - PEM encoded public key (required)
- `padding`: {integer} - Valor de padding opcional para RSA, uno de los siguientes:
  
  - `crypto.constants.RSA_PKCS1_PADDING` (por defecto)
  - `crypto.constants.RSA_PKCS1_PSS_PADDING`
  
  Note that `RSA_PKCS1_PSS_PADDING` will use MGF1 with the same hash function used to verify the message as specified in section 3.1 of [RFC 4055](https://www.rfc-editor.org/rfc/rfc4055.txt).

- `saltLength`: {integer} - longitud de la sal para cuando el relleno es `RSA_PKCS1_PSS_PADDING`. The special value `crypto.constants.RSA_PSS_SALTLEN_DIGEST` sets the salt length to the digest size, `crypto.constants.RSA_PSS_SALTLEN_AUTO` (default) causes it to be determined automatically.

The `signature` argument is the previously calculated signature for the data, in the `signatureFormat` which can be `'latin1'`, `'hex'` or `'base64'`. If a `signatureFormat` is specified, the `signature` is expected to be a string; otherwise `signature` is expected to be a [`Buffer`][], `TypedArray`, or `DataView`.

Returns `true` or `false` depending on the validity of the signature for the data and public key.

The `verify` object can not be used again after `verify.verify()` has been called. Multiple calls to `verify.verify()` will result in an error being thrown.

## `crypto` module methods and properties

### crypto.constants

<!-- YAML
added: v6.3.0
-->

Returns an object containing commonly used constants for crypto and security related operations. The specific constants currently defined are described in [Crypto Constants](#crypto_crypto_constants_1).

### crypto.DEFAULT_ENCODING

<!-- YAML
added: v0.9.3
-->

The default encoding to use for functions that can take either strings or [buffers][`Buffer`]. The default value is `'buffer'`, which makes methods default to [`Buffer`][] objects.

The `crypto.DEFAULT_ENCODING` mechanism is provided for backwards compatibility with legacy programs that expect `'latin1'` to be the default encoding.

New applications should expect the default to be `'buffer'`. This property may become deprecated in a future Node.js release.

### crypto.fips

<!-- YAML
added: v6.0.0
-->

Property for checking and controlling whether a FIPS compliant crypto provider is currently in use. Setting to true requires a FIPS build of Node.js.

### crypto.createCipher(algorithm, password[, options])

<!-- YAML
added: v0.1.94
-->

- `algorithm` {string}
- `password` {string | Buffer | TypedArray | DataView}
- `options` {Object} [`stream.transform` options][]

Creates and returns a `Cipher` object that uses the given `algorithm` and `password`. Optional `options` argument controls stream behavior.

The `algorithm` is dependent on OpenSSL, examples are `'aes192'`, etc. On recent OpenSSL releases, `openssl list-cipher-algorithms` will display the available cipher algorithms.

The `password` is used to derive the cipher key and initialization vector (IV). The value must be either a `'latin1'` encoded string, a [`Buffer`][], a `TypedArray`, or a `DataView`.

The implementation of `crypto.createCipher()` derives keys using the OpenSSL function [`EVP_BytesToKey`][] with the digest algorithm set to MD5, one iteration, and no salt. The lack of salt allows dictionary attacks as the same password always creates the same key. The low iteration count and non-cryptographically secure hash algorithm allow passwords to be tested very rapidly.

In line with OpenSSL's recommendation to use PBKDF2 instead of [`EVP_BytesToKey`][] it is recommended that developers derive a key and IV on their own using [`crypto.pbkdf2()`][] and to use [`crypto.createCipheriv()`][] to create the `Cipher` object. Users should not use ciphers with counter mode (e.g. CTR, GCM, or CCM) in `crypto.createCipher()`. A warning is emitted when they are used in order to avoid the risk of IV reuse that causes vulnerabilities. For the case when IV is reused in GCM, see [Nonce-Disrespecting Adversaries](https://github.com/nonce-disrespect/nonce-disrespect) for details.

### crypto.createCipheriv(algorithm, key, iv[, options])

- `algorithm` {string}
- `key` {string | Buffer | TypedArray | DataView}
- `iv` {string | Buffer | TypedArray | DataView}
- `options` {Object} [`stream.transform` options][]

Creates and returns a `Cipher` object, with the given `algorithm`, `key` and initialization vector (`iv`). Optional `options` argument controls stream behavior.

The `algorithm` is dependent on OpenSSL, examples are `'aes192'`, etc. On recent OpenSSL releases, `openssl list-cipher-algorithms` will display the available cipher algorithms.

The `key` is the raw key used by the `algorithm` and `iv` is an [initialization vector](https://en.wikipedia.org/wiki/Initialization_vector). Both arguments must be `'utf8'` encoded strings, [Buffers][`Buffer`], `TypedArray`, or `DataView`s.

### crypto.createCredentials(details)

<!-- YAML
added: v0.1.92
deprecated: v0.11.13
-->

> Stability: 0 - Deprecated: Use [`tls.createSecureContext()`][] instead.

- `details` {Object} Identical to [`tls.createSecureContext()`][].

The `crypto.createCredentials()` method is a deprecated function for creating and returning a `tls.SecureContext`. It should not be used. Replace it with [`tls.createSecureContext()`][] which has the exact same arguments and return value.

Returns a `tls.SecureContext`, as-if [`tls.createSecureContext()`][] had been called.

### crypto.createDecipher(algorithm, password[, options])

<!-- YAML
added: v0.1.94
-->

- `algorithm` {string}
- `password` {string | Buffer | TypedArray | DataView}
- `options` {Object} [`stream.transform` options][]

Creates and returns a `Decipher` object that uses the given `algorithm` and `password` (key). Optional `options` argument controls stream behavior.

The implementation of `crypto.createDecipher()` derives keys using the OpenSSL function [`EVP_BytesToKey`][] with the digest algorithm set to MD5, one iteration, and no salt. The lack of salt allows dictionary attacks as the same password always creates the same key. The low iteration count and non-cryptographically secure hash algorithm allow passwords to be tested very rapidly.

In line with OpenSSL's recommendation to use PBKDF2 instead of [`EVP_BytesToKey`][] it is recommended that developers derive a key and IV on their own using [`crypto.pbkdf2()`][] and to use [`crypto.createDecipheriv()`][] to create the `Decipher` object.

### crypto.createDecipheriv(algorithm, key, iv[, options])

<!-- YAML
added: v0.1.94
-->

- `algorithm` {string}
- `key` {string | Buffer | TypedArray | DataView}
- `iv` {string | Buffer | TypedArray | DataView}
- `options` {Object} [`stream.transform` options][]

Creates and returns a `Decipher` object that uses the given `algorithm`, `key` and initialization vector (`iv`). Optional `options` argument controls stream behavior.

The `algorithm` is dependent on OpenSSL, examples are `'aes192'`, etc. On recent OpenSSL releases, `openssl list-cipher-algorithms` will display the available cipher algorithms.

The `key` is the raw key used by the `algorithm` and `iv` is an [initialization vector](https://en.wikipedia.org/wiki/Initialization_vector). Both arguments must be `'utf8'` encoded strings or [buffers][`Buffer`].

### crypto.createDiffieHellman(prime\[, primeEncoding\]\[, generator\][, generatorEncoding])

<!-- YAML
added: v0.11.12
changes:

  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12223
    description: The `prime` argument can be any `TypedArray` or `DataView` now.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11983
    description: The `prime` argument can be a `Uint8Array` now.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default for the encoding parameters changed
                 from `binary` to `utf8`.
-->

- `prime` {string | Buffer | TypedArray | DataView}
- `primeEncoding` {string}
- `generator` {number | string | Buffer | TypedArray | DataView} Defaults to `2`.
- `generatorEncoding` {string}

Creates a `DiffieHellman` key exchange object using the supplied `prime` and an optional specific `generator`.

The `generator` argument can be a number, string, or [`Buffer`][]. If `generator` is not specified, the value `2` is used.

The `primeEncoding` and `generatorEncoding` arguments can be `'latin1'`, `'hex'`, or `'base64'`.

If `primeEncoding` is specified, `prime` is expected to be a string; otherwise a [`Buffer`][], `TypedArray`, or `DataView` is expected.

If `generatorEncoding` is specified, `generator` is expected to be a string; otherwise a number, [`Buffer`][], `TypedArray`, or `DataView` is expected.

### crypto.createDiffieHellman(primeLength[, generator])

<!-- YAML
added: v0.5.0
-->

- `primeLength` {number}
- `generator` {number | string | Buffer | TypedArray | DataView} Defaults to `2`.

Creates a `DiffieHellman` key exchange object and generates a prime of `primeLength` bits using an optional specific numeric `generator`. If `generator` is not specified, the value `2` is used.

### crypto.createECDH(curveName)

<!-- YAML
added: v0.11.14
-->

- `curveName` {string}

Creates an Elliptic Curve Diffie-Hellman (`ECDH`) key exchange object using a predefined curve specified by the `curveName` string. Use [`crypto.getCurves()`][] to obtain a list of available curve names. On recent OpenSSL releases, `openssl ecparam -list_curves` will also display the name and description of each available elliptic curve.

### crypto.createHash(algorithm[, options])

<!-- YAML
added: v0.1.92
-->

- `algorithm` {string}
- `options` {Object} [`stream.transform` options][]

Creates and returns a `Hash` object that can be used to generate hash digests using the given `algorithm`. Optional `options` argument controls stream behavior.

The `algorithm` is dependent on the available algorithms supported by the version of OpenSSL on the platform. Examples are `'sha256'`, `'sha512'`, etc. On recent releases of OpenSSL, `openssl list-message-digest-algorithms` will display the available digest algorithms.

Example: generating the sha256 sum of a file

```js
const filename = process.argv[2];
const crypto = require('crypto');
const fs = require('fs');

const hash = crypto.createHash('sha256');

const input = fs.createReadStream(filename);
input.on('readable', () => {
  const data = input.read();
  if (data)
    hash.update(data);
  else {
    console.log(`${hash.digest('hex')} ${filename}`);
  }
});
```

### crypto.createHmac(algorithm, key[, options])

<!-- YAML
added: v0.1.94
-->

- `algorithm` {string}
- `key` {string | Buffer | TypedArray | DataView}
- `options` {Object} [`stream.transform` options][]

Creates and returns an `Hmac` object that uses the given `algorithm` and `key`. Optional `options` argument controls stream behavior.

The `algorithm` is dependent on the available algorithms supported by the version of OpenSSL on the platform. Examples are `'sha256'`, `'sha512'`, etc. On recent releases of OpenSSL, `openssl list-message-digest-algorithms` will display the available digest algorithms.

The `key` is the HMAC key used to generate the cryptographic HMAC hash.

Example: generating the sha256 HMAC of a file

```js
const filename = process.argv[2];
const crypto = require('crypto');
const fs = require('fs');

const hmac = crypto.createHmac('sha256', 'a secret');

const input = fs.createReadStream(filename);
input.on('readable', () => {
  const data = input.read();
  if (data)
    hmac.update(data);
  else {
    console.log(`${hmac.digest('hex')} ${filename}`);
  }
});
```

### crypto.createSign(algorithm[, options])

<!-- YAML
added: v0.1.92
-->

- `algorithm` {string}
- `options` {Object} [`stream.Writable` options][]

Creates and returns a `Sign` object that uses the given `algorithm`. Use [`crypto.getHashes()`][] to obtain an array of names of the available signing algorithms. Optional `options` argument controls the `stream.Writable` behavior.

### crypto.createVerify(algorithm[, options])

<!-- YAML
added: v0.1.92
-->

- `algorithm` {string}
- `options` {Object} [`stream.Writable` options][]

Creates and returns a `Verify` object that uses the given algorithm. Use [`crypto.getHashes()`][] to obtain an array of names of the available signing algorithms. Optional `options` argument controls the `stream.Writable` behavior.

### crypto.getCiphers()

<!-- YAML
added: v0.9.3
-->

Returns an array with the names of the supported cipher algorithms.

Ejemplo:

```js
const ciphers = crypto.getCiphers();
console.log(ciphers); // ['aes-128-cbc', 'aes-128-ccm', ...]
```

### crypto.getCurves()

<!-- YAML
added: v2.3.0
-->

Returns an array with the names of the supported elliptic curves.

Ejemplo:

```js
const curves = crypto.getCurves();
console.log(curves); // ['Oakley-EC2N-3', 'Oakley-EC2N-4', ...]
```

### crypto.getDiffieHellman(groupName)

<!-- YAML
added: v0.7.5
-->

- `groupName` {string}

Creates a predefined `DiffieHellman` key exchange object. The supported groups are: `'modp1'`, `'modp2'`, `'modp5'` (defined in [RFC 2412](https://www.rfc-editor.org/rfc/rfc2412.txt), but see [Caveats](#crypto_support_for_weak_or_compromised_algorithms)) and `'modp14'`, `'modp15'`, `'modp16'`, `'modp17'`, `'modp18'` (defined in [RFC 3526](https://www.rfc-editor.org/rfc/rfc3526.txt)). The returned object mimics the interface of objects created by [`crypto.createDiffieHellman()`][], but will not allow changing the keys (with [`diffieHellman.setPublicKey()`][] for example). The advantage of using this method is that the parties do not have to generate nor exchange a group modulus beforehand, saving both processor and communication time.

Ejemplo (obteniendo un secreto compartido):

```js
const crypto = require('crypto');
const alice = crypto.getDiffieHellman('modp14');
const bob = crypto.getDiffieHellman('modp14');

alice.generateKeys();
bob.generateKeys();

const aliceSecret = alice.computeSecret(bob.getPublicKey(), null, 'hex');
const bobSecret = bob.computeSecret(alice.getPublicKey(), null, 'hex');

/* aliceSecret and bobSecret should be the same */
console.log(aliceSecret === bobSecret);
```

### crypto.getHashes()

<!-- YAML
added: v0.9.3
-->

Returns an array of the names of the supported hash algorithms, such as `RSA-SHA256`.

Ejemplo:

```js
const hashes = crypto.getHashes();
console.log(hashes); // ['DSA', 'DSA-SHA', 'DSA-SHA1', ...]
```

### crypto.pbkdf2(password, salt, iterations, keylen, digest, callback)

<!-- YAML
added: v0.5.5
changes:

  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11305
    description: The `digest` parameter is always required now.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/4047
    description: Calling this function without passing the `digest` parameter
                 is deprecated now and will emit a warning.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default encoding for `password` if it is a string changed
                 from `binary` to `utf8`.
-->

- `password` {string}
- `salt` {string}
- `iterations` {number}
- `keylen` {number}
- `digest` {string}
- `callback` {Function} 
  - `err` {Error}
  - `derivedKey` {Buffer}

Provides an asynchronous Password-Based Key Derivation Function 2 (PBKDF2) implementation. A selected HMAC digest algorithm specified by `digest` is applied to derive a key of the requested byte length (`keylen`) from the `password`, `salt` and `iterations`.

The supplied `callback` function is called with two arguments: `err` and `derivedKey`. If an error occurs, `err` will be set; otherwise `err` will be null. The successfully generated `derivedKey` will be passed as a [`Buffer`][].

The `iterations` argument must be a number set as high as possible. The higher the number of iterations, the more secure the derived key will be, but will take a longer amount of time to complete.

The `salt` should also be as unique as possible. It is recommended that the salts are random and their lengths are at least 16 bytes. See [NIST SP 800-132](http://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-132.pdf) for details.

Ejemplo:

```js
const crypto = require('crypto');
crypto.pbkdf2('secret', 'salt', 100000, 64, 'sha512', (err, derivedKey) => {
  if (err) throw err;
  console.log(derivedKey.toString('hex'));  // '3745e48...08d59ae'
});
```

An array of supported digest functions can be retrieved using [`crypto.getHashes()`][].

Note that this API uses libuv's threadpool, which can have surprising and negative performance implications for some applications, see the [`UV_THREADPOOL_SIZE`][] documentation for more information.

### crypto.pbkdf2Sync(password, salt, iterations, keylen, digest)

<!-- YAML
added: v0.9.3
changes:

  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/4047
    description: Calling this function without passing the `digest` parameter
                 is deprecated now and will emit a warning.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default encoding for `password` if it is a string changed
                 from `binary` to `utf8`.
-->

- `password` {string}
- `salt` {string}
- `iterations` {number}
- `keylen` {number}
- `digest` {string}

Provides a synchronous Password-Based Key Derivation Function 2 (PBKDF2) implementation. A selected HMAC digest algorithm specified by `digest` is applied to derive a key of the requested byte length (`keylen`) from the `password`, `salt` and `iterations`.

If an error occurs an Error will be thrown, otherwise the derived key will be returned as a [`Buffer`][].

The `iterations` argument must be a number set as high as possible. The higher the number of iterations, the more secure the derived key will be, but will take a longer amount of time to complete.

The `salt` should also be as unique as possible. It is recommended that the salts are random and their lengths are at least 16 bytes. See [NIST SP 800-132](http://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-132.pdf) for details.

Ejemplo:

```js
const crypto = require('crypto');
const key = crypto.pbkdf2Sync('secret', 'salt', 100000, 64, 'sha512');
console.log(key.toString('hex'));  // '3745e48...08d59ae'
```

An array of supported digest functions can be retrieved using [`crypto.getHashes()`][].

### crypto.privateDecrypt(privateKey, buffer)

<!-- YAML
added: v0.11.14
-->

- `privateKey` {Object | string} 
  - `key` {string} A PEM encoded private key.
  - `passphrase` {string} An optional passphrase for the private key.
  - `padding` {crypto.constants} An optional padding value defined in `crypto.constants`, which may be: `crypto.constants.RSA_NO_PADDING`, `RSA_PKCS1_PADDING`, or `crypto.constants.RSA_PKCS1_OAEP_PADDING`.
- `buffer` {Buffer | TypedArray | DataView}
- Returns: {Buffer} A new `Buffer` with the decrypted content.

Decrypts `buffer` with `privateKey`.

`privateKey` can be an object or a string. If `privateKey` is a string, it is treated as the key with no passphrase and will use `RSA_PKCS1_OAEP_PADDING`.

### crypto.privateEncrypt(privateKey, buffer)

<!-- YAML
added: v1.1.0
-->

- `privateKey` {Object | string} 
  - `key` {string} A PEM encoded private key.
  - `passphrase` {string} An optional passphrase for the private key.
  - `padding` {crypto.constants} An optional padding value defined in `crypto.constants`, which may be: `crypto.constants.RSA_NO_PADDING` or `RSA_PKCS1_PADDING`.
- `buffer` {Buffer | TypedArray | DataView}
- Returns: {Buffer} A new `Buffer` with the encrypted content.

Encrypts `buffer` with `privateKey`.

`privateKey` can be an object or a string. If `privateKey` is a string, it is treated as the key with no passphrase and will use `RSA_PKCS1_PADDING`.

### crypto.publicDecrypt(key, buffer)

<!-- YAML
added: v1.1.0
-->

- `key` {Object | string} 
  - `key` {string} A PEM encoded public or private key.
  - `passphrase` {string} An optional passphrase for the private key.
  - `padding` {crypto.constants} An optional padding value defined in `crypto.constants`, which may be: `crypto.constants.RSA_NO_PADDING` or `RSA_PKCS1_PADDING`.
- `buffer` {Buffer | TypedArray | DataView}
- Returns: {Buffer} A new `Buffer` with the decrypted content.

Decrypts `buffer` with `key`.

`key` can be an object or a string. If `key` is a string, it is treated as the key with no passphrase and will use `RSA_PKCS1_PADDING`.

Because RSA public keys can be derived from private keys, a private key may be passed instead of a public key.

### crypto.publicEncrypt(key, buffer)

<!-- YAML
added: v0.11.14
-->

- `key` {Object | string} 
  - `key` {string} A PEM encoded public or private key.
  - `passphrase` {string} An optional passphrase for the private key.
  - `padding` {crypto.constants} An optional padding value defined in `crypto.constants`, which may be: `crypto.constants.RSA_NO_PADDING`, `RSA_PKCS1_PADDING`, or `crypto.constants.RSA_PKCS1_OAEP_PADDING`.
- `buffer` {Buffer | TypedArray | DataView}
- Returns: {Buffer} A new `Buffer` with the encrypted content.

Encrypts the content of `buffer` with `key` and returns a new [`Buffer`][] with encrypted content.

`key` can be an object or a string. If `key` is a string, it is treated as the key with no passphrase and will use `RSA_PKCS1_OAEP_PADDING`.

Because RSA public keys can be derived from private keys, a private key may be passed instead of a public key.

### crypto.randomBytes(size[, callback])

<!-- YAML
added: v0.5.8
-->

- `size` {number}
- `callback` {Function} 
  - `err` {Error}
  - `buf` {Buffer}

Generates cryptographically strong pseudo-random data. The `size` argument is a number indicating the number of bytes to generate.

If a `callback` function is provided, the bytes are generated asynchronously and the `callback` function is invoked with two arguments: `err` and `buf`. If an error occurs, `err` will be an Error object; otherwise it is null. The `buf` argument is a [`Buffer`][] containing the generated bytes.

```js
// Asynchronous
const crypto = require('crypto');
crypto.randomBytes(256, (err, buf) => {
  if (err) throw err;
  console.log(`${buf.length} bytes of random data: ${buf.toString('hex')}`);
});
```

If the `callback` function is not provided, the random bytes are generated synchronously and returned as a [`Buffer`][]. An error will be thrown if there is a problem generating the bytes.

```js
// Synchronous
const buf = crypto.randomBytes(256);
console.log(
  `${buf.length} bytes of random data: ${buf.toString('hex')}`);
```

The `crypto.randomBytes()` method will not complete until there is sufficient entropy available. This should normally never take longer than a few milliseconds. The only time when generating the random bytes may conceivably block for a longer period of time is right after boot, when the whole system is still low on entropy.

Note that this API uses libuv's threadpool, which can have surprising and negative performance implications for some applications, see the [`UV_THREADPOOL_SIZE`][] documentation for more information.

*Note*: The asynchronous version of `crypto.randomBytes()` is carried out in a single threadpool request. To minimize threadpool task length variation, partition large `randomBytes` requests when doing so as part of fulfilling a client request.

### crypto.randomFillSync(buffer\[, offset\]\[, size\])

<!-- YAML
added: v7.10.0
-->

- `buffer` {Buffer|Uint8Array} Must be supplied.
- `offset` {number} Defaults to `0`.
- `size` {number} Defaults to `buffer.length - offset`.

Synchronous version of [`crypto.randomFill()`][].

Returns `buffer`

```js
const buf = Buffer.alloc(10);
console.log(crypto.randomFillSync(buf).toString('hex'));

crypto.randomFillSync(buf, 5);
console.log(buf.toString('hex'));

// The above is equivalent to the following:
crypto.randomFillSync(buf, 5, 5);
console.log(buf.toString('hex'));
```

### crypto.randomFill(buffer\[, offset\]\[, size\], callback)

<!-- YAML
added: v7.10.0
-->

- `buffer` {Buffer|Uint8Array} Must be supplied.
- `offset` {number} Defaults to `0`.
- `size` {number} Defaults to `buffer.length - offset`.
- `callback` {Function} `function(err, buf) {}`.

This function is similar to [`crypto.randomBytes()`][] but requires the first argument to be a [`Buffer`][] that will be filled. It also requires that a callback is passed in.

If the `callback` function is not provided, an error will be thrown.

```js
const buf = Buffer.alloc(10);
crypto.randomFill(buf, (err, buf) => {
  if (err) throw err;
  console.log(buf.toString('hex'));
});

crypto.randomFill(buf, 5, (err, buf) => {
  if (err) throw err;
  console.log(buf.toString('hex'));
});

// The above is equivalent to the following:
crypto.randomFill(buf, 5, 5, (err, buf) => {
  if (err) throw err;
  console.log(buf.toString('hex'));
});
```

Note that this API uses libuv's threadpool, which can have surprising and negative performance implications for some applications, see the [`UV_THREADPOOL_SIZE`][] documentation for more information.

*Note*: The asynchronous version of `crypto.randomFill()` is carried out in a single threadpool request. To minimize threadpool task length variation, partition large `randomFill` requests when doing so as part of fulfilling a client request.

### crypto.setEngine(engine[, flags])

<!-- YAML
added: v0.11.11
-->

- `engine` {string}
- `flags` {crypto.constants} Defaults to `crypto.constants.ENGINE_METHOD_ALL`.

Load and set the `engine` for some or all OpenSSL functions (selected by flags).

`engine` could be either an id or a path to the engine's shared library.

The optional `flags` argument uses `ENGINE_METHOD_ALL` by default. The `flags` is a bit field taking one of or a mix of the following flags (defined in `crypto.constants`):

- `crypto.constants.ENGINE_METHOD_RSA`
- `crypto.constants.ENGINE_METHOD_DSA`
- `crypto.constants.ENGINE_METHOD_DH`
- `crypto.constants.ENGINE_METHOD_RAND`
- `crypto.constants.ENGINE_METHOD_ECDH`
- `crypto.constants.ENGINE_METHOD_ECDSA`
- `crypto.constants.ENGINE_METHOD_CIPHERS`
- `crypto.constants.ENGINE_METHOD_DIGESTS`
- `crypto.constants.ENGINE_METHOD_STORE`
- `crypto.constants.ENGINE_METHOD_PKEY_METHS`
- `crypto.constants.ENGINE_METHOD_PKEY_ASN1_METHS`
- `crypto.constants.ENGINE_METHOD_ALL`
- `crypto.constants.ENGINE_METHOD_NONE`

### crypto.timingSafeEqual(a, b)

<!-- YAML
added: v6.6.0
-->

- `a` {Buffer | TypedArray | DataView}
- `b` {Buffer | TypedArray | DataView}

This function is based on a constant-time algorithm. Returns true if `a` is equal to `b`, without leaking timing information that would allow an attacker to guess one of the values. This is suitable for comparing HMAC digests or secret values like authentication cookies or [capability urls](https://www.w3.org/TR/capability-urls/).

`a` and `b` must both be `Buffer`s, `TypedArray`s, or `DataView`s, and they must have the same length.

*Note*: Use of `crypto.timingSafeEqual` does not guarantee that the *surrounding* code is timing-safe. Care should be taken to ensure that the surrounding code does not introduce timing vulnerabilities.

## Notes

### Legacy Streams API (pre Node.js v0.10)

The Crypto module was added to Node.js before there was the concept of a unified Stream API, and before there were [`Buffer`][] objects for handling binary data. As such, the many of the `crypto` defined classes have methods not typically found on other Node.js classes that implement the [streams](stream.html) API (e.g. `update()`, `final()`, or `digest()`). Also, many methods accepted and returned `'latin1'` encoded strings by default rather than Buffers. This default was changed after Node.js v0.8 to use [`Buffer`][] objects by default instead.

### Recent ECDH Changes

Usage of `ECDH` with non-dynamically generated key pairs has been simplified. Now, [`ecdh.setPrivateKey()`][] can be called with a preselected private key and the associated public point (key) will be computed and stored in the object. This allows code to only store and provide the private part of the EC key pair. [`ecdh.setPrivateKey()`][] now also validates that the private key is valid for the selected curve.

The [`ecdh.setPublicKey()`][] method is now deprecated as its inclusion in the API is not useful. Either a previously stored private key should be set, which automatically generates the associated public key, or [`ecdh.generateKeys()`][] should be called. The main drawback of using [`ecdh.setPublicKey()`][] is that it can be used to put the ECDH key pair into an inconsistent state.

### Support for weak or compromised algorithms

The `crypto` module still supports some algorithms which are already compromised and are not currently recommended for use. The API also allows the use of ciphers and hashes with a small key size that are considered to be too weak for safe use.

Users should take full responsibility for selecting the crypto algorithm and key size according to their security requirements.

Based on the recommendations of [NIST SP 800-131A](http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-131Ar1.pdf):

- MD5 and SHA-1 are no longer acceptable where collision resistance is required such as digital signatures.
- The key used with RSA, DSA, and DH algorithms is recommended to have at least 2048 bits and that of the curve of ECDSA and ECDH at least 224 bits, to be safe to use for several years.
- The DH groups of `modp1`, `modp2` and `modp5` have a key size smaller than 2048 bits and are not recommended.

See the reference for other recommendations and details.

## Crypto Constants

The following constants exported by `crypto.constants` apply to various uses of the `crypto`, `tls`, and `https` modules and are generally specific to OpenSSL.

### OpenSSL Options

<table>
  <tr>
    <th>Constant</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>SSL_OP_ALL</code></td>
    <td>Applies multiple bug workarounds within OpenSSL. See
    https://www.openssl.org/docs/man1.0.2/ssl/SSL_CTX_set_options.html for
    detail.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_ALLOW_UNSAFE_LEGACY_RENEGOTIATION</code></td>
    <td>Allows legacy insecure renegotiation between OpenSSL and unpatched
    clients or servers. See
    https://www.openssl.org/docs/man1.0.2/ssl/SSL_CTX_set_options.html.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_CIPHER_SERVER_PREFERENCE</code></td>
    <td>Attempts to use the server's preferences instead of the client's when
    selecting a cipher. Behavior depends on protocol version. See
    https://www.openssl.org/docs/man1.0.2/ssl/SSL_CTX_set_options.html.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_CISCO_ANYCONNECT</code></td>
    <td>Instructs OpenSSL to use Cisco's "speshul" version of DTLS_BAD_VER.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_COOKIE_EXCHANGE</code></td>
    <td>Instructs OpenSSL to turn on cookie exchange.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_CRYPTOPRO_TLSEXT_BUG</code></td>
    <td>Instructs OpenSSL to add server-hello extension from an early version
    of the cryptopro draft.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_DONT_INSERT_EMPTY_FRAGMENTS</code></td>
    <td>Instructs OpenSSL to disable a SSL 3.0/TLS 1.0 vulnerability
    workaround added in OpenSSL 0.9.6d.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_EPHEMERAL_RSA</code></td>
    <td>Instructs OpenSSL to always use the tmp_rsa key when performing RSA
    operations.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_LEGACY_SERVER_CONNECT</code></td>
    <td>Allows initial connection to servers that do not support RI.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_MICROSOFT_BIG_SSLV3_BUFFER</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_MICROSOFT_SESS_ID_BUG</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_MSIE_SSLV2_RSA_PADDING</code></td>
    <td>Instructs OpenSSL to disable the workaround for a man-in-the-middle
    protocol-version vulnerability in the SSL 2.0 server implementation.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NETSCAPE_CA_DN_BUG</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_NETSCAPE_CHALLENGE_BUG</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_NETSCAPE_DEMO_CIPHER_CHANGE_BUG</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_NETSCAPE_REUSE_CIPHER_CHANGE_BUG</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_COMPRESSION</code></td>
    <td>Instructs OpenSSL to disable support for SSL/TLS compression.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_QUERY_MTU</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_SESSION_RESUMPTION_ON_RENEGOTIATION</code></td>
    <td>Instructs OpenSSL to always start a new session when performing
    renegotiation.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_SSLv2</code></td>
    <td>Instructs OpenSSL to turn off SSL v2</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_SSLv3</code></td>
    <td>Instructs OpenSSL to turn off SSL v3</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_TICKET</code></td>
    <td>Instructs OpenSSL to disable use of RFC4507bis tickets.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_TLSv1</code></td>
    <td>Instructs OpenSSL to turn off TLS v1</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_TLSv1_1</code></td>
    <td>Instructs OpenSSL to turn off TLS v1.1</td>
  </tr>
  <tr>
    <td><code>SSL_OP_NO_TLSv1_2</code></td>
    <td>Instructs OpenSSL to turn off TLS v1.2</td>
  </tr>
    <td><code>SSL_OP_PKCS1_CHECK_1</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_PKCS1_CHECK_2</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_SINGLE_DH_USE</code></td>
    <td>Instructs OpenSSL to always create a new key when using
    temporary/ephemeral DH parameters.</td>
  </tr>
  <tr>
    <td><code>SSL_OP_SINGLE_ECDH_USE</code></td>
    <td>Instructs OpenSSL to always create a new key when using
    temporary/ephemeral ECDH parameters.</td>
  </tr>
    <td><code>SSL_OP_SSLEAY_080_CLIENT_DH_BUG</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_SSLREF2_REUSE_CERT_TYPE_BUG</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_TLS_BLOCK_PADDING_BUG</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_TLS_D5_BUG</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>SSL_OP_TLS_ROLLBACK_BUG</code></td>
    <td>Instructs OpenSSL to disable version rollback attack detection.</td>
  </tr>
</table>

### OpenSSL Engine Constants

<table>
  <tr>
    <th>Constant</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_RSA</code></td>
    <td>Limit engine usage to RSA</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_DSA</code></td>
    <td>Limit engine usage to DSA</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_DH</code></td>
    <td>Limit engine usage to DH</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_RAND</code></td>
    <td>Limit engine usage to RAND</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_ECDH</code></td>
    <td>Limit engine usage to ECDH</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_ECDSA</code></td>
    <td>Limit engine usage to ECDSA</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_CIPHERS</code></td>
    <td>Limit engine usage to CIPHERS</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_DIGESTS</code></td>
    <td>Limit engine usage to DIGESTS</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_STORE</code></td>
    <td>Limit engine usage to STORE</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_PKEY_METHS</code></td>
    <td>Limit engine usage to PKEY_METHDS</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_PKEY_ASN1_METHS</code></td>
    <td>Limit engine usage to PKEY_ASN1_METHS</td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_ALL</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>ENGINE_METHOD_NONE</code></td>
    <td></td>
  </tr>
</table>

### Other OpenSSL Constants

<table>
  <tr>
    <th>Constant</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>DH_CHECK_P_NOT_SAFE_PRIME</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>DH_CHECK_P_NOT_PRIME</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>DH_UNABLE_TO_CHECK_GENERATOR</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>DH_NOT_SUITABLE_GENERATOR</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>NPN_ENABLED</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>ALPN_ENABLED</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>RSA_PKCS1_PADDING</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>RSA_SSLV23_PADDING</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>RSA_NO_PADDING</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>RSA_PKCS1_OAEP_PADDING</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>RSA_X931_PADDING</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>RSA_PKCS1_PSS_PADDING</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>RSA_PSS_SALTLEN_DIGEST</code></td>
    <td>Sets the salt length for `RSA_PKCS1_PSS_PADDING` to the digest size
        when signing or verifying.</td>
  </tr>
  <tr>
    <td><code>RSA_PSS_SALTLEN_MAX_SIGN</code></td>
    <td>Sets the salt length for `RSA_PKCS1_PSS_PADDING` to the maximum
        permissible value when signing data.</td>
  </tr>
  <tr>
    <td><code>RSA_PSS_SALTLEN_AUTO</code></td>
    <td>Causes the salt length for `RSA_PKCS1_PSS_PADDING` to be determined
        automatically when verifying a signature.</td>
  </tr>
  <tr>
    <td><code>POINT_CONVERSION_COMPRESSED</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>POINT_CONVERSION_UNCOMPRESSED</code></td>
    <td></td>
  </tr>
  <tr>
    <td><code>POINT_CONVERSION_HYBRID</code></td>
    <td></td>
  </tr>
</table>

### Node.js Crypto Constants

<table>
  <tr>
    <th>Constant</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>defaultCoreCipherList</code></td>
    <td>Specifies the built-in default cipher list used by Node.js.</td>
  </tr>
  <tr>
    <td><code>defaultCipherList</code></td>
    <td>Specifies the active default cipher list used by the current Node.js
    process.</td>
  </tr>
</table>
