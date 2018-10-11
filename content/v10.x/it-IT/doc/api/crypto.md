# Crypto

<!--introduced_in=v0.3.6-->

> Stabilità: 2 - Stabile

Il modulo `crypto` fornisce una funzionalità crittografica che include un set di wrapper per le funzioni OpenSSL di hash, HMAC, cipher, decipher, sign e verify.

Utilizza `require('crypto')` per accedere a questo modulo.

```js
const crypto = require('crypto');

const secret = 'abcdefg';
const hash = crypto.createHmac('sha256', secret)
                   .update('I love cupcakes')
                   .digest('hex');
console.log(hash);
// Stampa:
//   c0fa1bc00531bd78ef38c628449c5102aeabd49b5dc3a2a516ea6ea959d6658e
```

## Determinare se il supporto crittografico non è disponibile

È possibile creare Node.js senza includere il supporto per il modulo `crypto`. In questi casi, la chiamata di `require('crypto')` genererà un errore.

```js
let crypto;
try {
  crypto = require('crypto');
} catch (err) {
  console.log('crypto support is disabled!');
}
```

## Class: Certificate

<!-- YAML
added: v0.11.8
-->

SPKAC è un meccanismo di Certificate Signing Request (CSR) originariamente implementato da Netscape ed è stato specificato formalmente come parte del [HTML5's `keygen` element][].

Da notare che `<keygen>` è obsoleto/deprecato poiché [HTML 5.2](https://www.w3.org/TR/html52/changes.html#features-removed) e i nuovi progetti non dovrebbero più utilizzare questo elemento.

Il modulo `crypto` fornisce la classe `Certificate` per lavorare con i dati SPKAC. L'utilizzo più comune è la gestione dell'output generato dall'elemento HTML5 `<keygen>`. Node.js utilizza [l'implementazione SPKAC di OpenSSL](https://www.openssl.org/docs/man1.1.0/apps/openssl-spkac.html) internamente.

### Certificate.exportChallenge(spkac)

<!-- YAML
added: v9.0.0
-->

- `spkac` {string | Buffer | TypedArray | DataView}
- Restituisce: {Buffer} La componente challenge della struttura dati di `spkac`, che include una public key ed una challenge.

```js
const { Certificate } = require('crypto');
const spkac = getSpkacSomehow();
const challenge = Certificate.exportChallenge(spkac);
console.log(challenge.toString('utf8'));
// Stampa: la challenge come una stringa UTF8
```

### Certificate.exportPublicKey(spkac[, encoding])

<!-- YAML
added: v9.0.0
-->

- `spkac` {string | Buffer | TypedArray | DataView}
- `encoding` {string}
- Restituisce: {Buffer} La componente public key della struttura dati di `spkac`, che include una public key ed una challenge.

```js
const { Certificate } = require('crypto');
const spkac = getSpkacSomehow();
const publicKey = Certificate.exportPublicKey(spkac);
console.log(publicKey);
// Stampa: la public key come un <Buffer ...>
```

### Certificate.verifySpkac(spkac)

<!-- YAML
added: v9.0.0
-->

- `spkac` {Buffer | TypedArray | DataView}
- Restituisce: {boolean} `true` se la struttura dati di `spkac` fornita è valida, in caso contrario `false`.

```js
const { Certificate } = require('crypto');
const spkac = getSpkacSomehow();
console.log(Certificate.verifySpkac(Buffer.from(spkac)));
// Stampa: true oppure false
```

### Legacy API

Come un'interfaccia legacy ancora supportata, è possibile (ma non consigliato) creare nuove istanze della classe `crypto.Certificate` come mostrato nei seguenti esempi.

#### new crypto.Certificate()

Le istanze della classe `Certificate` possono essere create utilizzando la parola chiave `new` o chiamando `crypto.Certificate()` come funzione:

```js
const crypto = require('crypto');

const cert1 = new crypto.Certificate();
const cert2 = crypto.Certificate();
```

#### certificate.exportChallenge(spkac)

<!-- YAML
added: v0.11.8
-->

- `spkac` {string | Buffer | TypedArray | DataView}
- Restituisce: {Buffer} La componente challenge della struttura dati di `spkac`, che include una public key ed una challenge.

```js
const cert = require('crypto').Certificate();
const spkac = getSpkacSomehow();
const challenge = cert.exportChallenge(spkac);
console.log(challenge.toString('utf8'));
// Stampa: la challenge come una stringa UTF8
```

#### certificate.exportPublicKey(spkac)

<!-- YAML
added: v0.11.8
-->

- `spkac` {string | Buffer | TypedArray | DataView}
- Restituisce: {Buffer} La componente public key della struttura dati di `spkac`, che include una public key ed una challenge.

```js
const cert = require('crypto').Certificate();
const spkac = getSpkacSomehow();
const publicKey = cert.exportPublicKey(spkac);
console.log(publicKey);
// Stampa: la public key come un <Buffer ...>
```

#### certificate.verifySpkac(spkac)

<!-- YAML
added: v0.11.8
-->

- `spkac` {Buffer | TypedArray | DataView}
- Restituisce: {boolean} `true` se la struttura dati di `spkac` fornita è valida, in caso contrario `false`.

```js
const cert = require('crypto').Certificate();
const spkac = getSpkacSomehow();
console.log(cert.verifySpkac(Buffer.from(spkac)));
// Stampa: true oppure false
```

## Class: Cipher

<!-- YAML
added: v0.1.94
-->

Le istanze della classe `Cipher` vengono utilizzate per crittografare i dati. La classe può essere utilizzata in due modi:

- Come uno [stream](stream.html) che è sia readable che writable (leggibile e scrivibile), sul quale vengono scritti tramite il writing semplici dati non criptati per produrre i dati criptati sul lato readable, oppure
- Utilizzando i metodi [`cipher.update()`][] e [`cipher.final()`][] per produrre i dati criptati.

I metodi [`crypto.createCipher()`][] o [`crypto.createCipheriv()`][] vengono usati per creare istanze `Cipher`. Gli `Cipher` object non devono essere creati direttamente utilizzando la parola chiave `new`.

Esempio: Utilizzando gli `Cipher` object come degli stream:

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
  // Stampa: ca981be48e90867604588e75d04feabb63cc007a8f8ad89b10616ed84d815504
});

cipher.write('some clear text data');
cipher.end();
```

Esempio: Utilizzando `Cipher` e i piped stream:

```js
const crypto = require('crypto');
const fs = require('fs');
const cipher = crypto.createCipher('aes192', 'a password');

const input = fs.createReadStream('test.js');
const output = fs.createWriteStream('test.enc');

input.pipe(cipher).pipe(output);
```

Esempio: Utilizzando i metodi [`cipher.update()`][] e [`cipher.final()`][]:

```js
const crypto = require('crypto');
const cipher = crypto.createCipher('aes192', 'a password');

let encrypted = cipher.update('some clear text data', 'utf8', 'hex');
encrypted += cipher.final('hex');
console.log(encrypted);
// Stampa: ca981be48e90867604588e75d04feabb63cc007a8f8ad89b10616ed84d815504
```

### cipher.final([outputEncoding])

<!-- YAML
added: v0.1.94
-->

- `outputEncoding` {string}
- Restituisce: {Buffer | string} Qualsiasi contenuto cifrato restante. Se l'`outputEncoding` è `'latin1'`, `'base64'` o `'hex'`, viene restituita una stringa. Se non viene fornito nessun `outputEncoding`, viene restituito un [`Buffer`][].

Una volta chiamato il metodo `cipher.final()`, il `Cipher` object non può più essere utilizzato per cifrare i dati. I tentativi di chiamare `cipher.final()` più di una volta genereranno un errore.

### cipher.setAAD(buffer[, options])

<!-- YAML
added: v1.0.0
-->

- `buffer` {Buffer}
- `options` {Object}
- Restituisce: {Cipher} per il method chaining.

Quando si utilizza una modalità di crittografia autenticata (attualmente sono supportate solo la `GCM` e la `CCM`), il metodo `cipher.setAAD()` imposta il valore utilizzato per il parametro input *additional authenticated data* (AAD).

L'argomento `options` è facoltativo per la `GCM`. Quando si utilizza la `CCM`, deve essere specificata l'opzione `plaintextLength` e il suo valore deve corrispondere alla lunghezza del testo normale in byte. Vedi [Modalità CCM](#crypto_ccm_mode).

Il metodo `cipher.setAAD()` dev'essere chiamato prima di [`cipher.update()`][].

### cipher.getAuthTag()

<!-- YAML
added: v1.0.0
-->

- Restituisce: {Buffer} Quando si utilizza una modalità di crittografia autenticata (attualmente sono supportate solo la `GCM` e la `CCM`), il metodo `cipher.getAuthTag()` restituisce un [`Buffer`][] contenente l'*authentication tag* che è stato calcolato dai dati forniti.

Il metodo `cipher.getAuthTag()` dev'essere chiamato solo dopo aver completato la crittografia utilizzando il metodo [`cipher.final()`][].

### cipher.setAutoPadding([autoPadding])

<!-- YAML
added: v0.7.1
-->

- `autoPadding` {boolean} **Default:** `true`
- Restituisce: {Cipher} per il method chaining.

Quando si utilizzano algoritmi di cifratura a blocchi, la classe `Cipher` eseguirà automaticamente il padding (riempimento) dei dati d'input fino a raggiungere la block size appropriata. Per disattivare il padding predefinito chiamare `cipher.setAutoPadding(false)`.

Quando `autoPadding` è `false`, la lunghezza di tutti i dati d'input dev'essere un multiplo della block size del cipher altrimenti [`cipher.final()`][] genererà un errore. La disattivazione del padding automatico è utile per il padding non standard, ad esempio utilizzando `0x0` al posto del padding PKCS.

Il metodo `cipher.setAutoPadding()` dev'essere chiamato prima di [`cipher.final()`][].

### cipher.update(data\[, inputEncoding\]\[, outputEncoding\])

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
- Restituisce: {Buffer | string}

Aggiorna il cipher con `data`. Se viene specificato l'argomento `inputEncoding`, il suo valore dev'essere `'utf8'`, `'ascii'`, oppure `'latin1'` e l'argomento `data` è una stringa che utilizza l'encoding specificato. Se l'argomento `inputEncoding` non è specificato, `data` dev'essere un [`Buffer`][], un `TypedArray`, oppure un `DataView`. Se `data` è un [`Buffer`][], un `TypedArray`, oppure un `DataView`, allora `inputEncoding` viene ignorato.

L'`outputEncoding` specifica il formato di output dei dati cifrati e può essere `'latin1'`, `'base64'` o `'hex'`. Se l'`outputEncoding` è specificato, viene restituita una stringa che utilizza l'encoding specificato. Se non viene fornito nessun `outputEncoding`, viene restituito un [`Buffer`][].

Il metodo `cipher.update()` può essere chiamato più volte con i nuovi dati finché non viene chiamato [`cipher.final()`][]. Chiamare `cipher.update()` dopo [`cipher.final()`][] genererà un errore.

## Class: Decipher

<!-- YAML
added: v0.1.94
-->

Le istanze della classe `Decipher` vengono utilizzate per decrittografare i dati. La classe può essere utilizzata in due modi:

- Come uno [stream](stream.html) che è sia readable che writable (leggibile e scrivibile), sul quale vengono scritti tramite il writing semplici dati criptati per produrre i dati non criptati sul lato readable, oppure
- Utilizzando i metodi [`decipher.update()`][] e [`decipher.final()`][] per produrre i dati non criptati.

I metodi [`crypto.createDecipher()`][] o [`crypto.createDecipheriv()`][] sono usati per creare istanze `Decipher`. Gli `Decipher` object non devono essere creati direttamente utilizzando la parola chiave `new`.

Esempio: Utilizzando gli `Decipher` object come degli stream:

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
  // Stampa: alcuni dati di testo chiari
});

const encrypted =
    'ca981be48e90867604588e75d04feabb63cc007a8f8ad89b10616ed84d815504';
decipher.write(encrypted, 'hex');
decipher.end();
```

