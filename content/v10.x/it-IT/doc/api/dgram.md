# Socket UDP / Datagram

<!--introduced_in=v0.10.0-->

> Stabilità: 2 - Stabile

<!-- name=dgram -->

Il modulo `dgram` fornisce un'implementazione dei socket UDP Datagram.

```js
const dgram = require('dgram');
const server = dgram.createSocket('udp4');

server.on('error', (err) => {
  console.log(`server error:\n${err.stack}`);
  server.close();
});

server.on('message', (msg, rinfo) => {
  console.log(`server got: ${msg} from ${rinfo.address}:${rinfo.port}`);
});

server.on('listening', () => {
  const address = server.address();
  console.log(`server listening ${address.address}:${address.port}`);
});

server.bind(41234);
// server listening 0.0.0.0:41234
```

## Class: dgram.Socket

<!-- YAML
added: v0.1.99
-->

Il `dgram.Socket` object è un [`EventEmitter`][] che incapsula la funzionalità del datagram.

Le nuove istanze di `dgram.Socket` sono create utilizzando [`dgram.createSocket()`][]. La parola chiave `new` non dev'essere utilizzata per creare istanze `dgram.Socket`.

### Event: 'close'

<!-- YAML
added: v0.1.99
-->

L'evento `'close'` viene emesso dopo che un socket è stato chiuso con [`close()`][]. Una volta attivato, non verrà emesso nessun nuovo evento `'message'` su questo socket.

### Event: 'error'

<!-- YAML
added: v0.1.99
-->

* `exception` {Error}

L'evento `'error'` viene emesso ogni volta che si verifica un errore. La funzione dell'event handler riceve un singolo `Error` object.

### Event: 'listening'

<!-- YAML
added: v0.1.99
-->

L'evento `'listening'` viene emesso ogni volta che un socket inizia ad ascoltare (listening) i messaggi del datagram. Ciò si verifica non appena vengono creati i socket UDP.

### Event: 'message'

<!-- YAML
added: v0.1.99
-->

L'evento `'message'` viene emesso quando un nuovo datagram è disponibile su un socket. La funzione dell'event handler riceve due argomenti: `msg` e `rinfo`.

* `msg` {Buffer} Il messaggio.
* `rinfo` {Object} Informazioni sull'indirizzo remoto. 
  * `address` {string} L'indirizzo del mittente.
  * `family` {string} La famiglia di indirizzi (`'IPv4'` o `'IPv6'`).
  * `port` {number} La porta del mittente.
  * `size` {number} La dimensione del messaggio.

### socket.addMembership(multicastAddress[, multicastInterface])

<!-- YAML
added: v0.6.9
-->

* `multicastAddress` {string}
* `multicastInterface` {string}

Indica al kernel di unirsi a un gruppo multicast sul `multicastAddress` e sul `multicastInterface` specificati utilizzando l'opzione socket `IP_ADD_MEMBERSHIP`. Se l'argomento `multicastInterface` non è specificato, il sistema operativo sceglierà un'interfaccia e ne aggiungerà l'iscrizione. Per aggiungere l'iscrizione a ogni interfaccia disponibile, chiama più volte `addMembership`, una volta per ogni interfaccia.

### socket.address()

<!-- YAML
added: v0.1.99
-->

* Restituisce: {Object}

Restituisce un object contenente le informazioni sull'indirizzo per un socket. Per i socket UDP, quest'object conterrà le proprietà `address`, `family` e `port`.

### socket.bind(\[port\]\[, address\][, callback])

<!-- YAML
added: v0.1.99
-->

* `port` {integer}
* `address` {string}
* `callback` {Function} senza parametri. Chiamato quando il binding (collegamento) è completo.

Per i socket UDP, fa sì che il `dgram.Socket` ascolti i messaggi dei datagram su una `port` denominata e sull'`address` facoltativo. Se `port` non è specificato oppure corrisponde a `0`, il sistema operativo tenterà il binding (collegamento) ad una porta casuale. Se `address` non è specificato, il sistema operativo tenterà di eseguire il listening (ascolto) su tutti gli indirizzi. Una volta completato il binding, viene emesso un evento `'listening'` e viene chiamata la funzione `callback` opzionale.

Da notare che specificare un listener di eventi `'listening'` e passare un `callback` al metodo `socket.bind()` non è dannoso ma neanche utile.

Un socket datagram che ha completato il binding mantiene il processo Node.js in esecuzione per ricevere i messaggi del datagram.

Se il binding fallisce, viene generato un evento `'error'`. In rari casi (ad esempio quando si tenta di eseguire il binding con un socket chiuso), potrebbe essere generato un [`Error`][].

