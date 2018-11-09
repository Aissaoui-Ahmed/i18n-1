# Rete

<!--introduced_in=v0.10.0-->

<!--lint disable maximum-line-length-->

> Stabilità: 2 - Stabile

Il modulo di `rete` fornisce un'API di rete asincrona per la creazione dei server basati sullo stream TCP o [IPC](#net_ipc_support) ([`net.createServer()`][]) e per i client ([`net.createConnection()`][]).

Ci si può accedere usando:

```js
const net = require('net');
```

## Supporto IPC

Il modulo di `rete` supporta IPC con pipe denominate su Windows e i socket di dominio UNIX su altri sistemi operativi.

### Identificazione dei percorsi per le connessioni IPC

[`net.connect()`][], [`net.createConnection()`][], [`server.listen()`][] e [`socket.connect()`][] prendi un parametro `path` per identificare gli endpoint IPC.

Su UNIX, il dominio locale è anche noto come il dominio UNIX. Il percorso è un pathname del filesystem. Esso viene troncato per `sizeof (sockaddr_un.sun_path) - 1`, che varia su diversi sistemi operativi compresi tra 91 e 107 byte. I valori tipici sono 107 su Linux e 103 su macOS. Il percorso è soggetto alle stesse convenzioni di denominazione e alle stesse verifiche dei permessi sulla creazione di file. Se il socket del dominio UNIX (che è visibile come un file system path) viene creato e utilizzato in combinazione con una delle astrazioni dell'API di Node.js come [`net.createServer()`][], sarà scollegato come parte del [`server.close()`][]. D'altra parte, se viene creato e utilizzato al di fuori di queste astrazioni, l'utente avrà bisogno di rimuoverlo manualmente. Lo stesso si applica quando il percorso è stato creato da un'API Node.js ma il programma si interrompe bruscamente. In breve, un socket di dominio UNIX una volta creato con successo sarà visibile nel filesystem e perdurerà fino a quando non verrà scollegato.

Su Windows, il dominio locale viene implementato utilizzando una pipe denominata. Il percorso *deve* fare riferimento ad un ingresso nella `\\?\pipe` o `\\.\pipe`. Qualsiasi carattere è permesso, ma quest'ultimo potrebbe eseguire alcune elaborazioni dei nomi del pipe, come la risoluzione di `..` sequenze. Nonostante quello che potrebbe sembrare, lo spazio dei nomi della pipe è flat. Le pipe *non perdureranno*. Vengono rimosse quando viene chiuso l'ultimo riferimento ad esse. A differenza dei socket di dominio UNIX, Windows chiuderà e rimuoverà la pipe quando esce dal processo di proprietà.

L'escaping della stringa JavaScript richiede che i percorsi siano specificati con una barra rovesciata extra di escaping come:

```js
net.createServer().listen(
  path.join('\\\\?\\pipe', process.cwd(), 'myctl'));
```

## Classe: Server di rete

<!-- YAML
added: v0.1.90
-->