Esempio: Utilizzando `Decipher` e i piped stream:

```js
const crypto = require('crypto');
const fs = require('fs');
const decipher = crypto.createDecipher('aes192', 'a password');

const input = fs.createReadStream('test.enc');
const output = fs.createWriteStream('test.js');

input.pipe(decipher).pipe(output);
```

Esempio: Utilizzando i metodi [`decipher.update()`][] e [`decipher.final()`][]:

```js
const crypto = require('crypto');
const decipher = crypto.createDecipher('aes192', 'a password');

const encrypted =
    'ca981be48e90867604588e75d04feabb63cc007a8f8ad89b10616ed84d815504';
let decrypted = decipher.update(encrypted, 'hex', 'utf8');
decrypted += decipher.final('utf8');
console.log(decrypted);
// Stampa: alcuni dati di testo chiari
```

### decipher.final([outputEncoding])

<!-- YAML
added: v0.1.94
-->

- `outputEncoding` {string}
- Restituisce: {Buffer | string} Qualsiasi contenuto decifrato restante. Se l'`outputEncoding` è `'latin1'`, `'ascii'` o `'utf8'`, viene restituita una stringa. Se non viene fornito nessun `outputEncoding`, viene restituito un [`Buffer`][].

Una volta chiamato il metodo `decipher.final()`, il `Decipher` object non può più essere utilizzato per decifrare i dati. I tentativi di chiamare `decipher.final()` più di una volta genereranno un errore.

### decipher.setAAD(buffer)

<!-- YAML
added: v1.0.0
changes:

  - version: v7.2.0
    pr-url: https://github.com/nodejs/node/pull/9398
    description: This method now returns a reference to `decipher`.
-->

- `buffer` {Buffer | TypedArray | DataView}
- Restituisce: {Cipher} per il method chaining.

Quando si utilizza una modalità di crittografia autenticata (attualmente sono supportate solo la `GCM` e la `CCM`), il metodo `decipher.setAAD()` imposta il valore utilizzato per il parametro input *additional authenticated data* (AAD).

Il metodo `decipher.setAAD()` dev'essere chiamato prima di [`decipher.update()`][].

### decipher.setAuthTag(buffer)

<!-- YAML
added: v1.0.0
changes:

  - version: v7.2.0
    pr-url: https://github.com/nodejs/node/pull/9398
    description: This method now returns a reference to `decipher`.
-->

- `buffer` {Buffer | TypedArray | DataView}
- Restituisce: {Cipher} per il method chaining.

Quando si utilizza una modalità di crittografia autenticata (attualmente sono supportare solo la `GCM` e la `CCM`), il metodo `decipher.setAuthTag()` viene utilizzato per passare l'*authentication tag* ricevuto. Se non viene fornito alcun tag, o se il testo cifrato è stato manomesso, verrà eseguito [`decipher.final()`][], indicando che il testo cifrato dovrebbe essere scartato a causa dell'autenticazione fallita.

Da notare che questa versione di Node.js non verifica la lunghezza degli authentication tag GCM. Tale controllo *deve* essere implementato dalle applicazioni ed è fondamentale per l'autenticità dei dati cifrati, altrimenti un utente malintenzionato potrebbe utilizzare un authentication tag arbitrariamente breve per aumentare le possibilità di passare l'autenticazione con successo (fino allo 0,39%). E' altamente consigliato di associare uno dei valori 16, 15, 14, 13, 12, 8 o 4 byte a ciascuna key e di accettare solo gli authentication tag di tale lunghezza, vedi [NIST SP 800-38D](http://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38d.pdf).

Il metodo `decipher.setAuthTag()` dev'essere chiamato prima di [`decipher.final()`][].

### decipher.setAutoPadding([autoPadding])

<!-- YAML
added: v0.7.1
-->

- `autoPadding` {boolean} **Default:** `true`
- Restituisce: {Cipher} per il method chaining.

Quando i dati sono stati cifrati senza il padding standard del blocco, la chiamata a `decipher.setAutoPadding(false)` disabiliterà il padding automatico in modo da impedire a [`decipher.final()`][] di cercare e rimuovere il padding.

La disattivazione del padding automatico sarà possibile solo se la lunghezza dei dati d'input è un multiplo della block size dei cipher.

Il metodo `decipher.setAutoPadding()` dev'essere chiamato prima di [`decipher.final()`][].

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
- Restituisce: {Buffer | string}

Aggiorna il decipher con `data`. Se viene specificato l'argomento `inputEncoding`, il suo valore dev'essere `'latin1'`, `'base64'` oppure `'hex'` e l'argomento `data` è una stringa che utilizza l'encoding specificato. Se non viene specificato l'argomento `inputEncoding`, `data` dev'essere un [`Buffer`][]. Se `data` è un [`Buffer`][] allora `inputEncoding` viene ignorato.

L'`outputEncoding` specifica il formato di output dei dati decifrati e può essere `'latin1'`, `'ascii'` o `'utf8'`. Se l'`outputEncoding` è specificato, viene restituita una stringa che utilizza l'encoding specificato. Se non viene fornito nessun `outputEncoding`, viene restituito un [`Buffer`][].

Il metodo `decipher.update()` può essere chiamato più volte con i nuovi dati finché non viene chiamato [`decipher.final()`][]. Chiamare `decipher.update()` dopo [`decipher.final()`][] genererà un errore.

## Class: DiffieHellman

<!-- YAML
added: v0.5.0
-->

La classe `DiffieHellman` è un'utility per la creazione di scambi di chiavi Diffie-Hellman.

Le istanze della classe `DiffieHellman` possono essere create utilizzando la funzione [`crypto.createDiffieHellman()`][].