Esempio di un UDP server listening sulla porta 41234:

```js
const dgram = require('dgram');
const server = dgram.createSocket('udp4');

server.on('error', (err) => {
  console.log(`server error:\n${err.stack}`);
  server.close();
});

server.on('message', (msg, rinfo) => {
  console.log(`server got: ${msg} from ${rinfo.address}:${rinfo.port}`);
});

server.on('listening', () => {
  const address = server.address();
  console.log(`server listening ${address.address}:${address.port}`);
});

server.bind(41234);
// server listening 0.0.0.0:41234
```

### socket.bind(options[, callback])

<!-- YAML
added: v0.11.14
-->

* `options` {Object} Obbligatorio. Supporta le seguenti proprietà: 
  * `port` {integer}
  * `address` {string}
  * `exclusive` {boolean}
* `callback` {Function}

Per i socket UDP, fa sì che `dgram.Socket` ascolti i messaggi dei datagram su una `port` denominata e sull'`address` facoltativo che vengono passati come proprietà di un `options` object passato come primo argomento. Se `port` non è specificato oppure corrisponde a `0`, il sistema operativo tenterà il binding (collegamento) a una porta casuale. Se `address` non è specificato, il sistema operativo tenterà di eseguire il listening (ascolto) su tutti gli indirizzi. Una volta completato il binding, viene emesso un evento `'listening'` e viene chiamata la funzione `callback` opzionale.

Da notare che specificare un listener di eventi `'listening'` e passare un `callback` al metodo `socket.bind()` non è dannoso ma neanche utile.

L'`options` object può contenere una proprietà `exclusive` aggiuntiva che viene utilizzata quando si utilizzano i `dgram.Socket` object con il modulo [`cluster`]. Quando `exclusive` è impostato su `false` (valore predefinito), gli worker del cluster utilizzeranno lo stesso socket handle sottostante permettendo la condivisione dei compiti di gestione delle connessioni. Tuttavia quando `exclusive` è `true`, l'handle non è condiviso e il tentativo di condivisione della porta genera un errore.

Un socket datagram che ha completato il binding mantiene il processo Node.js in esecuzione per ricevere i messaggi del datagram.

Se il binding fallisce, viene generato un evento `'error'`. In rari casi (ad esempio quando si tenta di eseguire il binding con un socket chiuso), potrebbe essere generato un [`Error`][].

Di seguito viene mostrato un esempio di socket che esegue il listening su una porta esclusiva.

```js
socket.bind({
  address: 'localhost',
  port: 8000,
  exclusive: true
});
```

### socket.close([callback])

<!-- YAML
added: v0.1.99
-->

Chiude il socket sottostante e interrompe il listening dei dati su di esso. Se viene fornito un callback, quest'ultimo viene aggiunto come listener per l'evento [`'close'`][].

### socket.dropMembership(multicastAddress[, multicastInterface])

<!-- YAML
added: v0.6.9
-->

* `multicastAddress` {string}
* `multicastInterface` {string}

Dà istruzioni al kernel di lasciare un gruppo multicast su `multicastAddress` utilizzando l'opzione socket `IP_DROP_MEMBERSHIP`. Questo metodo viene chiamato automaticamente dal kernel quando viene chiuso il socket o termina il processo, quindi la maggior parte delle app non avrà mai motivo di chiamarlo.

Se `multicastInterface` non è specificato, il sistema operativo tenterà di eliminare l'iscrizione su tutte le interfacce valide.

### socket.getRecvBufferSize()

<!-- YAML
added: v8.7.0
-->

* Restituisce: {number} il socket `SO_RCVBUF` riceve la dimensione del buffer in byte.

### socket.getSendBufferSize()

<!-- YAML
added: v8.7.0
-->

* Restituisce: {number} il socket `SO_SNDBUF` invia la dimensione del buffer in byte.

### socket.ref()

<!-- YAML
added: v0.9.1
-->

Di default, il binding di un socket causerà il blocco della conclusione del processo Node.js finché il socket è aperto. Il metodo `socket.unref()` può essere utilizzato per escludere il socket dal reference count che mantiene attivo il processo Node.js. Il metodo `socket.ref()` aggiunge il socket al reference count e ripristina il comportamento predefinito.

Chiamare `socket.ref()` più volte non avrà nessun effetto aggiuntivo.

Il metodo `socket.ref()` restituisce un riferimento al socket in modo che le chiamate possano essere concatenate.

### socket.send(msg, \[offset, length,] port [, address\] \[, callback\])