Questa classe viene utilizzata per creare un server TCP o [IPC](#net_ipc_support).

### nuovo Server di rete (\[options\]\[, connectionListener\])

* Restituisce: {net.Server}

Vedi [`net.createServer([options][, connectionListener])`][`net.createServer()`].

`net.Server` è un [`EventEmitter`][] con i seguenti eventi:

### Event: 'close'

<!-- YAML
added: v0.5.0
-->

Emesso quando il server si chiude. Tieni presente che se esistono connessioni, questo evento non viene emesso fino a quando tutte le connessioni non sono terminate.

### Event: 'connection'

<!-- YAML
added: v0.1.90
-->

* {net.Socket} L'object della connessione

Emesso quando viene effettuata una nuova connessione. `socket`è un'istanza di `net.socket`.

### Event: 'error'

<!-- YAML
added: v0.1.90
-->

* {Error}

Emesso quando si verifica un errore. A differenza di [`net.Socket`][], l'evento [`'close'`][] **non** sarà emesso direttamente in seguito a questo evento a meno che [`server.close ()`][] sia denominato manualmente. Vedi l'esempio nella discussione del [`server.listen()`][].

### Event: 'listening'

<!-- YAML
added: v0.1.90
-->

Emesso quando il server ha eseguito la funzione di binding dopo aver chiamato [` server.listen()`][].

### indirizzi del server()

<!-- YAML
added: v0.1.90
-->

* Restituisce: {Object}

Restituisce `l'indirizzo` della funzione binding, l'indirizzo denominato `family` e la `porta` del server come riportato dal sistema operativo se si esegue il listening su un socket IP (utile per trovare quale porta è stata assegnata quando si ottiene un indirizzo assegnato dal sistema operativo): `{ port: 12346, family: 'IPv4', address: '127.0.0.1' }`.

Per un server in ascolto su una pipe o un socket di dominio UNIX, viene restituito il nome sotto forma di una stringa.

Esempio:

```js
const server = net.createServer ((socket) => {
   socket.end('goodbye\n');
}). on('error', (err) = > {
   // esegui l'handle degli errori qui
   lancia err;
});

// prendi una porta inutilizzata arbitraria.
server.listen(() => {
   console.log ('server aperto su', server.address ());
});
```

Non chiamare `server.address()` finché non è stato emesso l'evento `'listening'`.

### server.close([callback])

<!-- YAML
added: v0.1.90
-->

* Restituisce: {net.Server}

Impedisci al server di accettare nuove connessioni e mantiene le connessioni esistenti. Questa funzione è asincrona, il server è finalmente chiuso quando tutte le connessioni sono terminate e il server emette un evento [`'close'`][]. Il `callback` facoltativo verrà chiamato una volta che si verifica l'evento `'close'`. Diversamente da quell'evento, sarà chiamato con un `Errore` come suo unico argomento se il server non era aperto quando era chiuso.

### server.connections

<!-- YAML
added: v0.2.0
deprecated: v0.9.7
-->

> Stabilità: 0 - Obsoleto: Utilizza invece [`server.getConnections()`][].

Il numero di connessioni simultanee sul server.

Questo diventa `null` quando si invia un socket a un child con [`child_process.fork()`][]. Per effettuare il polling dei fork e ottenere il numero corrente di connessioni attive, utilizza invece l'asincronia [`server.getConnections()`][].

### server.getConnections(callback)

<!-- YAML
added: v0.9.7
-->

* Restituisce: {net.Server}

Assegnare asincronicamente il numero di connessioni simultanee sul server. Funziona quando i socket sono stati inviati ai fork.

La callback dovrebbe accettare due argomenti `err` e `count`.

### server.listen()

Avvia un server che esegue il listening per le connessioni. Un `net.Server` può essere un TCP o un server [IPC](#net_ipc_support) in base a ciò che ascolta.

Possibili firme:

* [`server.listen(handle[, backlog][, callback])`][`server.listen(handle)`]
* [`server.listen(options[, callback])`][`server.listen(options)`]
* [`server.listen(path[, backlog][, callback])`][`server.listen(path)`] per i server [IPC](#net_ipc_support)
* [ `server.listen([port[, host[, backlog]]][, callback])`](#net_server_listen_port_host_backlog_callback) per i server TCP

Questa funzione è asincrona. Quando il server inizia ad eseguire il listening, il[`'listening'`][] verrà emesso. L'ultimo parametro `callback` verrà aggiunto come un listener per l'evento [`'listening'`][].

Tutti i metodi di `listen()` possono prendere un parametro `backlog` per specificare la massima lunghezza della coda delle connessioni in sospeso. La lunghezza effettiva sarà determinata dal sistema operativo attraverso le impostazioni di sysctl come `tcp_max_syn_backlog` e `somaxconn` su Linux. Il valore predefinito di questo parametro è 511 (non 512).

Tutti [`net.Socket`][] sono impostati su `SO_REUSEADDR` (vedi [socket(7)](http://man7.org/linux/man-pages/man7/socket.7.html)per i dettagli).

Il metodo `server.listen()` può essere chiamato ancora se e solo se c'è stato un errore durante la prima chiamata `server.listen()` o se il `server.close()` è stato chiamato. In caso contrario, verrà lanciato un errore `ERR_SERVER_ALREADY_LISTEN`.

Uno degli errori più comuni generati durante il listening è `EADDRINUSE`. Ciò accade quando un altro server sta già eseguendo il listening sulla/o `port` / `path` / `handle` richiesto. Un modo per eseguire l'handle sarebbe quello di riprovare dopo un certo periodo di tempo:

```js
server.on('error', (e) => {
  if (e.code === 'EADDRINUSE') {
    console.log('Address in use, retrying...');
    setTimeout(() => {
      server.close();
      server.listen(PORT, HOST);
    }, 1000);
  }
});
```

#### server.listen(handle\[, backlog\]\[, callback\])

<!-- YAML
added: v0.5.10
-->

* `handle` {Object}
* `backlog`{number} Parametro comune delle funzioni [`server.listen()`][]
* `callback`{Function} Parametro comune delle funzioni [`server.listen()`][]
* Restituisce: {net.Server}

Avvia un server che "ascolta" le connessioni su un determinato `handle` che è già stato associato a una porta, a un socket di dominio UNIX o a una pipe denominata Windows.

L'object `handle` può essere sia un server che un socket (qualsiasi cosa con un membro del `_handle ` sottostante), o un object con un membro `fd ` che è un descrittore di file valido.

Il listening su un descrittore di file non è supportato su Windows.

#### server.listen(options[, callback])

<!-- YAML
added: v0.11.14
-->

* `options` {Object} Obbligatorio. Supporta le seguenti proprietà: 
  * `port` {number}
  * `host`{string}
  * `path` {string} verrà ignorato se la `porta` è specificata. Vedi [Identificazione dei percorsi per le connessioni IPC](#net_identifying_paths_for_ipc_connections).
  * `backlog`{number} Parametro comune delle funzioni [`server.listen()`][].
  * `exclusive`{boolean}**Default** `false`
* `callback`{Function} Parametro comune delle funzioni [`server.listen()`][].
* Restituisce: {net.Server}

Se è la `porta` è specificata, si comporta come
<a href="#net_server_listen_port_host_backlog_callback">
<code> server.listen([port [, host[, backlog]]][, callback])</code> </a>. Altrimenti, se il `percorso` è specificato, si comporta come [`server.listen (path [, backlog] [, callback])`][`server.listen (path)`]. Se nessuno di essi viene specificato, verrà lanciato un errore.

Se `exclusive` è `false` (predefinito), i lavoratori del cluster, quindi, utilizzeranno lo stesso handle sottostante che consente di condividere i compiti di handling delle connessioni. Quando `exclusive` è `true`, l'handle non viene condiviso e il tentativo di condivisione della porta genera un errore. Un esempio che esegue il listening su una exclusive port è mostrato qui di seguito.

```js
server.listen({
  host: 'localhost',
  port: 80,
  exclusive: true
});
```

#### server.listen(path\[, backlog\]\[, callback\])

<!-- YAML
added: v0.1.90
-->

* `path` {string} Percorso che il server deve ascoltare. Vedi [Identificazione dei percorsi per le connessioni IPC](#net_identifying_paths_for_ipc_connections).
* `backlog`{number} Parametro comune delle funzioni [`server.listen()`][].
* `callback`{Function} Parametro comune delle funzioni [`server.listen()`][].
* Restituisce: {net.Server}

Avvia un server [IPC](#net_ipc_support) che esegua il listening per le connessioni sul `path` indicato.

#### server.listen(\[port[, host[, backlog]]\]\[, callback\])

<!-- YAML
added: v0.1.90
-->

* `port` {number}
* `host`{string}
* `backlog`{number} Parametro comune delle funzioni [`server.listen()`][].
* `callback`{Function} Parametro comune delle funzioni [`server.listen()`][].
* Restituisce: {net.Server}

Avvia un server TCP che esegua il listening per le connessioni sulla `porta` e sull' `host`.

Se la `porta` è omessa o è 0, il sistema operativo assegnerà arbitrariamente una porta non utilizzata, che può essere recuperata usando `server.address().port` dopo che l'evento [`'listening'`][] è stato emesso.

Se l'`host` viene omesso, il server accetterà connessioni su un [indirizzo IPv6 non specificato ](https://en.wikipedia.org/wiki/IPv6_address#Unspecified_address) (`::`) quando IPv6 è disponibile, oppure [l'indirizzo IPv4 non specificato ](https://en.wikipedia.org/wiki/0.0.0.0) o altrimenti su (` 0.0.0.0 `).

Nella maggior parte dei sistemi operativi, eseguire il listening per [l'indirizzo IPv6 non specificato](https://en.wikipedia.org/wiki/IPv6_address#Unspecified_address) (`::`) può portare anche il `net.Server` ad eseguire i listening sull' [indirizzo IPv4 non specificato](https://en.wikipedia.org/wiki/0.0.0.0) (`0.0.0.0`).

### server.listening

<!-- YAML
added: v5.7.0
-->

* {boolean} Indica se il server "sta ascoltando" o meno le connessioni.

### server.maxConnections

<!-- YAML
added: v0.2.0
-->

Imposta questa proprietà per rifiutare le connessioni quando il conteggio della connessione del server diventa alto.

Non è consigliabile utilizzare questa opzione una volta che un socket è stato inviato a un child con [`child_process.fork()`][].

### server.ref()

<!-- YAML
added: v0.9.1
-->

* Restituisce: {net.Server}

A differenza di`unref()`, chiamare `ref()` su un server precedente `unref`ed *non* permetterà l'uscita dal programma se esso è l'unico server rimasto ( l'azione predefinita). Se il server `ref`ed sta chiamando `ref()` di nuovo non avrà effetto.

### server.unref()

<!-- YAML
added: v0.9.1
-->

* Restituisce: {net.Server}

Chiamare `unref ()` su un server consentirà l'uscita dal programma se questo è l'unico server attivo nel sistema degli eventi. Se il server `unref`ed sta già chiamando `unref()` nuovamente, non avrà effetto.

## Classe: net.Socket

<!-- YAML
added: v0.3.4
-->

Questa classe è un'astrazione di un socket TCP o un endpoint di streaming [IPC](#net_ipc_support) (utilizza le pipe denominate su Windows e, in caso contrario, i socket UNIX del dominio). Un `net.Socket` è anche un [duplex stream](stream.html#stream_class_stream_duplex), in modo tale che possa essere sia leggibile che scrivibile ed è anche un [`EventEmitter`][].

Un `net.Socket` può essere creato dall'utente e utilizzato direttamente per interagire con un server. Per esempio, esso viene restituito da una [`net.createConnection()`][] in modo tale che l'utente possa utilizzarlo per comunicare con il server.

Esso può ache essere creato da Node.js e trasmesso all'utente quando una connessione viene ricevuta. Per esempio, esso viene trasmesso ai listener di un evento [`'connection'`][] emesso su un [`net.Server`][], in modo tale che l'utente possa utilizzarlo per interagire con il client.

### new net.Socket([options])

<!-- YAML
added: v0.3.4
-->

Crea un nuovo socket object.

* `options` {Object} Le opzioni disponibili sono: 
  * `fd`{number} Se specificato, rivesti un socket esistente con il descrittore di file specificato, altrimenti verrà creato un nuovo socket.
  * `allowHalfOpen`{boolean} Indica se sono consentite le connessioni TPC semi-aperte. Vedi [`net.createServer()`][] e l'evento [`'end'`][] per dettagli. **Default:** `false`.
  * `readable` {boolean} Consenti le letture sul socket quando viene passato un `fd`, altrimenti ignorato. **Default:** `false`.
  * `readable` {boolean} Consenti le scritture sul socket quando viene passato un `fd`, altrimenti ignorato. **Default:** `false`.
* Restituisce: {net.Socket}

Il socket recentemente creato può essere un socket TCP o uno streaming [IPC](#net_ipc_support) endpoint, a seconda di cosa esso [`connect ()`] [`socket.connect()`] per.

### Event: 'close'

<!-- YAML
added: v0.1.90
-->

* `hadError` {boolean} `true` se il socket ha avuto un errore di trasmissione.

Emesso quando il socket è completamente chiuso. L'argomento `hadError` è un booleano che dice se il socket è stato chiuso a causa di un errore di trasmissione.

### Event: 'connect'

<!-- YAML
added: v0.1.90
-->

Emesso quando una connessione del socket è stabilita con successo. Vedi [`net.createConnection()`][].

### Event: 'data'

<!-- YAML
added: v0.1.90
-->

* {Buffer|string}

Emesso quando i dati sono ricevuti. L'argomento `data` sarà un `Buffer` o una `string`. La codifica dei dati è impostata da [`socket.setEncoding()`][].

Ricorda che i **dati andranno persi** se non ci sono listener quando un `Socket` emette un evento `"data"`.

### Event: 'drain'

<!-- YAML
added: v0.1.90
-->

Emesso quando il buffer di scrittura diventa vuoto. Può essere utilizzato per eseguire il throttling degli uploads.

Vedi inoltre: i valori restituiti di `socket.write()`.

### Event: 'end'

<!-- YAML
added: v0.1.90
-->

Emesso quando l'altra estremità del socket invia un pacchetto FIN, terminando così il lato leggibile del socket.

Per impostazione predefinita (`allowHalfOpen` è `false`) il socket invierà un pacchetto FIN indietro e distruggerà il suo descrittore di file una volta che ha scritto la sua coda di scrittura in sospeso. Tuttavia, se `allowHalfOpen` è impostato su `true`, il socket non sarà automaticamente [`end ()`] [`socket.end()`] il suo lato scrivibile, consentendo all'utente di scrivere una quantità arbitraria di dati. L'utente deve chiamare [`end()`] [`socket.end ()`] esplicitamente per chiudere la connessione (cioè rinviare un pacchetto FIN).

### Event: 'error'

<!-- YAML
added: v0.1.90
-->

* {Error}

Emesso quando si verifica un errore. L'evento `'close'` sarà chiamata direttamente dopo questo evento.

### Event: 'lookup'

<!-- YAML
added: v0.11.3
changes:

  - version: v5.10.0
    pr-url: https://github.com/nodejs/node/pull/5598
    description: The `host` parameter is supported now.
-->

Emesso dopo aver risolto l'hostname ma prima della connessione. Non applicabile ai socket UNIX.

* `err` {Error|null} L'object dell'errore. Vedi [`dns.lookup()`][].
* `address` {string} L'indirizzo IP.
* `family` {string|null} Il tipo di indirizzo. Vedi [`dns.lookup()`][].
* `host` {string} L'hostname.

### Evento: 'ready'

<!-- YAML
added: v9.11.0
-->

Emesso quando un socket è pronto per essere utilizzato.

Attivato immediatamente dopo `'connect'`.

### Event: 'timeout'

<!-- YAML
added: v0.1.90
-->

Emesso se il socket scade dall'inattività. Questo è solo per informare che il socket è rimasto inattivo. L'utente deve chiudere manualmente la connessione.

Vedi anche: [`socket.setTimeout()`][].

### socket.address()

<!-- YAML
added: v0.1.90
-->

* Restituisce: {Object}

Restituisce `l'indirizzo` della funzione binding, l'indirizzo denominato `family` e la `porta` del socket come riportato dal sistema operativo: `{ port: 12346, family: 'IPv4', address: '127.0.0.1' }`

### socket.bufferSize

<!-- YAML
added: v0.3.8
-->

`net.Socket` ha la proprietà che permette al `socket.write()` di funzionare sempre. Questo è per aiutare gli utenti ad essere operativi nell'immediato. Il computer non può sempre tenere il passo con la quantità di dati che viene scritta per un socket: la connessione di rete potrebbe essere troppo lenta. Node.js accoderà internamente i dati scritti per un socket e li invierà via cavo quando è possibile. (Internamente sta eseguendo il polling sul descrittore di file del socket per essere scrivibile).

La conseguenza di questo buffering interno è che la memoria può crescere. Questa proprietà mostra il numero di caratteri attualmente memorizzati nel buffer per essere scritti. (Il numero di caratteri è approssimativamente uguale al numero di byte da scrivere, ma il buffer può contenere stringhe e le stringhe sono codificate con la modalità lazy, quindi il numero esatto di byte non è noto.)

Gli utenti con esperienza di `bufferSize` di grandi dimensioni o in crescita dovrebbero tentare di "accelerare" i flussi di dati nel loro programma con [`socket.pause()`][] e [`socket.resume()`][].

### socket.bytesRead

<!-- YAML
added: v0.5.3
-->

La quantità di byte ricevuti.

### socket.bytesWritten

<!-- YAML
added: v0.5.3
-->

La quantità di byte inviati.

### socket.connect()

Inizia una connessione su un socket indicato.

Possibili firme:

* [`socket.connect(options[, connectListener])`][`socket.connect(options)`]
* [`socket.connect(path[, connectListener])`][`socket.connect(path)`] per le connessioni [IPC](#net_ipc_support).
* [`socket.connect(port[, host][, connectListener])`][`socket.connect(port, host)`] per le connessioni TPC.
* Restituisce: {net.Socket} Il socket stesso.

Questa funzione è asincrona. Quando viene stabilita la connessione, verrà emesso l'evento [`'connect'`][]. Se c'è un problema di connessione, invece di un evento [`'connect'`][], verrà generato un evento [`'error'`][] con l'errore passato al listener dell' [`'error'`][]. L'ultimo parametro `connectListener`, se fornito, sarà aggiunto **una volta** come un listener per l'evento [`'connect'`][].

#### socket.connect(options[, connectListener])

<!-- YAML
added: v0.1.90
changes:

  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/6021
    description: The `hints` option defaults to `0` in all cases now.
                 Previously, in the absence of the `family` option it would
                 default to `dns.ADDRCONFIG | dns.V4MAPPED`.
  - version: v5.11.0
    pr-url: https://github.com/nodejs/node/pull/6000
    description: The `hints` option is supported now.
-->

* `options` {Object}
* `connectListener`{Function} Parametro comune dei metodi [`server.listen()`][]. Verrà aggiunto una volta come un listener per l'evento [`'connect'`][].
* Restituisce: {net.Socket} Il socket stesso.

Inizia una connessione su un socket indicato. Normalmente questo metodo non è necessario, il socket deve essere creato e aperto con [`net.createConnection()`][]. Utilizza questo solo quando si implementa un socket personalizzato.

Per le connessioni TPC, le `options` disponibili sono:

* `port` {number} Richiesto. Porta a cui il socket dovrebbe connettersi.
* `host`{string} Host a cui il socket dovrebbe connettersi. **Default:** `'localhost'`.
* `localAdress`{string} Indirizzo locale dal quale il socket dovrebbe connettersi.
* `localPort`{number} porta locale dalla quale dovrebbe connettersi il socket.
* `family` {number}: La versione dello stack IP può essere `4` o `6`. **Default:** `4`.
* `hints` {number} Facoltativo [`dns.lookup()` hints][].
* `lookup` {Function} Funzione lookup (di ricerca) personalizzata. **Default:** [`dns.lookup()`][].

Per le connessioni IPC<0>, le `options` disponibili sono:</p> 

* `path` {string} Richiesto. Percorso a cui il client dovrebbe connettersi. Vedi [Identificazione dei percorsi per le connessioni IPC](#net_identifying_paths_for_ipc_connections). Se fornito, le opzioni specifiche TCP sopra sono ignorate.

#### socket.connect(path[, connectListener])

* `path` {string} Percorso a cui il client dovrebbe connettersi. Vedi [Identificazione dei percorsi per le connessioni IPC](#net_identifying_paths_for_ipc_connections).
* `connectListener`{Function} Parametro comune dei metodi [`server.listen()`][]. Verrà aggiunto una volta come un listener per l'evento [`'connect'`][].
* Restituisce: {net.Socket} Il socket stesso.

Inizia una connessione [IPC](#net_ipc_support) sul socket indicato.

Pseudonimo per `socket.connect(options[, connectListener])`][`socket.connect(options)`] chiamati/e con`{ path: path }` come `options`.

#### socket.connect(port\[, host\]\[, connectListener\])

<!-- YAML
added: v0.1.90
-->

* `port` {number} Porta a cui il client dovrebbe connettersi.
* `host` {string} Host a cui il client dovrebbe connettersi.
* `connectListener`{Function} Parametro comune dei metodi [`server.listen()`][]. Verrà aggiunto una volta come un listener per l'evento [`'connect'`][].
* Restituisce: {net.Socket} Il socket stesso.

Inizia una connessione TPC sul socket indicato.

Pseudonimo per `socket.connect(options[, connectListener])`][`socket.connect(options)`] chiamati/e con`{port: port, host: host}` come `options`.

### socket.connecting

<!-- YAML
added: v6.1.0
-->

Se`true` - [`socket.connect(options[, connectListener])`][`socket.connect(options)`] è stato chiamato e non è stato ancora finito. Sarà impostato su `false` prima di emettere l'evento `'connect'` e/o chiamare [`socket.connect(options[,connectListener])`][` socket.connect (options)`]'s callback.

### socket.destroy([exception])

<!-- YAML
added: v0.1.90
-->

* Restituisce: {net.Socket}

Garantisce che non si verifichi più attività di I/O su questo socket. Solo necessario in caso di errori (errore di analisi o simili).

Se `exception` è specificata, verrà emesso un evento [`'error'`][] e tutti i listener per quell'evento riceveranno `exception` come argomento.

### socket.destroyed

* {boolean} Indicates if the connection is destroyed or not. Once a connection is destroyed no further data can be transferred using it.

### socket.end(\[data\]\[, encoding\])

<!-- YAML
added: v0.1.90
-->

* Returns: {net.Socket} The socket itself.

Half-closes the socket. i.e., it sends a FIN packet. It is possible the server will still send some data.

If `data` is specified, it is equivalent to calling `socket.write(data, encoding)` followed by [`socket.end()`][].

### socket.localAddress

<!-- YAML
added: v0.9.6
-->

The string representation of the local IP address the remote client is connecting on. For example, in a server listening on `'0.0.0.0'`, if a client connects on `'192.168.1.1'`, the value of `socket.localAddress` would be `'192.168.1.1'`.

### socket.localPort

<!-- YAML
added: v0.9.6
-->

The numeric representation of the local port. For example, `80` or `21`.

### socket.pause()

* Returns: {net.Socket} The socket itself.

Pauses the reading of data. That is, [`'data'`][] events will not be emitted. Useful to throttle back an upload.

### socket.ref()

<!-- YAML
added: v0.9.1
-->

* Returns: {net.Socket} The socket itself.

Opposite of `unref()`, calling `ref()` on a previously `unref`ed socket will *not* let the program exit if it's the only socket left (the default behavior). If the socket is `ref`ed calling `ref` again will have no effect.

### socket.remoteAddress

<!-- YAML
added: v0.5.10
-->

The string representation of the remote IP address. For example, `'74.125.127.100'` or `'2001:4860:a005::68'`. Value may be `undefined` if the socket is destroyed (for example, if the client disconnected).

### socket.remoteFamily

<!-- YAML
added: v0.11.14
-->

The string representation of the remote IP family. `'IPv4'` or `'IPv6'`.

### socket.remotePort

<!-- YAML
added: v0.5.10
-->

The numeric representation of the remote port. For example, `80` or `21`.

### socket.resume()

* Returns: {net.Socket} The socket itself.

Resumes reading after a call to [`socket.pause()`][].

### socket.setEncoding([encoding])

<!-- YAML
added: v0.1.90
-->

* Returns: {net.Socket} The socket itself.

Set the encoding for the socket as a [Readable Stream](stream.html#stream_class_stream_readable). See [`readable.setEncoding()`][] for more information.

### socket.setKeepAlive(\[enable\]\[, initialDelay\])

<!-- YAML
added: v0.1.92
-->

* `enable` {boolean} **Default:** `false`
* `initialDelay` {number} **Default:** `0`
* Returns: {net.Socket} The socket itself.

Enable/disable keep-alive functionality, and optionally set the initial delay before the first keepalive probe is sent on an idle socket.

Set `initialDelay` (in milliseconds) to set the delay between the last data packet received and the first keepalive probe. Setting `0` for `initialDelay` will leave the value unchanged from the default (or previous) setting.

### socket.setNoDelay([noDelay])

<!-- YAML
added: v0.1.90
-->

* `noDelay` {boolean} **Default:** `true`
* Returns: {net.Socket} The socket itself.

Disables the Nagle algorithm. By default TCP connections use the Nagle algorithm, they buffer data before sending it off. Setting `true` for `noDelay` will immediately fire off data each time `socket.write()` is called.

### socket.setTimeout(timeout[, callback])

<!-- YAML
added: v0.1.90
-->

* Returns: {net.Socket} The socket itself.

Sets the socket to timeout after `timeout` milliseconds of inactivity on the socket. By default `net.Socket` do not have a timeout.

When an idle timeout is triggered the socket will receive a [`'timeout'`][] event but the connection will not be severed. The user must manually call [`socket.end()`][] or [`socket.destroy()`][] to end the connection.

```js
socket.setTimeout(3000);
socket.on('timeout', () => {
  console.log('socket timeout');
  socket.end();
});
```

If `timeout` is 0, then the existing idle timeout is disabled.

The optional `callback` parameter will be added as a one-time listener for the [`'timeout'`][] event.

### socket.unref()

<!-- YAML
added: v0.9.1
-->

* Returns: {net.Socket} The socket itself.

Calling `unref()` on a socket will allow the program to exit if this is the only active socket in the event system. If the socket is already `unref`ed calling `unref()` again will have no effect.

### socket.write(data\[, encoding\]\[, callback\])

<!-- YAML
added: v0.1.90
-->

* `data` {string|Buffer|Uint8Array}
* `encoding` {string} Only used when data is `string`. **Default:** `utf8`.
* `callback` {Function}
* Returns: {boolean}

Sends data on the socket. The second parameter specifies the encoding in the case of a string — it defaults to UTF8 encoding.

Returns `true` if the entire data was flushed successfully to the kernel buffer. Returns `false` if all or part of the data was queued in user memory. [`'drain'`][] will be emitted when the buffer is again free.

The optional `callback` parameter will be executed when the data is finally written out - this may not be immediately.

See `Writable` stream [`write()`](stream.html#stream_writable_write_chunk_encoding_callback) method for more information.

## net.connect()

Aliases to [`net.createConnection()`][`net.createConnection()`].

Possible signatures:

* [`net.connect(options[, connectListener])`][`net.connect(options)`]
* [`net.connect(path[, connectListener])`][`net.connect(path)`] for [IPC](#net_ipc_support) connections.
* [`net.connect(port[, host][, connectListener])`][`net.connect(port, host)`] for TCP connections.

### net.connect(options[, connectListener])

<!-- YAML
added: v0.7.0
--> Alias to [

`net.createConnection(options[, connectListener])`][`net.createConnection(options)`].

### net.connect(path[, connectListener])

<!-- YAML
added: v0.1.90
-->

Alias to [`net.createConnection(path[, connectListener])`][`net.createConnection(path)`].

### net.connect(port\[, host\]\[, connectListener\])

<!-- YAML
added: v0.1.90
-->

Alias to [`net.createConnection(port[, host][, connectListener])`][`net.createConnection(port, host)`].

## net.createConnection()

A factory function, which creates a new [`net.Socket`][], immediately initiates connection with [`socket.connect()`][], then returns the `net.Socket` that starts the connection.

When the connection is established, a [`'connect'`][] event will be emitted on the returned socket. The last parameter `connectListener`, if supplied, will be added as a listener for the [`'connect'`][] event **once**.

Possible signatures:

* [`net.createConnection(options[, connectListener])`][`net.createConnection(options)`]
* [`net.createConnection(path[, connectListener])`][`net.createConnection(path)`] for [IPC](#net_ipc_support) connections.
* [`net.createConnection(port[, host][, connectListener])`][`net.createConnection(port, host)`] for TCP connections.

The [`net.connect()`][] function is an alias to this function.

### net.createConnection(options[, connectListener])

<!-- YAML
added: v0.1.90
-->

* `options` {Object} Required. Will be passed to both the [`new net.Socket([options])`][`new net.Socket(options)`] call and the [`socket.connect(options[, connectListener])`][`socket.connect(options)`] method.
* `connectListener` {Function} Common parameter of the [`net.createConnection()`][] functions. If supplied, will be added as a listener for the [`'connect'`][] event on the returned socket once.
* Returns: {net.Socket} The newly created socket used to start the connection.

For available options, see [`new net.Socket([options])`][`new net.Socket(options)`] and [`socket.connect(options[, connectListener])`][`socket.connect(options)`].

Additional options:

* `timeout` {number} If set, will be used to call [`socket.setTimeout(timeout)`][] after the socket is created, but before it starts the connection.

Following is an example of a client of the echo server described in the [`net.createServer()`][] section:

```js
const net = require('net');
const client = net.createConnection({ port: 8124 }, () => {
  // 'connect' listener
  console.log('connected to server!');
  client.write('world!\r\n');
});
client.on('data', (data) => {
  console.log(data.toString());
  client.end();
});
client.on('end', () => {
  console.log('disconnected from server');
});
```

To connect on the socket `/tmp/echo.sock` the second line would just be changed to:

```js
const client = net.createConnection({ path: '/tmp/echo.sock' });
```

### net.createConnection(path[, connectListener])

<!-- YAML
added: v0.1.90
-->

* `path` {string} Path the socket should connect to. Will be passed to [`socket.connect(path[, connectListener])`][`socket.connect(path)`]. See [Identifying paths for IPC connections](#net_identifying_paths_for_ipc_connections).
* `connectListener` {Function} Common parameter of the [`net.createConnection()`][] functions, an "once" listener for the `'connect'` event on the initiating socket. Will be passed to [`socket.connect(path[, connectListener])`][`socket.connect(path)`].
* Returns: {net.Socket} The newly created socket used to start the connection.

Initiates an [IPC](#net_ipc_support) connection.

This function creates a new [`net.Socket`][] with all options set to default, immediately initiates connection with [`socket.connect(path[, connectListener])`][`socket.connect(path)`], then returns the `net.Socket` that starts the connection.

### net.createConnection(port\[, host\]\[, connectListener\])

<!-- YAML
added: v0.1.90
-->

* `port` {number} Port the socket should connect to. Will be passed to [`socket.connect(port[, host][, connectListener])`][`socket.connect(port, host)`].
* `host` {string} Host the socket should connect to. Will be passed to [`socket.connect(port[, host][, connectListener])`][`socket.connect(port, host)`]. **Default:** `'localhost'`.
* `connectListener` {Function} Common parameter of the [`net.createConnection()`][] functions, an "once" listener for the `'connect'` event on the initiating socket. Will be passed to [`socket.connect(path[, connectListener])`][`socket.connect(port, host)`].
* Returns: {net.Socket} The newly created socket used to start the connection.

Initiates a TCP connection.

This function creates a new [`net.Socket`][] with all options set to default, immediately initiates connection with [`socket.connect(port[, host][, connectListener])`][`socket.connect(port, host)`], then returns the `net.Socket` that starts the connection.

## net.createServer(\[options\]\[, connectionListener\])

<!-- YAML
added: v0.5.0
-->

Creates a new TCP or [IPC](#net_ipc_support) server.

* `options` {Object} 
  * `allowHalfOpen` {boolean} Indicates whether half-opened TCP connections are allowed. **Default:** `false`.
  * `pauseOnConnect` {boolean} Indicates whether the socket should be paused on incoming connections. **Default:** `false`.
* `connectionListener` {Function} Automatically set as a listener for the [`'connection'`][] event.
* Returns: {net.Server}

If `allowHalfOpen` is set to `true`, when the other end of the socket sends a FIN packet, the server will only send a FIN packet back when [`socket.end()`][] is explicitly called, until then the connection is half-closed (non-readable but still writable). See [`'end'`][] event and [RFC 1122](https://tools.ietf.org/html/rfc1122) (section 4.2.2.13) for more information.

If `pauseOnConnect` is set to `true`, then the socket associated with each incoming connection will be paused, and no data will be read from its handle. This allows connections to be passed between processes without any data being read by the original process. To begin reading data from a paused socket, call [`socket.resume()`][].

The server can be a TCP server or an [IPC](#net_ipc_support) server, depending on what it [`listen()`][`server.listen()`] to.

Here is an example of an TCP echo server which listens for connections on port 8124:

```js
const net = require('net');
const server = net.createServer((c) => {
  // 'connection' listener
  console.log('client connected');
  c.on('end', () => {
    console.log('client disconnected');
  });
  c.write('hello\r\n');
  c.pipe(c);
});
server.on('error', (err) => {
  throw err;
});
server.listen(8124, () => {
  console.log('server bound');
});
```

Test this by using `telnet`:

```console
$ telnet localhost 8124
```

To listen on the socket `/tmp/echo.sock` the third line from the last would just be changed to:

```js
server.listen('/tmp/echo.sock', () => {
  console.log('server bound');
});
```

Use `nc` to connect to a UNIX domain socket server:

```console
$ nc -U /tmp/echo.sock
```

## net.isIP(input)

<!-- YAML
added: v0.3.0
-->

* Returns: {integer}

Tests if input is an IP address. Returns `0` for invalid strings, returns `4` for IP version 4 addresses, and returns `6` for IP version 6 addresses.

## net.isIPv4(input)

<!-- YAML
added: v0.3.0
-->

* Returns: {boolean}

Returns `true` if input is a version 4 IP address, otherwise returns `false`.

## net.isIPv6(input)

<!-- YAML
added: v0.3.0
-->

* Returns: {boolean}

Returns `true` if input is a version 6 IP address, otherwise returns `false`.