```js
const crypto = require('crypto');
const assert = require('assert');

// Genera le chiavi di Alice...
const alice = crypto.createDiffieHellman(2048);
const aliceKey = alice.generateKeys();

// Genera le chiavi di Bob...
const bob = crypto.createDiffieHellman(alice.getPrime(), alice.getGenerator());
const bobKey = bob.generateKeys();

// Esegue lo scambio e genera la chiave segreta...
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
- Restituisce: {Buffer | string}

Calcola la chiave segreta condivisa utilizzando `otherPublicKey` come la public key (chiave pubblica) dell'altra parte e restituisce la chiave segreta condivisa calcolata. La chiave fornita viene interpretata utilizzando l'`inputEncoding` specificato, e la chiave segreta viene codificata utilizzando l'`outputEncoding` specificato. L'encoding può essere `'latin1'`, `'hex'` o `'base64'`. Se non viene fornito l'`inputEncoding`, `otherPublicKey` dovrebbe essere un [`Buffer`][], un `TypedArray` o un `DataView`.

Se viene fornito l'`outputEncoding` viene restituita una stringa; in caso contrario, viene restituito un [`Buffer`][].

### diffieHellman.generateKeys([encoding])

<!-- YAML
added: v0.5.0
-->

- `encoding` {string}
- Restituisce: {Buffer | string}

Genera valori di chiave Diffie-Hellman privata e pubblica, e restituisce la chiave pubblica nell'`encoding` specificato. Questa chiave dovrebbe essere trasferita all'altra parte. L'encoding può essere `'latin1'`, `'hex'` o `'base64'`. Se viene fornito l'`encoding` viene restituita una stringa; in caso contrario, viene restituito un [`Buffer`][].

### diffieHellman.getGenerator([encoding])

<!-- YAML
added: v0.5.0
-->

- `encoding` {string}
- Restituisce: {Buffer | string}

Restituisce il generatore Diffie-Hellman nell'`encoding` specificato, che può essere `'latin1'`, `'hex'` o `'base64'`. Se viene fornito l'`encoding` viene restituita una stringa; in caso contrario, viene restituito un [`Buffer`][].

### diffieHellman.getPrime([encoding])

<!-- YAML
added: v0.5.0
-->

- `encoding` {string}
- Restituisce: {Buffer | string}

Restituisce il Diffie-Hellman prime (numero primo) nell'`encoding` specificato, che può essere `'latin1'`, `'hex'` o `'base64'`. Se viene fornito l'`encoding` viene restituita una stringa; in caso contrario, viene restituito un [`Buffer`][].

### diffieHellman.getPrivateKey([encoding])

<!-- YAML
added: v0.5.0
-->

- `encoding` {string}
- Restituisce: {Buffer | string}

Restituisce la Diffie-Hellman private key (chiave privata) nell'`encoding` specificato, che può essere `'latin1'`, `'hex'` o `'base64'`. Se viene fornito l'`encoding` viene restituita una stringa; in caso contrario, viene restituito un [`Buffer`][].

### diffieHellman.getPublicKey([encoding])

<!-- YAML
added: v0.5.0
-->

- `encoding` {string}
- Restituisce: {Buffer | string}

Restituisce la chiave pubblica Diffie-Hellman nell'`encoding` specificato, che può essere `'latin1'`, `'hex'` o `'base64'`. Se viene fornito l'`encoding` viene restituita una stringa; in caso contrario, viene restituito un [`Buffer`][].

### diffieHellman.setPrivateKey(privateKey[, encoding])

<!-- YAML
added: v0.5.0
-->

- `privateKey` {string | Buffer | TypedArray | DataView}
- `encoding` {string}

Imposta la chiave privata Diffie-Hellman. Se viene fornito l'argomento `encoding` ed è `'latin1'`, `'hex'` o `'base64'`, `privateKey` dovrebbe essere una stringa. Se non viene fornito nessun `encoding`, `privateKey` dovrebbe essere un [`Buffer`][], un `TypedArray` o un `DataView`.

### diffieHellman.setPublicKey(publicKey[, encoding])

<!-- YAML
added: v0.5.0
-->

- `publicKey` {string | Buffer | TypedArray | DataView}
- `encoding` {string}

Imposta la chiave pubblica Diffie-Hellman. Se viene fornito l'argomento `encoding` ed è `'latin1'`, `'hex'` o `'base64'`, `publicKey` dovrebbe essere una stringa. Se non viene fornito nessun `encoding`, `publicKey` dovrebbe essere un [`Buffer`][], un `TypedArray` o un `DataView`.

### diffieHellman.verifyError

<!-- YAML
added: v0.11.12
-->

Un campo di bit contenente eventuali avvisi e/o errori risultanti da un controllo eseguito durante l'inizializzazione del `DiffieHellman` object.

I seguenti valori sono validi per questa proprietà (come definita nel modulo `constants`):

- `DH_CHECK_P_NOT_SAFE_PRIME`
- `DH_CHECK_P_NOT_PRIME`
- `DH_UNABLE_TO_CHECK_GENERATOR`
- `DH_NOT_SUITABLE_GENERATOR`

## Class: ECDH

<!-- YAML
added: v0.11.14
-->

La classe `ECDH` è un'utility per la creazione di scambi di chiavi Elliptic Curve Diffie-Hellman (ECDH).

Le istanze della classe `ECDH` possono essere create utilizzando la funzione [`crypto.createECDH()`][].

```js
const crypto = require('crypto');
const assert = require('assert');

// Genera le chiavi di Alice...
const alice = crypto.createECDH('secp521r1');
const aliceKey = alice.generateKeys();

// Genera le chiavi di Bob...
const bob = crypto.createECDH('secp521r1');
const bobKey = bob.generateKeys();

// Esegue lo scambio e genera la chiave segreta...
const aliceSecret = alice.computeSecret(bobKey);
const bobSecret = bob.computeSecret(aliceKey);

assert.strictEqual(aliceSecret.toString('hex'), bobSecret.toString('hex'));
// OK
```

### ECDH.convertKey(key, curve[, inputEncoding[, outputEncoding[, format]]])

<!-- YAML
added: v10.0.0
-->

- `key` {string | Buffer | TypedArray | DataView}
- `curve` {string}
- `inputEncoding` {string}
- `outputEncoding` {string}
- `format` {string} **Default:** `'uncompressed'`
- Restituisce: {Buffer | string}

Converte la chiave pubblica EC Diffie-Hellman specificata tramite `key` e `curve` nel formato specificato da `format`. L'argomento `format` specifica l'encoding del punto e può essere `'compressed'`, `'uncompressed'` oppure `'hybrid'`. La chiave fornita viene interpretata utilizzando l'`inputEncoding` specificato e la chiave restituita viene codificata utilizzando l'`outputEncoding` specificato. Gli encoding possono essere `'latin1'`, `'hex'` o `'base64'`.

Utilizza [`crypto.getCurves()`][] per ottenere un elenco di nomi di curve disponibili. Nelle versioni OpenSSL recenti, `openssl ecparam -list_curves` mostrerà anche il nome e la descrizione di ciascuna curva ellittica disponibile.

Se non viene specificato `format`, il punto verrà restituito nel formato `'uncompressed'`.

Se non viene fornito l'`inputEncoding`, `key` dovrebbe essere un [`Buffer`][], un `TypedArray` o un `DataView`.

Esempio (decompressione di una chiave):

```js
const { ECDH } = require('crypto');

const ecdh = ECDH('secp256k1');
ecdh.generateKeys();

const compressedKey = ecdh.getPublicKey('hex', 'compressed');

const uncompressedKey = ECDH.convertKey(compressedKey,
                                        'secp256k1',
                                        'hex',
                                        'hex',
                                        'uncompressed');

// la chiave convertita e la chiave pubblica non compressa devono essere uguali
console.log(uncompressedKey === ecdh.getPublicKey('hex'));
```

### ecdh.computeSecret(otherPublicKey\[, inputEncoding\]\[, outputEncoding\])

<!-- YAML
added: v0.11.14
changes:

  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5522
    description: The default `inputEncoding` changed from `binary` to `utf8`
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/16849
    description: Changed error format to better support invalid public key
                 error
-->

- `otherPublicKey` {string | Buffer | TypedArray | DataView}
- `inputEncoding` {string}
- `outputEncoding` {string}
- Restituisce: {Buffer | string}

Calcola la chiave segreta condivisa utilizzando `otherPublicKey` come chiave pubblica dell'altra parte e restituisce la chiave segreta condivisa calcolata. La chiave fornita viene interpretata utilizzando l'`inputEncoding` specificato e la chiave segreta viene codificata utilizzando l'`outputEncoding` specificato. Gli encoding possono essere `'latin1'`, `'hex'` o `'base64'`. Se non viene fornito l'`inputEncoding`, `otherPublicKey` dovrebbe essere un [`Buffer`][], un `TypedArray` o un `DataView`.

Se viene fornito l'`outputEncoding` viene restituita una stringa; in caso contrario, viene restituito un [`Buffer`][].

`ecdh.computeSecret` genererà un errore `ERR_CRYPTO_ECDH_INVALID_PUBLIC_KEY` quando `otherPublicKey` si trova al di fuori della curva ellittica. Poiché `otherPublicKey` viene solitamente fornita da un utente remoto su una rete non sicura, di conseguenza gli sviluppatori sono invitati a gestire quest'eccezione.

### ecdh.generateKeys([encoding[, format]])

<!-- YAML
added: v0.11.14
-->

- `encoding` {string}
- `format` {string} **Default:** `'uncompressed'`
- Restituisce: {Buffer | string}

Genera valori di chiave EC Diffie-Hellman privata e pubblica, e restituisce la chiave pubblica nel `format` e nell'`encoding` specificati. Questa chiave dovrebbe essere trasferita all'altra parte.

L'argomento `format` specifica l'encoding del punto e può essere `'compressed'` oppure `'uncompressed'`. Se non viene specificato `format`, il punto verrà restituito nel formato `'uncompressed'`.

L'argomento `encoding` può essere `'latin1'`, `'hex'` o `'base64'`. Se viene fornito l'`encoding` viene restituita una stringa; in caso contrario, viene restituito un [`Buffer`][].

### ecdh.getPrivateKey([encoding])

<!-- YAML
added: v0.11.14
-->

- `encoding` {string}
- Restituisce: {Buffer | string} La chiave privata EC Diffie-Hellman nell'`encoding` specificato, che può essere `'latin1'`, `'hex'` o `'base64'`. Se viene fornito l'`encoding` viene restituita una stringa; in caso contrario, viene restituito un [`Buffer`][].

### ecdh.getPublicKey(\[encoding\]\[, format\])

<!-- YAML
added: v0.11.14
-->

- `encoding` {string}
- `format` {string} **Default:** `'uncompressed'`
- Restituisce: {Buffer | string} La chiave pubblica EC Diffie-Hellman nell'`encoding` e nel `format` specificati.

L'argomento `format` specifica l'encoding del punto e può essere `'compressed'` oppure `'uncompressed'`. Se non viene specificato `format`, il punto verrà restituito nel formato `'uncompressed'`.

L'argomento `encoding` può essere `'latin1'`, `'hex'` o `'base64'`. Se viene fornito l'`encoding` viene restituita una stringa; in caso contrario, viene restituito un [`Buffer`][].

### ecdh.setPrivateKey(privateKey[, encoding])

<!-- YAML
added: v0.11.14
-->

- `privateKey` {string | Buffer | TypedArray | DataView}
- `encoding` {string}

Imposta la chiave privata EC Diffie-Hellman. L'`encoding` può essere `'latin1'`, `'hex'` o `'base64'`. Se viene fornito l'`encoding`, `privateKey` dovrebbe essere una stringa; in caso contrario `privateKey` dovrebbe essere un [`Buffer`][], un `TypedArray` o un `DataView`.

Se la `privateKey` non è valida per la curva specificata quando è stato creato l'`ECDH` object, viene generato un errore. Dopo aver impostato la chiave privata, viene generato e impostato nel `ECDH` object anche il punto (chiave) pubblico associato.

### ecdh.setPublicKey(publicKey[, encoding])

<!-- YAML
added: v0.11.14
deprecated: v5.2.0
-->

> Stabilità: 0 - Obsoleto

- `publicKey` {string | Buffer | TypedArray | DataView}
- `encoding` {string}

Imposta la chiave pubblica EC Diffie-Hellman. L'encoding della chiave può essere `'latin1'`, `'hex'` o `'base64'`. Se viene fornito l'`encoding`, la `publicKey` dovrebbe essere una stringa; in caso contrario dovrebbe essere un [`Buffer`][], un `TypedArray` o un `DataView`.

Da notare che normalmente non esiste un motivo per chiamare questo metodo poiché `ECDH` richiede solo una chiave privata e la chiave pubblica dell'altra parte per calcolare la chiave segreta condivisa. Solitamente viene chiamato [`ecdh.generateKeys()`][] oppure [`ecdh.setPrivateKey()`][]. Il metodo [`ecdh.setPrivateKey()`][] tenta di generare il punto pubblico/la chiave pubblica associati alla chiave privata impostata.

Esempio (ottenendo una chiave segreta condivisa):

```js
const crypto = require('crypto');
const alice = crypto.createECDH('secp256k1');
const bob = crypto.createECDH('secp256k1');