<!-- YAML
added: v0.1.99
changes:

  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/11985
    description: The `msg` parameter can be an `Uint8Array` now.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10473
    description: The `address` parameter is always optional now.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5929
    description: On success, `callback` will now be called with an `error`
                 argument of `null` rather than `0`.
  - version: v5.7.0
    pr-url: https://github.com/nodejs/node/pull/4374
    description: The `msg` parameter can be an array now. Also, the `offset`
                 and `length` parameters are optional now.
-->

* `msg` {Buffer|Uint8Array|string|Array} Messaggio da inviare.
* `offset` {integer} Offset nel buffer in cui inizia il messaggio.
* `length` {integer} Numero di byte nel messaggio.
* `port` {integer} Porto di destinazione.
* `address` {string} Hostname di destinazione o indirizzo IP.
* `callback` {Function} Chiamato quando il messaggio è stato inviato.

Trasmette un datagram sul socket. La `port` e l'`address` di destinazione devono essere specificate.

L'argomento `msg` contiene il messaggio da inviare. A seconda del tipo, può essere applicato un comportamento diverso. Se `msg` è un `Buffer` o un `Uint8Array`, i valori `offset` e `length` specificano rispettivamente l'offset all'interno del `Buffer` dove inizia il messaggio e il numero di byte all'interno del messaggio. Se `msg` è una `String`, allora viene automaticamente convertito in un `Buffer` con codifica `'utf8'`. Con i messaggi contenenti caratteri multibyte, `offset` e `length` verranno calcolati rispetto alla [lunghezza del byte](buffer.html#buffer_class_method_buffer_bytelength_string_encoding) e non alla posizione del carattere. Se `msg` è un array, `offset` e `length` non devono essere specificati.

L'argomento `address` è una stringa. Se il valore di `address` è un hostname, verrà utilizzato il DNS per risolvere l'indirizzo dell'host. Se `address` non è fornito oppure corrisponde a false, di default verrà utilizzato `'127.0.0.1'` (per i socket `udp4`) oppure `'::1'` (per i socket `udp6`).

Se il socket non è stato precedentemente sottoposto al binding con una chiamata a `bind`, gli viene assegnato un numero di porta casuale e viene sottoposto al binding con l'indirizzo di "tutte le interfacce" (`'0.0.0.0'` per i socket `udp4`, `'::0'` per i socket `udp6`.)

Potrebbe essere specificata una funzione `callback` opzionale come un modo per segnalare gli errori del DNS o per determinare quando è sicuro il riutilizzo del `buf` object. Da notare che le ricerche del DNS ritardano il tempo di invio del ciclo di eventi Node.js (event loop) di almeno un secondo.

L'unico modo per sapere con certezza che il datagram sia stato inviato è l'utilizzo di un `callback`. Se si verifica un errore e viene fornito un `callback`, l'errore verrà passato al `callback` come primo argomento. Se non viene fornito un `callback`, l'errore viene emesso come un evento `'error'` sul `socket` object.

Offset e length sono facoltativi, ma *devono* essere impostati se vengono utilizzati entrambi. Sono supportati solo quando il primo argomento è un `Buffer` o un `Uint8Array`.

Esempio d'invio di un pacchetto UDP a una porta su `localhost`;

```js
const dgram = require('dgram');
const message = Buffer.from('Some bytes');
const client = dgram.createSocket('udp4');
client.send(message, 41234, 'localhost', (err) => {
  client.close();
});
```

Esempio d'invio di un pacchetto UDP composto da più buffer su una porta su `127.0.0.1`;

```js
const dgram = require('dgram');
const buf1 = Buffer.from('Some ');
const buf2 = Buffer.from('bytes');
const client = dgram.createSocket('udp4');
client.send([buf1, buf2], 41234, (err) => {
  client.close();
});
```

L'invio di più buffer potrebbe essere più veloce o più lento a seconda dell'applicazione e del sistema operativo. È importante eseguire i benchmark per determinare la strategia ottimale per ogni singolo caso. Generalmente parlando, tuttavia, l'invio di più buffer è più veloce.

**Una Nota sulla dimensione del UDP Datagram**

La dimensione massima di un datagram `IPv4/v6` dipende dal `MTU` (*Maximum Transmission Unit*) e dalla dimensione del campo `Payload Length`.

* Il campo `Payload Length` è di larghezza `16 bits`, il che significa che un normale payload supera i 64 mila ottetti *inclusi* l'intestazione e i dati d'internet (65,507 byte = 65,535 - 8 byte intestazione UDP - 20 byte intestazione IP); questo è generalmente vero per le interfacce di loopback, ma i messaggi datagram così lunghi sono poco pratici per la maggior parte degli host e delle reti.

* L'`MTU` è la dimensione più grande che una determinata tecnologia a livello di collegamento può supportare per i messaggi datagram. Per ogni collegamento, `IPv4` richiede un `MTU` minimo di `68` ottetti, mentre l'`MTU` raccomandato per IPv4 è di `576` (in genere raccomandato come `MTU` per le applicazioni di tipo dial-up), sia che arrivino interi o in frammenti.
  
  Per `IPv6`, l'`MTU` minimo è di `1280` ottetti, tuttavia, la dimensione obbligatoria del buffer di riassemblaggio del frammento è di `1500` ottetti. Il valore di `68` ottetti è molto piccolo, poiché la maggior parte delle tecnologie attuali a livello di collegamento, come l'Ethernet, hanno un `MTU` minimo di `1500`.

È impossibile sapere in anticipo l'MTU di ciascun collegamento attraverso il quale un pacchetto potrebbe viaggiare. L'invio di un datagram maggiore del receiver `MTU` non funzionerà perché il pacchetto verrà rilasciato automaticamente senza informare la sorgente del fatto che i dati non hanno raggiunto il destinatario previsto.

### socket.setBroadcast(flag)

<!-- YAML
added: v0.6.9
-->

* `flag` {boolean}

Sets or clears the `SO_BROADCAST` socket option. When set to `true`, UDP packets may be sent to a local interface's broadcast address.

### socket.setMulticastInterface(multicastInterface)

<!-- YAML
added: v8.6.0
-->

* `multicastInterface` {string}

*Note: All references to scope in this section are referring to [IPv6 Zone Indices](https://en.wikipedia.org/wiki/IPv6_address#Scoped_literal_IPv6_addresses), which are defined by [RFC 4007](https://tools.ietf.org/html/rfc4007). In string form, an IP with a scope index is written as `'IP%scope'` where scope is an interface name or interface number.*

Sets the default outgoing multicast interface of the socket to a chosen interface or back to system interface selection. The `multicastInterface` must be a valid string representation of an IP from the socket's family.

For IPv4 sockets, this should be the IP configured for the desired physical interface. All packets sent to multicast on the socket will be sent on the interface determined by the most recent successful use of this call.

For IPv6 sockets, `multicastInterface` should include a scope to indicate the interface as in the examples that follow. In IPv6, individual `send` calls can also use explicit scope in addresses, so only packets sent to a multicast address without specifying an explicit scope are affected by the most recent successful use of this call.

#### Examples: IPv6 Outgoing Multicast Interface

On most systems, where scope format uses the interface name:

```js
const socket = dgram.createSocket('udp6');

socket.bind(1234, () => {
  socket.setMulticastInterface('::%eth1');
});
```

On Windows, where scope format uses an interface number:

```js
const socket = dgram.createSocket('udp6');

socket.bind(1234, () => {
  socket.setMulticastInterface('::%2');
});
```

#### Example: IPv4 Outgoing Multicast Interface

All systems use an IP of the host on the desired physical interface:

```js
const socket = dgram.createSocket('udp4');

socket.bind(1234, () => {
  socket.setMulticastInterface('10.0.0.2');
});
```

#### Call Results

A call on a socket that is not ready to send or no longer open may throw a *Not running* [`Error`][].

If `multicastInterface` can not be parsed into an IP then an *EINVAL* [`System Error`][] is thrown.

On IPv4, if `multicastInterface` is a valid address but does not match any interface, or if the address does not match the family then a [`System Error`][] such as `EADDRNOTAVAIL` or `EPROTONOSUP` is thrown.

On IPv6, most errors with specifying or omitting scope will result in the socket continuing to use (or returning to) the system's default interface selection.

A socket's address family's ANY address (IPv4 `'0.0.0.0'` or IPv6 `'::'`) can be used to return control of the sockets default outgoing interface to the system for future multicast packets.

### socket.setMulticastLoopback(flag)

<!-- YAML
added: v0.3.8
-->

* `flag` {boolean}

Sets or clears the `IP_MULTICAST_LOOP` socket option. When set to `true`, multicast packets will also be received on the local interface.

### socket.setMulticastTTL(ttl)

<!-- YAML
added: v0.3.8
-->

* `ttl` {integer}

Sets the `IP_MULTICAST_TTL` socket option. While TTL generally stands for "Time to Live", in this context it specifies the number of IP hops that a packet is allowed to travel through, specifically for multicast traffic. Each router or gateway that forwards a packet decrements the TTL. If the TTL is decremented to 0 by a router, it will not be forwarded.

The argument passed to `socket.setMulticastTTL()` is a number of hops between 0 and 255. The default on most systems is `1` but can vary.

### socket.setRecvBufferSize(size)

<!-- YAML
added: v8.7.0
-->

* `size` {integer}

Sets the `SO_RCVBUF` socket option. Sets the maximum socket receive buffer in bytes.

### socket.setSendBufferSize(size)

<!-- YAML
added: v8.7.0
-->

* `size` {integer}

Sets the `SO_SNDBUF` socket option. Sets the maximum socket send buffer in bytes.

### socket.setTTL(ttl)

<!-- YAML
added: v0.1.101
-->

* `ttl` {integer}

Sets the `IP_TTL` socket option. While TTL generally stands for "Time to Live", in this context it specifies the number of IP hops that a packet is allowed to travel through. Each router or gateway that forwards a packet decrements the TTL. If the TTL is decremented to 0 by a router, it will not be forwarded. Changing TTL values is typically done for network probes or when multicasting.

The argument to `socket.setTTL()` is a number of hops between 1 and 255. The default on most systems is 64 but can vary.

### socket.unref()

<!-- YAML
added: v0.9.1
-->

By default, binding a socket will cause it to block the Node.js process from exiting as long as the socket is open. The `socket.unref()` method can be used to exclude the socket from the reference counting that keeps the Node.js process active, allowing the process to exit even if the socket is still listening.

Calling `socket.unref()` multiple times will have no addition effect.

The `socket.unref()` method returns a reference to the socket so calls can be chained.

### Change to asynchronous `socket.bind()` behavior

As of Node.js v0.10, [`dgram.Socket#bind()`][] changed to an asynchronous execution model. Legacy code that assumes synchronous behavior, as in the following example:

```js
const s = dgram.createSocket('udp4');
s.bind(1234);
s.addMembership('224.0.0.114');
```

Must be changed to pass a callback function to the [`dgram.Socket#bind()`][] function:

```js
const s = dgram.createSocket('udp4');
s.bind(1234, () => {
  s.addMembership('224.0.0.114');
});
```

## `dgram` module functions

### dgram.createSocket(options[, callback])

<!-- YAML
added: v0.11.13
changes:

  - version: v8.6.0
    pr-url: https://github.com/nodejs/node/pull/14560
    description: The `lookup` option is supported.
  - version: v8.7.0
    pr-url: https://github.com/nodejs/node/pull/13623
    description: The `recvBufferSize` and `sendBufferSize` options are
                 supported now.
-->

* `options` {Object} Available options are: 
  * `type` {string} The family of socket. Must be either `'udp4'` or `'udp6'`. Required.
  * `reuseAddr` {boolean} When `true` [`socket.bind()`][] will reuse the address, even if another process has already bound a socket on it. **Default:** `false`.
  * `recvBufferSize` {number} - Sets the `SO_RCVBUF` socket value.
  * `sendBufferSize` {number} - Sets the `SO_SNDBUF` socket value.
  * `lookup` {Function} Custom lookup function. **Default:** [`dns.lookup()`][].
* `callback` {Function} Attached as a listener for `'message'` events. Optional.
* Returns: {dgram.Socket}

Creates a `dgram.Socket` object. Once the socket is created, calling [`socket.bind()`][] will instruct the socket to begin listening for datagram messages. When `address` and `port` are not passed to [`socket.bind()`][] the method will bind the socket to the "all interfaces" address on a random port (it does the right thing for both `udp4` and `udp6` sockets). The bound address and port can be retrieved using [`socket.address().address`][] and [`socket.address().port`][].

### dgram.createSocket(type[, callback])

<!-- YAML
added: v0.1.99
-->

* `type` {string} - Either `'udp4'` or `'udp6'`.
* `callback` {Function} - Attached as a listener to `'message'` events.
* Returns: {dgram.Socket}

Creates a `dgram.Socket` object of the specified `type`. The `type` argument can be either `'udp4'` or `'udp6'`. An optional `callback` function can be passed which is added as a listener for `'message'` events.

Once the socket is created, calling [`socket.bind()`][] will instruct the socket to begin listening for datagram messages. When `address` and `port` are not passed to [`socket.bind()`][] the method will bind the socket to the "all interfaces" address on a random port (it does the right thing for both `udp4` and `udp6` sockets). The bound address and port can be retrieved using [`socket.address().address`][] and [`socket.address().port`][].