// Nota: questa è una scorciatoia per specificare una delle chiavi private di Alice
// precedenti. Non sarebbe saggio usare una chiave privata così prevedibile in una vera
// applicazione.
alice.setPrivateKey(
  crypto.createHash('sha256').update('alice', 'utf8').digest()
);

// Bob usa una coppia di chiavi pseudocasuali
// ben cifrate
bob.generateKeys();

const aliceSecret = alice.computeSecret(bob.getPublicKey(), null, 'hex');
const bobSecret = bob.computeSecret(alice.getPublicKey(), null, 'hex');

// aliceSecret e bobSecret dovrebbero avere lo stesso valore segreto condiviso
console.log(aliceSecret === bobSecret);
```

## Class: Hash

<!-- YAML
added: v0.1.92
-->

La classe `Hash` è un'utility per creare hash digest di dati. Può essere utilizzata in due modi:

- Come uno [stream](stream.html) che è sia readable che writable (leggibile e scrivibile), sul quale vengono scritti tramite il writing i dati per produrre un hash digest calcolato sul lato readable, oppure
- Utilizzando i metodi [`hash.update()`][] e [`hash.digest()`][] per produrre l'hash calcolato.

Le istanze `Hash` vengono create utilizzando il metodo [`crypto.createHash()`][]. Gli `Hash` object non devono essere creati direttamente utilizzando la parola chiave `new`.

Esempio: Utilizzando gli `Hash` object come degli stream:

```js
const crypto = require('crypto');
const hash = crypto.createHash('sha256');

hash.on('readable', () => {
  const data = hash.read();
  if (data) {
    console.log(data.toString('hex'));
    // Stampa:
    //   6a2da20943931e9834fc12cfe5bb47bbd9ae43489a30726962b576f4e3993e50
  }
});

hash.write('some data to hash');
hash.end();
```

Esempio: Utilizzando `Hash` e i piped stream:

```js
const crypto = require('crypto');
const fs = require('fs');
const hash = crypto.createHash('sha256');

const input = fs.createReadStream('test.js');
input.pipe(hash).pipe(process.stdout);
```

Esempio: Utilizzando i metodi [`hash.update()`][] e [`hash.digest()`][]:

```js
const crypto = require('crypto');
const hash = crypto.createHash('sha256');

hash.update('some data to hash');
console.log(hash.digest('hex'));
// Stampa:
//   6a2da20943931e9834fc12cfe5bb47bbd9ae43489a30726962b576f4e3993e50
```

### hash.digest([encoding])

<!-- YAML
added: v0.1.92
-->

- `encoding` {string}
- Restituisce: {Buffer | string}

Calcola il digest di tutti i dati passati per essere sottoposti all'hash (utilizzando il metodo [`hash.update()`][]). L'`encoding` può essere `'hex'`, `'latin1'` o `'base64'`. Se viene fornito l'`encoding` viene restituita una stringa; in caso contrario, viene restituito un [`Buffer`][].

L'`Hash` object non può essere utilizzato nuovamente dopo aver chiamato il metodo `hash.digest()`. Chiamate multiple genereranno un errore.

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

Aggiorna il contenuto dell'hash con il `data` fornito, il cui encoding è fornito all'interno dell'`inputEncoding` e può essere `'utf8'`, `'ascii'` o `'latin1'`. Se non viene fornito l'`encoding`, e `data` è una stringa, viene imposto un encoding di `'utf8'`. Se `data` è un [`Buffer`][], `TypedArray` o un `DataView`, allora `inputEncoding` viene ignorato.

Può essere chiamato più volte con i nuovi dati mentre viene eseguito lo streaming.

## Class: Hmac

<!-- YAML
added: v0.1.94
-->

La classe `Hmac` è un'utility per la creazione di digest HMAC crittografici. Può essere utilizzata in due modi:

- Come uno [stream](stream.html) che è sia readable che writable (leggibile e scrivibile), sul quale vengono scritti tramite il writing i dati per produrre un HMAC digest calcolato sul lato readable, oppure
- Utilizzando i metodi [`hmac.update()`][] e [`hmac.digest()`][] per produrre l'HMAC digest calcolato.

Le istanze `Hmac` vengono create utilizzando il metodo [`crypto.createHmac()`][]. Gli `Hmac` object non devono essere creati direttamente utilizzando la parola chiave `new`.

Esempio: Utilizzando gli `Hmac` object come degli stream:

```js
const crypto = require('crypto');
const hmac = crypto.createHmac('sha256', 'a secret');

hmac.on('readable', () => {
  const data = hmac.read();
  if (data) {
    console.log(data.toString('hex'));
    // Stampa:
    //   7fd04df92f636fd450bc841c9418e5825c17f33ad9c87c518115a45971f7f77e
  }
});

hmac.write('some data to hash');
hmac.end();
```

Esempio: Utilizzando `Hmac` e i piped stream:

```js
const crypto = require('crypto');
const fs = require('fs');
const hmac = crypto.createHmac('sha256', 'a secret');

const input = fs.createReadStream('test.js');
input.pipe(hmac).pipe(process.stdout);
```

Esempio: Utilizzando i metodi [`hmac.update()`][] e [`hmac.digest()`][]:

```js
const crypto = require('crypto');
const hmac = crypto.createHmac('sha256', 'a secret');

hmac.update('some data to hash');
console.log(hmac.digest('hex'));
// Stampa:
//   7fd04df92f636fd450bc841c9418e5825c17f33ad9c87c518115a45971f7f77e
```

### hmac.digest([encoding])

<!-- YAML
added: v0.1.94
-->

- `encoding` {string}
- Restituisce: {Buffer | string}

Calcola l'HMAC digest di tutti i dati passati utilizzando [`hmac.update()`][]. L'`encoding` può essere `'hex'`, `'latin1'` o `'base64'`. Se viene fornito l'`encoding` viene restituita una stringa; in caso contrario, viene restituito un [`Buffer`][];

L'`Hmac` object non può essere utilizzato nuovamente dopo aver chiamato il metodo `hmac.digest()`. Chiamate multiple di `hmac.digest()` genereranno un errore.

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

Aggiorna il contenuto dell'`Hmac` con il `data` fornito, il cui encoding è fornito all'interno dell'`inputEncoding` e può essere `'utf8'`, `'ascii'` o `'latin1'`. Se non viene fornito l'`encoding`, e `data` è una stringa, viene imposto un encoding di `'utf8'`. Se `data` è un [`Buffer`][], `TypedArray` o un `DataView`, allora `inputEncoding` viene ignorato.

Può essere chiamato più volte con i nuovi dati mentre viene eseguito lo streaming.

## Class: Sign

<!-- YAML
added: v0.1.92
-->

La classe `Sign` è un'utility per la generazione di firme. Può essere utilizzata in due modi:

- Come un writable [stream](stream.html), sul quale vengono scritti tramite il writing i dati da firmare e il metodo [`sign.sign()`][] viene utilizzato per generare e restituire la firma, oppure
- Utilizzando i metodi [`sign.update()`][] e [`sign.sign()`][] per produrre la firma.

Le istanze `Sign` vengono create utilizzando il metodo [`crypto.createSign()`][]. L'argomento è il nome della stringa della funzione hash da utilizzare. I `Sign` object non devono essere creati direttamente utilizzando la parola chiave `new`.

Esempio: Utilizzando i `Sign` object come degli stream:

```js
const crypto = require('crypto');
const sign = crypto.createSign('SHA256');

sign.write('some data to sign');
sign.end();

const privateKey = getPrivateKeySomehow();
console.log(sign.sign(privateKey, 'hex'));
// Stampa: la firma calcolata utilizzando la chiave privata e
// l'SHA-256 specificati. Per le chiavi RSA, l'algoritmo è 
// RSASSA-PKCS1-v1_5 (vedi il parametro padding in basso per RSASSA-PSS). Per le chiavi EC, l'algoritmo è ECDSA.
```

Esempio: Utilizzando i metodi [`sign.update()`][] e [`sign.sign()`][]:

```js
const crypto = require('crypto');
const sign = crypto.createSign('SHA256');

sign.update('some data to sign');

const privateKey = getPrivateKeySomehow();
console.log(sign.sign(privateKey, 'hex'));
// Stampa: la firma calcolata
```

In alcuni casi, è anche possibile creare un'istanza `Sign` passando un nome dell'algoritmo di una firma, come ad esempio 'RSA-SHA256'. Questo utilizzerà l'algoritmo del digest corrispondente. Questo non funziona per tutti gli algoritmi delle firma, ad esempio per 'ecdsa-with-SHA256' non funziona. In questo caso utilizza i nomi dei digest.

Esempio: firma utilizzando il nome dell'algoritmo della firma legacy

```js
const crypto = require('crypto');
const sign = crypto.createSign('RSA-SHA256');

sign.update('some data to sign');

const privateKey = getPrivateKeySomehow();
console.log(sign.sign(privateKey, 'hex'));
// Stampa: la firma calcolata
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
- Restituisce: {Buffer | string}

Calcola la firma su tutti i dati passati utilizzando [`sign.update()`][] oppure [`sign.write()`](stream.html#stream_writable_write_chunk_encoding_callback).

L'argomento `privateKey` può essere un object o una stringa. Se `privateKey` è una stringa, viene trattato come una raw key senza passphrase (frase d'accesso). Se `privateKey` è un object, deve contenere una o più delle seguenti proprietà:

- `key`: {string} - Chiave privata con codifica PEM (obbligatoria)
- `passphrase`: {string} - passphrase (frase d'accesso) per la chiave privata
- `padding`: {integer} - Valore padding opzionale per RSA, può essere uno dei seguenti:
  
  - `crypto.constants.RSA_PKCS1_PADDING` (valore di default)
  - `crypto.constants.RSA_PKCS1_PSS_PADDING`
  
  Da notare che `RSA_PKCS1_PSS_PADDING` utilizzerà MGF1 con la stessa funzione hash utilizzata per firmare il messaggio come specificato nella sezione 3.1 del documento [RFC 4055](https://www.rfc-editor.org/rfc/rfc4055.txt).

- `saltLength`: {integer} - lunghezza del salt per quando il padding è `RSA_PKCS1_PSS_PADDING`. Il valore speciale `crypto.constants.RSA_PSS_SALTLEN_DIGEST` imposta la lunghezza del salt nella dimensione del digest, `crypto.constants.RSA_PSS_SALTLEN_MAX_SIGN` (valore di default) lo imposta sul valore massimo consentito.

L'`outputFormat` può essere `'latin1'`, `'hex'` oppure `'base64'`. Se viene fornito l'`outputFormat` viene restituita una stringa; in caso contrario, viene restituito un [`Buffer`][].

Il `Sign` object non può essere utilizzato nuovamente dopo aver chiamato il metodo `sign.sign()`. Chiamate multiple di `sign.sign()` genereranno un errore.

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

Aggiorna il contenuto di `Sign` con il `data` fornito, il cui encoding è fornito all'interno dell'`inputEncoding` e può essere `'utf8'`, `'ascii'` o `'latin1'`. Se non viene fornito l'`encoding`, e `data` è una stringa, viene imposto un encoding di `'utf8'`. Se `data` è un [`Buffer`][], `TypedArray` o un `DataView`, allora `inputEncoding` viene ignorato.

Può essere chiamato più volte con i nuovi dati mentre viene eseguito lo streaming.

## Class: Verify

<!-- YAML
added: v0.1.92
-->

La classe `Verify` è un'utility per verificare le firme. Può essere utilizzata in due modi:

- Come un writable [stream](stream.html), sul quale i dati scritti tramite il writing vengono utilizzati per convalidare la firma fornita, oppure
- Utilizzando i metodi [`verify.update()`][] e [`verify.verify()`][] per verificare la firma.

Le istanze `Verify` vengono create utilizzando il metodo [`crypto.createVerify()`][]. Gli `Verify` object non devono essere creati direttamente utilizzando la parola chiave `new`.

Esempio: Utilizzando gli `Verify` object come degli stream:

```js
const crypto = require('crypto');
const verify = crypto.createVerify('SHA256');

verify.write('some data to sign');
verify.end();

const publicKey = getPublicKeySomehow();
const signature = getSignatureToVerify();
console.log(verify.verify(publicKey, signature));
// Stampa: true o false
```

Esempio: Utilizzando i metodi [`verify.update()`][] e [`verify.verify()`][]:

```js
const crypto = require('crypto');
const verify = crypto.createVerify('SHA256');

verify.update('some data to sign');

const publicKey = getPublicKeySomehow();
const signature = getSignatureToVerify();
console.log(verify.verify(publicKey, signature));
// Stampa: true o false
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

Aggiorna il contenuto di `Verify` con il `data` fornito, il cui encoding è fornito all'interno dell'`inputEncoding` e può essere `'utf8'`, `'ascii'` o `'latin1'`. Se non viene fornito l'`encoding`, e `data` è una stringa, viene imposto un encoding di `'utf8'`. Se `data` è un [`Buffer`][], `TypedArray` o un `DataView`, allora `inputEncoding` viene ignorato.

Può essere chiamato più volte con i nuovi dati mentre viene eseguito lo streaming.

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
- Restituisce: {boolean} `true` o `false` a seconda della validità della firma per i dati e la chiave pubblica.

Verifica i dati forniti utilizzando l'`object` e la `signature` specificati. L'argomento `object` può essere una stringa contenente un object con codifica PEM, il quale può essere una chiave pubblica RSA, una chiave pubblica DSA oppure un certificato X.509 o un object con una o più delle seguenti proprietà:

- `key`: {string} - Chiave pubblica con codifica PEM (obbligatoria)
- `padding`: {integer} - Valore padding opzionale per RSA, può essere uno dei seguenti:
  
  - `crypto.constants.RSA_PKCS1_PADDING` (valore di default)
  - `crypto.constants.RSA_PKCS1_PSS_PADDING`
  
  Da notare che `RSA_PKCS1_PSS_PADDING` utilizzerà MGF1 con la stessa funzione hash utilizzata per verificare il messaggio come specificato nella sezione 3.1 del documento [RFC 4055](https://www.rfc-editor.org/rfc/rfc4055.txt).

- `saltLength`: {integer} - lunghezza del salt per quando il padding è `RSA_PKCS1_PSS_PADDING`. Il valore speciale `crypto.constants.RSA_PSS_SALTLEN_DIGEST` imposta la lunghezza del salta nella dimensione del digest, `crypto.constants.RSA_PSS_SALTLEN_AUTO` (valore di default) fa sì che venga determinato automaticamente.

L'argomento `signature` è la firma calcolata precedentemente per i dati, all'interno del `signatureFormat` che può essere `'latin1'`, `'hex'` o `'base64'`. Se viene fornito `signatureFormat`, la `signature` dovrebbe essere una stringa; in caso contrario la `signature` dovrebbe essere un [`Buffer`][], un `TypedArray`, o un `DataView`.

Il `verify` object non può essere utilizzato nuovamente dopo aver chiamato `verify.verify()`. Chiamate multiple di `verify.verify()` genereranno un errore.

## Metodi e proprietà del modulo `crypto`

### crypto.constants

<!-- YAML
added: v6.3.0
-->

- Restituisce: {Object} Un object contenente costanti di uso comune per operazioni crittografiche e relative alla sicurezza. Le costanti specifiche attualmente definite sono descritte in [Costanti Crittografiche](#crypto_crypto_constants_1).

### crypto.DEFAULT_ENCODING

<!-- YAML
added: v0.9.3
deprecated: v10.0.0
-->

L'encoding predefinito da utilizzare per le funzioni che possono utilizzare le stringhe oppure i [buffers] [`Buffer`]. Il valore predefinito è `'buffer'`, il quale rende i metodi predefiniti ai [`Buffer`][] object.

Il meccanismo `crypto.DEFAULT_ENCODING` viene fornito per la retrocompatibilità con i programmi legacy che prevedono che `'latin1'` sia l'encoding predefinito.

Le nuove applicazioni dovrebbero aspettarsi che il valore predefinito sia `'buffer'`.

Questa proprietà è obsoleta.

### crypto.fips

<!-- YAML
added: v6.0.0
deprecated: v10.0.0
-->

Proprietà per verificare e controllare se è attualmente in uso un provider crittografico conforme alle norme FIPS. L'impostazione su true richiede una build delle norme FIPS di Node.js.

Questa proprietà è obsoleta. Perfavore utilizza `crypto.setFips()` al posto di `crypto.getFips()`.

### crypto.createCipher(algorithm, password[, options])

<!-- YAML
added: v0.1.94
deprecated: v10.0.0
-->

> Stabilità: 0 - Obsoleto: Utilizza invece [`crypto.createCipheriv()`][].

- `algorithm` {string}
- `password` {string | Buffer | TypedArray | DataView}
- `options` {Object} [`stream.transform` options][]
- Restituisce: {Cipher}

Crea e restituisce un `Cipher` object che utilizza l'`algorithm` e la `password` specificati.

L'argomento `options` controlla il comportamento dello stream ed è facoltativo eccetto quando viene utilizzato un cipher in modalità CCM (ad es. `'aes-128-ccm'`). In tal caso, è richiesta l'opzione `authTagLength` che specifica la lunghezza dell'authentication tag in byte, vedi [Modalità CCM](#crypto_ccm_mode).

L'`algorithm` dipende da OpenSSL, alcuni esempi sono `'aes192'`, ecc. Nelle versioni OpenSSL recenti, `openssl list -cipher-algorithms` (`openssl list-cipher-algorithms` per le versioni precedenti di OpenSSL) mostrerà gli algoritmi cipher disponibili.

La `password` viene utilizzata per ricavare la chiave di cifratura (cipher key) e il vettore di inizializzazione (IV). Il valore deve essere una stringa con codifica `'latin1'`, un [`Buffer`][], un `TypedArray`, oppure un `DataView`.

L'implementazione di `crypto.createCipher()` deriva le chiavi utilizzando la funzione OpenSSL [`EVP_BytesToKey`][] con l'algoritmo digest impostato su MD5, una iterazione e nessun salt. La mancanza di salt consente attacchi a dizionario poiché la stessa password crea sempre la stessa chiave. Il basso numero di iterazioni e l'algoritmo hash, non crittograficamente sicuro, permettono di testare le password molto rapidamente.

In linea con la raccomandazione di OpenSSL di utilizzare PBKDF2 invece di [`EVP_BytesToKey`][], si consiglia agli sviluppatori di derivare una chiave e l'IV autonomamente utilizzando [`crypto.pbkdf2()`][] e di utilizzare [`crypto.createCipheriv()`][] per creare il `Cipher` object. Gli utenti non devono utilizzare i cipher con la counter mode (ad es. CTR, GCM, o CCM) all'interno di `crypto.createCipher()`. Viene emesso un avviso quando vengono utilizzati al fine di non rischiare riutilizzando l'IV in quanto il riutilizzo causa vulnerabilità. Per il caso in cui l'IV viene riutilizzato in modalità GCM, vedi [Nonce-Disrespecting Adversaries](https://github.com/nonce-disrespect/nonce-disrespect) per maggiori dettagli.

### crypto.createCipheriv(algorithm, key, iv[, options])

<!-- YAML
added: v0.1.94
changes:

  - version: v9.9.0
    pr-url: https://github.com/nodejs/node/pull/18644
    description: The `iv` parameter may now be `null` for ciphers which do not
                 need an initialization vector.
-->

- `algorithm` {string}
- `key` {string | Buffer | TypedArray | DataView}
- `iv` {string | Buffer | TypedArray | DataView}
- `options` {Object} [`stream.transform` options][]
- Restituisce: {Cipher}

Crea e restituisce un `Cipher` object, con l'`algorithm`, la `key` e il vettore di inizializzazione (`iv`) specificati.

L'argomento `options` controlla il comportamento dello stream ed è facoltativo eccetto quando viene utilizzato un cipher in modalità CCM (ad es. `'aes-128-ccm'`). In tal caso, è richiesta l'opzione `authTagLength` che specifica la lunghezza dell'authentication tag in byte, vedi [Modalità CCM](#crypto_ccm_mode).

L'`algorithm` dipende da OpenSSL, alcuni esempi sono `'aes192'`, ecc. Nelle versioni OpenSSL recenti, `openssl list -cipher-algorithms` (`openssl list-cipher-algorithms` per le versioni precedenti di OpenSSL) mostrerà gli algoritmi cipher disponibili.

La `key` è la raw key utilizzata dall'`algorithm` e `iv` è un [vettore di inizializzazione](https://en.wikipedia.org/wiki/Initialization_vector). Entrambi gli argomenti devono essere delle stringhe con codifica `'utf8'`, dei [Buffers][`Buffer`], dei `TypedArray`, o dei `DataView`. Se il cipher non ha bisogno di un vettore di inizializzazione, `iv` potrebbe essere `null`.

I vettori di inizializzazione dovrebbero essere imprevedibili e unici; idealmente, saranno crittograficamente casuali. Non devono essere segreti: gli IV vengono in genere aggiunti ai messaggi ciphertext non crittografati. Potrebbe sembrare contraddittorio che un qualcosa debba essere imprevedibile e unico, ma non debba essere segreto; è importante ricordare che un aggressore non deve essere in grado di prevedere in anticipo quale possa essere un dato IV.

### crypto.createCredentials(details)

<!-- YAML
added: v0.1.92
deprecated: v0.11.13
-->

> Stabilità: 0 - Obsoleto: Utilizza invece [`tls.createSecureContext()`][].

- `details` {Object} Identico a [`tls.createSecureContext()`][].
- Restituisce: {tls.SecureContext}

Il metodo `crypto.createCredentials()` è una funzione obsoleta per la creazione e la restituzione di un `tls.SecureContext`. Non dovrebbe essere usato. Sostituiscilo con [`tls.createSecureContext()`][] che ha gli stessi identici argomenti e lo stesso valore di return.

Restituisce un `tls.SecureContext`, come se [`tls.createSecureContext()`][] fosse stato chiamato.

### crypto.createDecipher(algorithm, password[, options])

<!-- YAML
added: v0.1.94
deprecated: v10.0.0
-->

> Stabilità: 0 - Obsoleto: Utilizza invece [`crypto.createDecipheriv()`][].

- `algorithm` {string}
- `password` {string | Buffer | TypedArray | DataView}
- `options` {Object} [`stream.transform` options][]
- Restituisce: {Decipher}

Crea e restituisce un `Decipher` object che utilizza l'`algorithm` e la `password` (chiave) specificati.

L'argomento `options` controlla il comportamento dello stream ed è facoltativo eccetto quando viene utilizzato un cipher in modalità CCM (ad es. `'aes-128-ccm'`). In tal caso, è richiesta l'opzione `authTagLength` che specifica la lunghezza dell'authentication tag in byte, vedi [Modalità CCM](#crypto_ccm_mode).

L'implementazione di `crypto.createDecipher()` deriva le chiavi utilizzando la funzione OpenSSL [`EVP_BytesToKey`][] con l'algoritmo digest impostato su MD5, una iterazione e nessun salt. La mancanza di salt consente attacchi a dizionario poiché la stessa password crea sempre la stessa chiave. Il basso numero di iterazioni e l'algoritmo hash, non crittograficamente sicuro, permettono di testare le password molto rapidamente.

In linea con la raccomandazione di OpenSSL di utilizzare PBKDF2 invece di [`EVP_BytesToKey`][], si consiglia agli sviluppatori di derivare una chiave e l'IV autonomamente utilizzando [`crypto.pbkdf2()`][] e di utilizzare [`crypto.createDecipheriv()`][] per creare il `Decipher` object.

### crypto.createDecipheriv(algorithm, key, iv[, options])

<!-- YAML
added: v0.1.94
changes:

  - version: v9.9.0
    pr-url: https://github.com/nodejs/node/pull/18644
    description: The `iv` parameter may now be `null` for ciphers which do not
                 need an initialization vector.
-->

- `algorithm` {string}
- `key` {string | Buffer | TypedArray | DataView}
- `iv` {string | Buffer | TypedArray | DataView}
- `options` {Object} [`stream.transform` options][]
- Restituisce: {Decipher}

Crea e restituisce un `Decipher` object che utilizza l'`algorithm`, la `key` e il vettore di inizializzazione (`iv`) specificati.

L'argomento `options` controlla il comportamento dello stream ed è facoltativo eccetto quando viene utilizzato un cipher in modalità CCM (ad es. `'aes-128-ccm'`). In tal caso, è richiesta l'opzione `authTagLength` che specifica la lunghezza dell'authentication tag in byte, vedi [Modalità CCM](#crypto_ccm_mode).

L'`algorithm` dipende da OpenSSL, alcuni esempi sono `'aes192'`, ecc. Nelle versioni OpenSSL recenti, `openssl list -cipher-algorithms` (`openssl list-cipher-algorithms` per le versioni precedenti di OpenSSL) mostrerà gli algoritmi cipher disponibili.

La `key` è la raw key utilizzata dall'`algorithm` e `iv` è un [vettore di inizializzazione](https://en.wikipedia.org/wiki/Initialization_vector). Entrambi gli argomenti devono essere delle stringhe con codifica `'utf8'`, dei [Buffers][`Buffer`], dei `TypedArray`, o dei `DataView`. Se il cipher non ha bisogno di un vettore di inizializzazione, `iv` potrebbe essere `null`.

I vettori di inizializzazione dovrebbero essere imprevedibili e unici; idealmente, saranno crittograficamente casuali. Non devono essere segreti: gli IV vengono in genere aggiunti ai messaggi ciphertext non crittografati. Potrebbe sembrare contraddittorio che un qualcosa debba essere imprevedibile e unico, ma non debba essere segreto; è importante ricordare che un aggressore non deve essere in grado di prevedere in anticipo quale possa essere un dato IV.

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
- `generator` {number | string | Buffer | TypedArray | DataView} **Default:** `2`
- `generatorEncoding` {string}

Crea un `DiffieHellman` key exchange object utilizzando il `prime` fornito e un `generator` specifico opzionale.

L'argomento `generator` può essere un numero, una stringa o un [`Buffer`][]. Se `generator` non viene specificato, viene utilizzato il valore `2`.

Gli argomenti `primeEncoding` e `generatorEncoding` possono essere `'latin1'`, `'hex'`, o `'base64'`.

Se viene specificato il `primeEncoding`, `prime` dovrebbe essere una stringa; in caso contrario, dovrebbe essere un [`Buffer`][], un `TypedArray`, o un `DataView`.

Se viene specificato il `generatorEncoding`, `generator` dovrebbe essere una stringa; in caso contrario dovrebbe essere un numero, un [`Buffer`][], un `TypedArray`, o un `DataView`.

### crypto.createDiffieHellman(primeLength[, generator])

<!-- YAML
added: v0.5.0
-->

- `primeLength` {number}
- `generator` {number | string | Buffer | TypedArray | DataView} **Default:** `2`

Crea un `DiffieHellman` key exchange object e genera un prime di `primeLength` bit utilizzando un `generator` numerico specifico opzionale. Se `generator` non viene specificato, viene utilizzato il valore `2`.

### crypto.createECDH(curveName)

<!-- YAML
added: v0.11.14
-->

- `curveName` {string}

Crea un Elliptic Curve Diffie-Hellman (`ECDH`) key exchange object utilizzando una curva predefinita specificata dalla stringa `curveName`. Utilizza [`crypto.getCurves()`][] per ottenere un elenco di nomi di curve disponibili. Nelle versioni OpenSSL recenti, `openssl ecparam -list_curves` mostrerà anche il nome e la descrizione di ciascuna curva ellittica disponibile.

### crypto.createHash(algorithm[, options])

<!-- YAML
added: v0.1.92
-->

- `algorithm` {string}
- `options` {Object} [`stream.transform` options][]
- Restituisce: {Hash}

Crea e restituisce un `Hash` object che può essere utilizzato per generare degli hash digest tramite l'`algorithm` specificato. L'argomento `options` opzionale controlla il comportamento dello stream.

L'`algorithm` dipende dagli algoritmi disponibili supportati dalla versione OpenSSL sulla piattaforma. Alcuni esempi sono `'sha256'`, `'sha512'`, ecc. Nelle versioni OpenSSl recenti, `openssl list -digest-algorithms` (`openssl list-message-digest-algorithms` per le versioni precedenti di OpenSSL) mostrerà gli algoritmi digest disponibili.

Esempio: generazione della somma sha256 di un file

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
- Restituisce: {Hmac}

Crea e restituisce un `Hmac` object che utilizza l'`algorithm` e la `key` specificati. L'argomento `options` opzionale controlla il comportamento dello stream.

L'`algorithm` dipende dagli algoritmi disponibili supportati dalla versione OpenSSL sulla piattaforma. Alcuni esempi sono `'sha256'`, `'sha512'`, ecc. Nelle versioni OpenSSl recenti, `openssl list -digest-algorithms` (`openssl list-message-digest-algorithms` per le versioni precedenti di OpenSSL) mostrerà gli algoritmi digest disponibili.

La `key` è la chiave HMAC utilizzata per generare l'hash crittografico HMAC.

Esempio: generazione dell'HMAC sha256 di un file

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
- Restituisce: {Sign}

Crea e restituisce un `Sign` object che utilizza l'`algorithm` specificato. Utilizza [`crypto.getHashes()`][] per ottenere un array di nomi degli algoritmi per la firma disponibili. L'argomento `options` opzionale controlla il comportamento di `stream.Writable`.

### crypto.createVerify(algorithm[, options])

<!-- YAML
added: v0.1.92
-->

- `algorithm` {string}
- `options` {Object} [`stream.Writable` options][]
- Restituisce: {Verify}

Crea e restituisce un `Verify` object che utilizza l'algoritmo specificato. Utilizza [`crypto.getHashes()`][] per ottenere un array di nomi degli algoritmi per la firma disponibili. L'argomento `options` opzionale controlla il comportamento di `stream.Writable`.

### crypto.getCiphers()

<!-- YAML
added: v0.9.3
-->

- Restituisce: {string[]} Un array con i nomi degli algoritmi cipher supportati.

Esempio:

```js
const ciphers = crypto.getCiphers();
console.log(ciphers); // ['aes-128-cbc', 'aes-128-ccm', ...]
```

### crypto.getCurves()

<!-- YAML
added: v2.3.0
-->

- Restituisce: {string[]} Un array con i nomi delle curve ellittiche supportate.

Esempio:

```js
const curves = crypto.getCurves();
console.log(curves); // ['Oakley-EC2N-3', 'Oakley-EC2N-4', ...]
```

### crypto.getDiffieHellman(groupName)

<!-- YAML
added: v0.7.5
-->

- `groupName` {string}
- Restituisce: {Object}

Crea un `DiffieHellman` key exchange object predefinito. I gruppi supportati sono: `'modp1'`, `'modp2'`, `'modp5'` (definiti nel documento [RFC 2412](https://www.rfc-editor.org/rfc/rfc2412.txt), ma vedi anche gli [Avvertimenti](#crypto_support_for_weak_or_compromised_algorithms)) e `'modp14'`, `'modp15'`, `'modp16'`, `'modp17'`, `'modp18'` (definiti nel documento [RFC 3526](https://www.rfc-editor.org/rfc/rfc3526.txt)). L'object restituito simula l'interfaccia degli object creati da [`crypto.createDiffieHellman()`][], ma non consente di cambiare le chiavi (con [`diffieHellman.setPublicKey()`][] per esempio). Il vantaggio dell'utilizzo di questo metodo è che le parti non devono generare né scambiare preventivamente un modulo di gruppo, risparmiando tempo per il processore e la comunicazione.

Esempio (ottenendo una chiave segreta condivisa):

```js
const crypto = require('crypto');
const alice = crypto.getDiffieHellman('modp14');
const bob = crypto.getDiffieHellman('modp14');

alice.generateKeys();
bob.generateKeys();

const aliceSecret = alice.computeSecret(bob.getPublicKey(), null, 'hex');
const bobSecret = bob.computeSecret(alice.getPublicKey(), null, 'hex');

/* aliceSecret e bobSecret dovrebbero essere uguali */
console.log(aliceSecret === bobSecret);
```

### crypto.getFips()

<!-- YAML
added: v10.0.0
-->

- Restituisce: {boolean} `true` se e solo se è attualmente in uso un provider crittografico conforme alle norme FIPS.

### crypto.getHashes()

<!-- YAML
added: v0.9.3
-->

- Restituisce: {string[]} Un array dei nomi degli algoritmi hash supportati, come ad esempio `'RSA-SHA256'`.

Esempio:

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

- `password` {string|Buffer|TypedArray}
- `salt` {string|Buffer|TypedArray}
- `iterations` {number}
- `keylen` {number}
- `digest` {string}
- `callback` {Function} 
  - `err` {Error}
  - `derivedKey` {Buffer}

Fornisce un'implementazione asincrona della Password-Based Key Derivation Function 2 (PBKDF2). Viene applicato un algoritmo dell'HMAC digest selezionato specificato da `digest` per derivare una chiave della lunghezza di byte richiesta (`keylen`) dalla `password`, dal `salt` e dalle `iterations`.

La funzione `callback` fornita viene chiamata con due argomenti: `err` e `derivedKey`. Se si verifica un errore durante la derivazione della chiave, verrà impostato `err`; in caso contrario `err` sarà `null`. Di default, la `derivedKey` generata correttamente verrà passata al callback come un [`Buffer`][]. Se uno qualsiasi degli argomenti d'input specifica valori o tipi non validi verrà generato un errore.

L'argomento `iterations` dev'essere un numero impostato con il valore più alto possibile. Maggiore è il numero di iterazioni, più sicura sarà la chiave derivata, ma sarà necessario più tempo per completarla.

Anche il `salt` dovrebbe essere il più unico possibile. It is recommended that the salts are random and their lengths are at least 16 bytes. See [NIST SP 800-132](http://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-132.pdf) for details.

Example:

```js
const crypto = require('crypto');
crypto.pbkdf2('secret', 'salt', 100000, 64, 'sha512', (err, derivedKey) => {
  if (err) throw err;
  console.log(derivedKey.toString('hex'));  // '3745e48...08d59ae'
});
```

The `crypto.DEFAULT_ENCODING` property can be used to change the way the `derivedKey` is passed to the callback. This property, however, has been deprecated and use should be avoided.

```js
const crypto = require('crypto');
crypto.DEFAULT_ENCODING = 'hex';
crypto.pbkdf2('secret', 'salt', 100000, 512, 'sha512', (err, derivedKey) => {
  if (err) throw err;
  console.log(derivedKey);  // '3745e48...aa39b34'
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

- `password` {string|Buffer|TypedArray}
- `salt` {string|Buffer|TypedArray}
- `iterations` {number}
- `keylen` {number}
- `digest` {string}
- Returns: {Buffer}

Provides a synchronous Password-Based Key Derivation Function 2 (PBKDF2) implementation. A selected HMAC digest algorithm specified by `digest` is applied to derive a key of the requested byte length (`keylen`) from the `password`, `salt` and `iterations`.

If an error occurs an `Error` will be thrown, otherwise the derived key will be returned as a [`Buffer`][].

The `iterations` argument must be a number set as high as possible. The higher the number of iterations, the more secure the derived key will be, but will take a longer amount of time to complete.

The `salt` should also be as unique as possible. It is recommended that the salts are random and their lengths are at least 16 bytes. See [NIST SP 800-132](http://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-132.pdf) for details.

Example:

```js
const crypto = require('crypto');
const key = crypto.pbkdf2Sync('secret', 'salt', 100000, 64, 'sha512');
console.log(key.toString('hex'));  // '3745e48...08d59ae'
```

The `crypto.DEFAULT_ENCODING` property may be used to change the way the `derivedKey` is returned. This property, however, has been deprecated and use should be avoided.

```js
const crypto = require('crypto');
crypto.DEFAULT_ENCODING = 'hex';
const key = crypto.pbkdf2Sync('secret', 'salt', 100000, 512, 'sha512');
console.log(key);  // '3745e48...aa39b34'
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
changes:

  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/16454
    description: Passing `null` as the `callback` argument now throws
                 `ERR_INVALID_CALLBACK`.
-->

- `size` {number}
- `callback` {Function} 
  - `err` {Error}
  - `buf` {Buffer}
- Returns: {Buffer} if the `callback` function is not provided.

Generates cryptographically strong pseudo-random data. The `size` argument is a number indicating the number of bytes to generate.

If a `callback` function is provided, the bytes are generated asynchronously and the `callback` function is invoked with two arguments: `err` and `buf`. If an error occurs, `err` will be an `Error` object; otherwise it is `null`. The `buf` argument is a [`Buffer`][] containing the generated bytes.

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

The asynchronous version of `crypto.randomBytes()` is carried out in a single threadpool request. To minimize threadpool task length variation, partition large `randomBytes` requests when doing so as part of fulfilling a client request.

### crypto.randomFillSync(buffer\[, offset\]\[, size\])

<!-- YAML
added: v7.10.0
changes:

  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/15231
    description: The `buffer` argument may be any `TypedArray` or `DataView`.
-->

- `buffer` {Buffer|TypedArray|DataView} Must be supplied.
- `offset` {number} **Default:** `0`
- `size` {number} **Default:** `buffer.length - offset`
- Returns: {Buffer}

Synchronous version of [`crypto.randomFill()`][].

```js
const buf = Buffer.alloc(10);
console.log(crypto.randomFillSync(buf).toString('hex'));

crypto.randomFillSync(buf, 5);
console.log(buf.toString('hex'));

// The above is equivalent to the following:
crypto.randomFillSync(buf, 5, 5);
console.log(buf.toString('hex'));
```

Any `TypedArray` or `DataView` instance may be passed as `buffer`.

```js
const a = new Uint32Array(10);
console.log(crypto.randomFillSync(a).toString('hex'));

const b = new Float64Array(10);
console.log(crypto.randomFillSync(a).toString('hex'));

const c = new DataView(new ArrayBuffer(10));
console.log(crypto.randomFillSync(a).toString('hex'));
```

### crypto.randomFill(buffer\[, offset\]\[, size\], callback)

<!-- YAML
added: v7.10.0
changes:

  - version: v9.0.0
    pr-url: https://github.com/nodejs/node/pull/15231
    description: The `buffer` argument may be any `TypedArray` or `DataView`.
-->

- `buffer` {Buffer|TypedArray|DataView} Must be supplied.
- `offset` {number} **Default:** `0`
- `size` {number} **Default:** `buffer.length - offset`
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

Any `TypedArray` or `DataView` instance may be passed as `buffer`.

```js
const a = new Uint32Array(10);
crypto.randomFill(a, (err, buf) => {
  if (err) throw err;
  console.log(buf.toString('hex'));
});

const b = new Float64Array(10);
crypto.randomFill(b, (err, buf) => {
  if (err) throw err;
  console.log(buf.toString('hex'));
});

const c = new DataView(new ArrayBuffer(10));
crypto.randomFill(c, (err, buf) => {
  if (err) throw err;
  console.log(buf.toString('hex'));
});
```

Note that this API uses libuv's threadpool, which can have surprising and negative performance implications for some applications, see the [`UV_THREADPOOL_SIZE`][] documentation for more information.

The asynchronous version of `crypto.randomFill()` is carried out in a single threadpool request. To minimize threadpool task length variation, partition large `randomFill` requests when doing so as part of fulfilling a client request.

### crypto.setEngine(engine[, flags])

<!-- YAML
added: v0.11.11
-->

- `engine` {string}
- `flags` {crypto.constants} **Default:** `crypto.constants.ENGINE_METHOD_ALL`

Load and set the `engine` for some or all OpenSSL functions (selected by flags).

`engine` could be either an id or a path to the engine's shared library.

The optional `flags` argument uses `ENGINE_METHOD_ALL` by default. The `flags` is a bit field taking one of or a mix of the following flags (defined in `crypto.constants`):

- `crypto.constants.ENGINE_METHOD_RSA`
- `crypto.constants.ENGINE_METHOD_DSA`
- `crypto.constants.ENGINE_METHOD_DH`
- `crypto.constants.ENGINE_METHOD_RAND`
- `crypto.constants.ENGINE_METHOD_EC`
- `crypto.constants.ENGINE_METHOD_CIPHERS`
- `crypto.constants.ENGINE_METHOD_DIGESTS`
- `crypto.constants.ENGINE_METHOD_PKEY_METHS`
- `crypto.constants.ENGINE_METHOD_PKEY_ASN1_METHS`
- `crypto.constants.ENGINE_METHOD_ALL`
- `crypto.constants.ENGINE_METHOD_NONE`

The flags below are deprecated in OpenSSL-1.1.0.

- `crypto.constants.ENGINE_METHOD_ECDH`
- `crypto.constants.ENGINE_METHOD_ECDSA`
- `crypto.constants.ENGINE_METHOD_STORE`

### crypto.setFips(bool)

<!-- YAML
added: v10.0.0
-->

- `bool` {boolean} `true` to enable FIPS mode.

Enables the FIPS compliant crypto provider in a FIPS-enabled Node.js build. Throws an error if FIPS mode is not available.

### crypto.timingSafeEqual(a, b)

<!-- YAML
added: v6.6.0
-->

- `a` {Buffer | TypedArray | DataView}
- `b` {Buffer | TypedArray | DataView}
- Returns: {boolean}

This function is based on a constant-time algorithm. Returns true if `a` is equal to `b`, without leaking timing information that would allow an attacker to guess one of the values. This is suitable for comparing HMAC digests or secret values like authentication cookies or [capability urls](https://www.w3.org/TR/capability-urls/).

`a` and `b` must both be `Buffer`s, `TypedArray`s, or `DataView`s, and they must have the same length.

Use of `crypto.timingSafeEqual` does not guarantee that the *surrounding* code is timing-safe. Care should be taken to ensure that the surrounding code does not introduce timing vulnerabilities.

## Notes

### Legacy Streams API (pre Node.js v0.10)

The Crypto module was added to Node.js before there was the concept of a unified Stream API, and before there were [`Buffer`][] objects for handling binary data. As such, the many of the `crypto` defined classes have methods not typically found on other Node.js classes that implement the [streams](stream.html) API (e.g. `update()`, `final()`, or `digest()`). Also, many methods accepted and returned `'latin1'` encoded strings by default rather than `Buffer`s. This default was changed after Node.js v0.8 to use [`Buffer`][] objects by default instead.

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

### CCM mode

CCM is one of the two supported [AEAD algorithms](https://en.wikipedia.org/wiki/Authenticated_encryption). Applications which use this mode must adhere to certain restrictions when using the cipher API:

- The authentication tag length must be specified during cipher creation by setting the `authTagLength` option and must be one of 4, 6, 8, 10, 12, 14 or 16 bytes.
- The length of the initialization vector (nonce) `N` must be between 7 and 13 bytes (`7 ≤ N ≤ 13`).
- The length of the plaintext is limited to `2 ** (8 * (15 - N))` bytes.
- When decrypting, the authentication tag must be set via `setAuthTag()` before specifying additional authenticated data and / or calling `update()`. Otherwise, decryption will fail and `final()` will throw an error in compliance with section 2.6 of [RFC 3610](https://www.rfc-editor.org/rfc/rfc3610.txt).
- Using stream methods such as `write(data)`, `end(data)` or `pipe()` in CCM mode might fail as CCM cannot handle more than one chunk of data per instance.
- When passing additional authenticated data (AAD), the length of the actual message in bytes must be passed to `setAAD()` via the `plaintextLength` option. This is not necessary if no AAD is used.
- As CCM processes the whole message at once, `update()` can only be called once.
- Even though calling `update()` is sufficient to encrypt / decrypt the message, applications *must* call `final()` to compute and / or verify the authentication tag.

```js
const crypto = require('crypto');

const key = 'keykeykeykeykeykeykeykey';
const nonce = crypto.randomBytes(12);

const aad = Buffer.from('0123456789', 'hex');

const cipher = crypto.createCipheriv('aes-192-ccm', key, nonce, {
  authTagLength: 16
});
const plaintext = 'Hello world';
cipher.setAAD(aad, {
  plaintextLength: Buffer.byteLength(plaintext)
});
const ciphertext = cipher.update(plaintext, 'utf8');
cipher.final();
const tag = cipher.getAuthTag();

// Now transmit { ciphertext, nonce, tag }.

const decipher = crypto.createDecipheriv('aes-192-ccm', key, nonce, {
  authTagLength: 16
});
decipher.setAuthTag(tag);
decipher.setAAD(aad, {
  plaintextLength: ciphertext.length
});
const receivedPlaintext = decipher.update(ciphertext, null, 'utf8');

try {
  decipher.final();
} catch (err) {
  console.error('Authentication failed!');
}

console.log(receivedPlaintext);
```

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
    <td><code>ENGINE_METHOD_EC</code></td>
    <td>Limit engine usage to EC</td>
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
