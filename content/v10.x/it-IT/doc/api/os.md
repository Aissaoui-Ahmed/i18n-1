# SO

<!--introduced_in=v0.10.0-->

> Stabilità: 2 - Stabile

Il modulo `os` fornisce un numero di metodi di utilità relativi al sistema operativo. Ci si può accedere utilizzando:

```js
const os = require('os');
```

## os.EOL

<!-- YAML
added: v0.7.8
-->

* {string}

Una costante di stringa che definisce il marker di fine riga specifico del sistema operativo:

* `\n` su POSIX
* `\r\n` su Windows

## os.arch()

<!-- YAML
added: v0.5.0
-->

* Restituisce: {string}

Il metodo `os.arch()` restituisce una stringa che identifica l'architettura della CPU del sistema operativo per la quale è stato compilato il binario Node.js.

I possibili valori correnti sono: `'arm'`, `'arm64'`, `'ia32'`, `'mips'`, `'mipsel'`, `'ppc'`, `'ppc64'`, `'s390'`, `'s390x'`, `'x32'` e `'x64'`.

Equivalente a [`process.arch`][].

## os.constants

<!-- YAML
added: v6.3.0
-->

* {Object}

Restituisce un object contenente costanti specifiche del sistema operativo comunemente utilizzate per i codici di errore, segnali di processo e così via. Le costanti specifiche attualmente definite sono descritte nelle [OS Costants](#os_os_constants_1).

## os.cpus()

<!-- YAML
added: v0.3.3
-->

* Restituisce: {Object[]}

Il metodo `os.cpus()` restituisce un array degli object contenenti informazioni riguardo a ciascun core della CPU logica.

Le proprietà incluse in ogni oggetto includono:

* `model` {string}
* `speed` {number} (in MHz)
* `times` {Object} 
  * `user` {number} Il numero di millisecondi che la CPU ha speso in modalità utente.
  * `nice` {number} Il numero di millisecondi che la CPU ha speso nella modalità nice.
  * `sys` {number} Il numero di millisecondi che la CPU ha speso nella modalità sys.
  * `idle` {number} Il numero di millisecondi che la CPU ha speso nella modalità idle.
  * `irq` {number} Il numero di millisecondi che la CPU ha speso nella modalità irq.

<!-- eslint-disable semi -->

```js
[
  {
    model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
    speed: 2926,
    times: {
      user: 252020,
      nice: 0,
      sys: 30340,
      idle: 1070356870,
      irq: 0
    }
  },
  {
    model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
    speed: 2926,
    times: {
      user: 306960,
      nice: 0,
      sys: 26980,
      idle: 1071569080,
      irq: 0
    }
  },
  {
    model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
    speed: 2926,
    times: {
      user: 248450,
      nice: 0,
      sys: 21750,
      idle: 1070919370,
      irq: 0
    }
  },
  {
    model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
    speed: 2926,
    times: {
      user: 256880,
      nice: 0,
      sys: 19430,
      idle: 1070905480,
      irq: 20
    }
  },
  {
    model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
    speed: 2926,
    times: {
      user: 511580,
      nice: 20,
      sys: 40900,
      idle: 1070842510,
      irq: 0
    }
  },
  {
    model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
    speed: 2926,
    times: {
      user: 291660,
      nice: 0,
      sys: 34360,
      idle: 1070888000,
      irq: 10
    }
  },
  {
    model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
    speed: 2926,
    times: {
      user: 308260,
      nice: 0,
      sys: 55410,
      idle: 1071129970,
      irq: 880
    }
  },
  {
    model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
    speed: 2926,
    times: {
      user: 266450,
      nice: 1480,
      sys: 34920,
      idle: 1072572010,
      irq: 30
    }
  }
]
```

Poiché i valori `nice` sono specifici di UNIX, su Windows i valori `nice` di tutti i processori sono sempre 0.

## os.endianness()

<!-- YAML
added: v0.9.4
-->

* Restituisce: {string}

Il metodo`os.endianness()` restituisce una stringa che identifica l'endianness (l'ordine dei byte) della CPU *per la quale il binario Node.js è stato compilato*.

I valori possibili sono:

* `'BE'` per big endian
* `'LE'` per little endian.

## os.freemem()

<!-- YAML
added: v0.3.3
-->

* Restituisce: {integer}

Il metodo `os.freemem()` restituisce la quantità di memoria di sistema libera in byte come un numero intero.

## os.homedir()

<!-- YAML
added: v2.3.0
-->

* Restituisce: {string}

Il metodo `os.homedir()` restituisce la directory home dell'utente corrente come una stringa.

## os.hostname()

<!-- YAML
added: v0.3.3
-->

* Restituisce: {string}

Il metodo `os.hostname()` restituisce l'hostname del sistema operativo come una stringa.

## os.loadavg()

<!-- YAML
added: v0.3.3
-->

* Restituisce: {number[]}

Il metodo `os.loadavg()` restituisce un array contenente le medie di caricamento di 1, 5 e 15 minuti.

La media di caricamento è una misura dell'attività del sistema, calcolata dal sistema operativo ed espressa come un numero frazionario. Come regola generale, la media di caricamento dovrebbe idealmente essere inferiore al numero di CPU logiche nel sistema.

La media del caricamento è un concetto specifico UNIX con nessun equivalente reale sulle piattaforme Windows. Su Windows, il valore di ritorno è sempre `[0, 0, 0]`.

## os.networkInterfaces()

<!-- YAML
added: v0.6.0
-->

* Restituisce: {Object}

Il metodo `os.networkInterfaces()` restituisce un object contenente solo le interfacce di rete a cui è stato assegnato un indirizzo di rete.

Ogni chiave sull'oggetto restituito identifica un'interfaccia di rete. Il valore associato è un array degli object che descrivono, ognuno, un indirizzo di rete assegnato.

Le proprietà disponibili sull'object dell'indirizzo di rete assegnato includono:

* `address` {string} L'indirizzo IPv4 o IPv6 assegnato
* `netmask` {string} La maschera di rete IPv4 o IPv6
* `family` {string} O `IPv4` oppure `IPv6`
* `mac` {string} L'indirizzo MAC dell'interfaccia di rete
* `internal` {boolean} `true` se l'interfaccia di rete è un loopback o un'interfaccia simile che non è accessibile da remoto; altrimenti`false`
* `scopeid` {number} L'ID di scope IPv6 numerico (specificato solo quando`family` è `IPv6`)
* `cidr` {string} L'indirizzo IPv4 o IPv6 assegnato con il prefisso di routing nella notazione CIDR. Se la `netmask` non è valida, questa proprietà è impostata su `null`.

<!-- eslint-skip -->

```js
{
  lo: [
    {
      address: '127.0.0.1',
      netmask: '255.0.0.0',
      family: 'IPv4',
      mac: '00:00:00:00:00:00',
      internal: true,
      cidr: '127.0.0.1/8'
    },
    {
      address: '::1',
      netmask: 'ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff',
      family: 'IPv6',
      mac: '00:00:00:00:00:00',
      internal: true,
      cidr: '::1/128'
    }
  ],
  eth0: [
    {
      address: '192.168.1.108',
      netmask: '255.255.255.0',
      family: 'IPv4',
      mac: '01:02:03:0a:0b:0c',
      internal: false,
      cidr: '192.168.1.108/24'
    },
    {
      address: 'fe80::a00:27ff:fe4e:66a1',
      netmask: 'ffff:ffff:ffff:ffff::',
      family: 'IPv6',
      mac: '01:02:03:0a:0b:0c',
      internal: false,
      cidr: 'fe80::a00:27ff:fe4e:66a1/64'
    }
  ]
}
```

## os.platform()

<!-- YAML
added: v0.5.0
-->

* Restituisce: {string}

Il metodo `os.platform()` restituisce una stringa che identifica la piattaforma del sistema operativo impostata durante il tempo di compilazione di Node.js.

I valori attualmente possibili sono:

* `'aix'`
* `'darwin'`
* `'freebsd'`
* `'linux'`
* `'openbsd'`
* `'sunos'`
* `'win32'`

Equivalente a [`process.platform`][].

Può anche essere restituito il valore `'android'` se il Node.js è costruito sul sistema operativo Android. Tuttavia, al momento, il supporto Android in Node.js è considerato [come sperimentale](https://github.com/nodejs/node/blob/master/BUILDING.md#androidandroid-based-devices-eg-firefox-os).

## os.release()

<!-- YAML
added: v0.3.3
-->

* Restituisce: {string}

Il metodo `os.release()` restituisce una stringa che identifica la versione del sistema operativo.

Sui sistemi POSIX, la versione del sistema operativo è determinata chiamando [uname(3)](https://linux.die.net/man/3/uname). Su Windows, viene utilizzata `GetVersionExW()`. Si prega di consultare https://en.wikipedia.org/wiki/Uname#Examples per ulteriori informazioni.

## os.tmpdir()

<!-- YAML
added: v0.9.9
changes:

  - version: v2.0.0
    pr-url: https://github.com/nodejs/node/pull/747
    description: This function is now cross-platform consistent and no longer
                 returns a path with a trailing slash on any platform
-->

* Restituisce: {string}

Il metodo `os.tmpdir()` restituisce una stringa che specifica la directory predefinita del sistema operativo per i file temporanei.

## os.totalmem()

<!-- YAML
added: v0.3.3
-->

* Restituisce: {integer}

Il metodo `os.totalmem()` restituisce la quantità totale di memoria di sistema in byte come un numero intero.

## os.type()

<!-- YAML
added: v0.3.3
-->

* Restituisce: {string}

Il metodo `os.type()` restituisce una stringa che identifica il nome del sistema operativo restituito da [uname(3)](https://linux.die.net/man/3/uname). Per esempio su linux `'Linux'`, su macOS `'Darwin'` e su Windows `'Windows_NT'`.

Si prega di consultare https://en.wikipedia.org/wiki/Uname#Examples per ulteriori informazioni sull'output dell'esecuzione di [uname(3)](https://linux.die.net/man/3/uname) su vari sistemi operativi.

## os.uptime()

<!-- YAML
added: v0.3.3
changes:

  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/20129
    description: The result of this function no longer contains a fraction
                 component on Windows.
-->

* Restituisce: {integer}

Il metodo `os.uptime()` restituisce il tempo di attività del sistema in un numero di secondi.

## os.userInfo([options])

<!-- YAML
added: v6.0.0
-->

* `options` {Object} 
  * `encoding` {string} Codifica dei caratteri utilizzata per interpretare le stringhe risultanti. Se l'`encoding` è impostato su `'buffer'`, i valori `username`, `shell` e `homedir` diverranno istanze del `Buffer`. **Default:** `'utf8'`.
* Restituisce: {Object}

Il metodo `os.userInfo()` restituisce informazioni sull'utente attualmente efficace - sulle piattaforme POSIX, questo è solitamente un sottoinsieme del file delle password. L'oggetto restituito include `username`, `uid`, `gid`, `shell` e `homedir`. Su Windows, i campi `uid` e `gid` sono `-1` e `shell` è `null`.

Il valore di `homedir` restituito da `os.userInfo()` è fornito dal sistema operativo. Questo differisce dal risultato di `os.homedir()`, il quale esegue il query di diverse variabili d'ambiente per la home directory prima di ritornare alla risposta del sistema operativo.

## Constanti del SO

Le seguenti costante vengono esportate da `os.constants`.

Non tutte le costanti saranno disponibili su tutti i sistemi operativi.

### Costanti di Segnale

<!-- YAML
changes:

  - version: v5.11.0
    pr-url: https://github.com/nodejs/node/pull/6093
    description: Added support for `SIGINFO`.
-->

Le seguenti costanti di segnale vengono esportate da `os.constants.signals`:

<table>
  <tr>
    <th>Costante</th>
    <th>Descrizione</th>
  </tr>
  <tr>
    <td><code>SIGHUP</code></td>
    <td>Inviato per segnalare quando un terminale di controllo è chiuso o un parent    process si chiude.</td>
  </tr>
  <tr>
    <td><code>SIGINT</code></td>
    <td>Inviato per segnalare quando un utente desidera interrompere un processo    (`(Ctrl + C)`).</td>
  </tr>
  <tr>
    <td><code>SIGQUIT</code></td>
    <td>Inviato per segnalare quando un utente desidera terminare un processo ed eseguire un    core dump.</td>
  </tr>
  <tr>
    <td><code>SIGILL</code></td>
    <td>Inviato a un processo per notificare che si è tentato di eseguire un'istruzione illegale, non corretta, sconosciuta o privilegiata.</td>
  </tr>
  <tr>
    <td><code>SIGTRAP</code></td>
    <td>Inviato a un processo quando si è verificata un'eccezione.</td>
  </tr>
  <tr>
    <td><code>SIGABRT</code></td>
    <td>Inviato a un processo per richiedere l'annullamento.</td>
  </tr>
  <tr>
    <td><code>SIGIOT</code></td>
    <td>Sinonimo di <code>SIGABRT</code></td>
  </tr>
  <tr>
    <td><code>SIGBUS</code></td>
    <td>Inviato a un processo per notificare che ha causato un errore del bus.</td>
  </tr>
  <tr>
    <td><code>SIGFPE</code></td>
    <td>Inviato a un processo per notificare che ha eseguito un'operazione aritmetica non consentita.</td>
  </tr>
  <tr>
    <td><code>SIGKILL</code></td>
    <td>Inviato a un processo per terminarlo immediatamente.</td>
  </tr>
  <tr>
    <td><code>SIGUSR1</code> <code>SIGUSR2</code></td>
    <td>Inviato a un processo per identificare le condizioni definite dall'utente.</td>
  </tr>
  <tr>
    <td><code>SIGSEGV</code></td>
    <td>Inviato a un processo per notificare un errore di segmentazione.</td>
  </tr>
  <tr>
    <td><code>SIGPIPE</code></td>
    <td>Inviato a un processo quando ha tentato di scrivere su una pipe    disconnessa.</td>
  </tr>
  <tr>
    <td><code>SIGALRM</code></td>
    <td>Sent to a process when a system timer elapses.</td>
  </tr>
  <tr>
    <td><code>SIGTERM</code></td>
    <td>Nviato a un processo per richiedere la cessazione.</td>
  </tr>
  <tr>
    <td><code>SIGCHLD</code></td>
    <td>Inviato a un processo quando termina un child process.</td>
  </tr>
  <tr>
    <td><code>SIGSTKFLT</code></td>
    <td>Inviato a un processo per segnalare un errore di stack su un coprocessore.</td>
  </tr>
  <tr>
    <td><code>SIGCONT</code></td>
    <td>Inviato per dare istruzioni al sistema operativo di continuare un processo in sospeso.</td>
  </tr>
  <tr>
    <td><code>SIGSTOP</code></td>
    <td>Inviato per dare istruzioni al sistema operativo di interrompere un processo.</td>
  </tr>
  <tr>
    <td><code>SIGTSTP</code></td>
    <td>Inviato a un processo per richiedergli di fermarsi.</td>
  </tr>
  <tr>
    <td><code>SIGBREAK</code></td>
    <td>Inviato per indicare quando un utente desidera interrompere un processo.</td>
  </tr>
  <tr>
    <td><code>SIGTTIN</code></td>
    <td>Inviato a un processo quando viene letto dal TTY mentre è in    background.</td>
  </tr>
  <tr>
    <td><code>SIGTTOU</code></td>
    <td>Sent to a process when it writes to the TTY while in the
    background.</td>
  </tr>
  <tr>
    <td><code>SIGURG</code></td>
    <td>Inviato a un processo quando un socket ha dati urgenti da leggere.</td>
  </tr>
  <tr>
    <td><code>SIGXCPU</code></td>
    <td>Inviato a un processo quando ha superato il limite di utilizzo della CPU.</td>
  </tr>
  <tr>
    <td><code>SIGXFSZ</code></td>
    <td>Inviato a un processo quando cresce un file più grande del massimo    consentito.</td>
  </tr>
  <tr>
    <td><code>SIGVTALRM</code></td>
    <td>Inviato a un processo quando è trascorso un timer virtuale.</td>
  </tr>
  <tr>
    <td><code>SIGPROF</code></td>
    <td>Inviato a un processo quando è trascorso un timer di sistema.</td>
  </tr>
  <tr>
    <td><code>SIGWINCH</code></td>
    <td>Inviato a un processo quando il terminale di controllo ha cambiato le sue dimensioni.</td>
  </tr>
  <tr>
    <td><code>SIGIO</code></td>
    <td>Inviato a un processo quando I/O è disponibile.</td>
  </tr>
  <tr>
    <td><code>SIGPOLL</code></td>
    <td>Sinonimo di <code>SIGIO</code></td>
  </tr>
  <tr>
    <td><code>SIGLOST</code></td>
    <td>Inviato a un processo quando un file lock è stato perso.</td>
  </tr>
  <tr>
    <td><code>SIGPWR</code></td>
    <td>Inviato a un processo per notificare l'interruzione di corrente.</td>
  </tr>
  <tr>
    <td><code>SIGINFO</code></td>
    <td>Sinonimo di <code>SIGPWR</code></td>
  </tr>
  <tr>
    <td><code>SIGSYS</code></td>
    <td>Inviato a un processo per notificare un argomento non valido.</td>
  </tr>
  <tr>
    <td><code>SIGUNUSED</code></td>
    <td>Sinonimo di <code>SIGSYS</code></td>
  </tr>
</table>

### Costanti di Errore

Le seguenti costanti di errore vengono esportate da `os.constants.errno`:

#### Costanti di errore POSIX

<table>
  <tr>
    <th>Costante</th>
    <th>Descrizione</th>
  </tr>
  <tr>
    <td><code>E2BIG</code></td>
    <td>Indica che l'elenco degli argomenti è più lungo del previsto.</td>
  </tr>
  <tr>
    <td><code>EACCES</code></td>
    <td>Indica che l'operazione non ha avuto permessi sufficienti.</td>
  </tr>
  <tr>
    <td><code>EADDRINUSE</code></td>
    <td>Indica che l'indirizzo di rete è già in uso.</td>
  </tr>
  <tr>
    <td><code>EADDRNOTAVAIL</code></td>
    <td>Indica che l'indirizzo di rete non è al momento disponibile per    l'uso.</td>
  </tr>
  <tr>
    <td><code>EAFNOSUPPORT</code></td>
    <td>Indica che la famiglia di indirizzi di rete non è supportata.</td>
  </tr>
  <tr>
    <td><code>EAGAIN</code></td>
    <td>Indica che al momento non ci sono dati disponibili e di riprovare l'operazione più tardi.</td>
  </tr>
  <tr>
    <td><code>EALREADY</code></td>
    <td>Indica che il socket ha già una connessione in sospeso in corso.</td>
  </tr>
  <tr>
    <td><code>EBADF</code></td>
    <td>Indica che un file descriptor non è valido.</td>
  </tr>
  <tr>
    <td><code>EBADMSG</code></td>
    <td>Indica un messaggio di dati non valido.</td>
  </tr>
  <tr>
    <td><code>EBUSY</code></td>
    <td>Indica che un dispositivo o una risorsa sono occupati.</td>
  </tr>
  <tr>
    <td><code>ECANCELED</code></td>
    <td>Indica che un operazione è stata annullata.</td>
  </tr>
  <tr>
    <td><code>ECHILD</code></td>
    <td>Indica che non esiste nessun child processes.</td>
  </tr>
  <tr>
    <td><code>ECONNABORTED</code></td>
    <td>Indica che la connessione di rete è stata interrotta.</td>
  </tr>
  <tr>
    <td><code>ECONNREFUSED</code></td>
    <td>Indica che la connessione di rete è stata rifiutata.</td>
  </tr>
  <tr>
    <td><code>ECONNRESET</code></td>
    <td>Indica che la connessione di rete è stata ripristinata.</td>
  </tr>
  <tr>
    <td><code>EDEADLK</code></td>
    <td>Indica che è stato evitato un deadlock delle risorse.</td>
  </tr>
  <tr>
    <td><code>EDESTADDRREQ</code></td>
    <td>Indica che è richiesto un indirizzo di destinazione.</td>
  </tr>
  <tr>
    <td><code>EDOM</code></td>
    <td>Indica che un argomento è fuori dal dominio della funzione.</td>
  </tr>
  <tr>
    <td><code>EDQUOT</code></td>
    <td>Indica che la quota del disco è stata superata.</td>
  </tr>
  <tr>
    <td><code>EEXIST</code></td>
    <td>Indica che il file esiste già.</td>
  </tr>
  <tr>
    <td><code>EFAULT</code></td>
    <td>Indica un indirizzo del puntatore non valido.</td>
  </tr>
  <tr>
    <td><code>EFBIG</code></td>
    <td>Indica che il file è troppo grande.</td>
  </tr>
  <tr>
    <td><code>EHOSTUNREACH</code></td>
    <td>Indica che l'host non è raggiungibile.</td>
  </tr>
  <tr>
    <td><code>EIDRM</code></td>
    <td>Indica che l'identifier è stato rimosso.</td>
  </tr>
  <tr>
    <td><code>EIDRM</code></td>
    <td>Indica una sequenza di byte non valida.</td>
  </tr>
  <tr>
    <td><code>EINPROGRESS</code></td>
    <td>Indica che un'operazione è già in corso.</td>
  </tr>
  <tr>
    <td><code>EINTR</code></td>
    <td>Indica che una chiamata di funzione è stata interrotta.</td>
  </tr>
  <tr>
    <td><code>EINVAL</code></td>
    <td>Indica che è stato fornito un argomento non valido.</td>
  </tr>
  <tr>
    <td><code>EIO</code></td>
    <td>Indica un errore di I/O altrimenti non specificato.</td>
  </tr>
  <tr>
    <td><code>EISCONN</code></td>
    <td>Indica che il socket è connesso.</td>
  </tr>
  <tr>
    <td><code>EISDIR</code></td>
    <td>Indica che il percorso è una directory.</td>
  </tr>
  <tr>
    <td><code>ELOOP</code></td>
    <td>Indica troppi livelli di collegamenti simbolici in un percorso.</td>
  </tr>
  <tr>
    <td><code>EMFILE</code></td>
    <td>Indica che ci sono troppi file aperti.</td>
  </tr>
  <tr>
    <td><code>EMLINK</code></td>
    <td>Indica che ci sono troppi collegamenti fisici a un file.</td>
  </tr>
  <tr>
    <td><code>EMSGSIZE</code></td>
    <td>Indica che il messaggio fornito è troppo lungo.</td>
  </tr>
  <tr>
    <td><code>EMULTIHOP</code></td>
    <td>Indica che è stato tentato un multihop.</td>
  </tr>
  <tr>
    <td><code>ENAMETOOLONG</code></td>
    <td>Indica che il filename è troppo grande.</td>
  </tr>
  <tr>
    <td><code>ENETDOWN</code></td>
    <td>Indica che la rete è inattiva.</td>
  </tr>
  <tr>
    <td><code>ENETRESET</code></td>
    <td>Indica che la connessione è stata interrotta dalla rete.</td>
  </tr>
  <tr>
    <td><code>ENETUNREACH</code></td>
    <td>Indica che il network non è raggiungibile.</td>
  </tr>
  <tr>
    <td><code>ENFILE</code></td>
    <td>Indica troppi file aperti nel sistema.</td>
  </tr>
  <tr>
    <td><code>ENOBUFS</code></td>
    <td>Indica che non è disponibile ulteriore spazio sul buffer.</td>
  </tr>
  <tr>
    <td><code>ENODATA</code></td>
    <td>Indica che nessun messaggio è disponibile sulla coda    di lettura dello stream head.</td>
  </tr>
  <tr>
    <td><code>ENODEV</code></td>
    <td>Indica che non esiste un dispositivo del genere.</td>
  </tr>
  <tr>
    <td><code>ENOENT</code></td>
    <td>Indica che non esiste tale file o directory.</td>
  </tr>
  <tr>
    <td><code>ENOEXEC</code></td>
    <td>Indica un errore di formato exec.</td>
  </tr>
  <tr>
    <td><code>ENOLCK</code></td>
    <td>Indica che non ci sono blocchi disponibili.</td>
  </tr>
  <tr>
    <td><code>ENOLINK</code></td>
    <td>Indicazione che un collegamento è stato interrotto.</td>
  </tr>
  <tr>
    <td><code>ENOMEM</code></td>
    <td>Indica che non c'è abbastanza spazio.</td>
  </tr>
  <tr>
    <td><code>ENOMSG</code></td>
    <td>Indica che non vi è alcun messaggio del tipo desiderato.</td>
  </tr>
  <tr>
    <td><code>ENOPROTOOPT</code></td>
    <td>Indica che un determinato protocollo non è disponibile.</td>
  </tr>
  <tr>
    <td><code>ENOSPC</code></td>
    <td>Indica che non c'è spazio disponibile sul dispositivo.</td>
  </tr>
  <tr>
    <td><code>ENOSR</code></td>
    <td>Indica che non ci sono risorse di streaming disponibili.</td>
  </tr>
  <tr>
    <td><code>ENOSTR</code></td>
    <td>Indica che una determinata risorsa non è uno stream.</td>
  </tr>
  <tr>
    <td><code>ENOSYS</code></td>
    <td>Indica che una funzione non è stata implementata.</td>
  </tr>
  <tr>
    <td><code>ENOTCONN</code></td>
    <td>Indica che il socket non è connesso.</td>
  </tr>
  <tr>
    <td><code>ENOTDIR</code></td>
    <td>Indica che il percorso non è una directory.</td>
  </tr>
  <tr>
    <td><code>ENOTEMPTY</code></td>
    <td>Indica che la directory non è vuota.</td>
  </tr>
  <tr>
    <td><code>ENOTSOCK</code></td>
    <td>Indica che l'elemento considerato non è un socket.</td>
  </tr>
  <tr>
    <td><code>ENOTSUP</code></td>
    <td>Indica che una determinata operazione non è supportata.</td>
  </tr>
  <tr>
    <td><code>ENOTTY</code></td>
    <td>Indica un'operazione di controllo I/O inappropriata.</td>
  </tr>
  <tr>
    <td><code>ENXIO</code></td>
    <td>Indica che non vi è alcun dispositivo o indirizzo di questo tipo.</td>
  </tr>
  <tr>
    <td><code>EOPNOTSUPP</code></td>
    <td>Indica che un operazione non è supportata sul socket.
    Ricorda che mentre `ENOTSUP` e` EOPNOTSUPP` hanno lo stesso valore su Linux, secondo POSIX.1 questi valori di errore dovrebbero essere distinti.)</td>
  </tr>
  <tr>
    <td><code>EOVERFLOW</code></td>
    <td>Indica che un valore è troppo grande per essere memorizzato nel tipo di dati forniti.</td>
  </tr>
  <tr>
    <td><code>EPERM</code></td>
    <td>Indica che l'operazione non è consentita.</td>
  </tr>
  <tr>
    <td><code>EPIPE</code></td>
    <td>Indica una pipe danneggiata.</td>
  </tr>
  <tr>
    <td><code>EPROTO</code></td>
    <td>Indica un errore di protocollo.</td>
  </tr>
  <tr>
    <td><code>EPROTONOSUPPORT</code></td>
    <td>Indica che un protocollo non è supportato.</td>
  </tr>
  <tr>
    <td><code>EPROTOTYPE</code></td>
    <td>Indica il tipo sbagliato di protocollo per un socket.</td>
  </tr>
  <tr>
    <td><code>ERANGE</code></td>
    <td>Indica che i risultati sono troppo grandi.</td>
  </tr>
  <tr>
    <td><code>EROFS</code></td>
    <td>Indica che il file system è di sola lettura.</td>
  </tr>
  <tr>
    <td><code>ESPIPE</code></td>
    <td>Indicates an invalid seek operation.</td>
  </tr>
  <tr>
    <td><code>ESRCH</code></td>
    <td>Indicates that there is no such process.</td>
  </tr>
  <tr>
    <td><code>ESTALE</code></td>
    <td>Indicates that the file handle is stale.</td>
  </tr>
  <tr>
    <td><code>ETIME</code></td>
    <td>Indicates an expired timer.</td>
  </tr>
  <tr>
    <td><code>ETIMEDOUT</code></td>
    <td>Indicates that the connection timed out.</td>
  </tr>
  <tr>
    <td><code>ETXTBSY</code></td>
    <td>Indicates that a text file is busy.</td>
  </tr>
  <tr>
    <td><code>EWOULDBLOCK</code></td>
    <td>Indicates that the operation would block.</td>
  </tr>
  <tr>
    <td><code>EXDEV</code></td>
    <td>Indicates an improper link.
  </tr>
</table>

#### Windows Specific Error Constants

The following error codes are specific to the Windows operating system:

<table>
  <tr>
    <th>Constant</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>WSAEINTR</code></td>
    <td>Indicates an interrupted function call.</td>
  </tr>
  <tr>
    <td><code>WSAEBADF</code></td>
    <td>Indicates an invalid file handle.</td>
  </tr>
  <tr>
    <td><code>WSAEACCES</code></td>
    <td>Indicates insufficient permissions to complete the operation.</td>
  </tr>
  <tr>
    <td><code>WSAEFAULT</code></td>
    <td>Indicates an invalid pointer address.</td>
  </tr>
  <tr>
    <td><code>WSAEINVAL</code></td>
    <td>Indicates that an invalid argument was passed.</td>
  </tr>
  <tr>
    <td><code>WSAEMFILE</code></td>
    <td>Indicates that there are too many open files.</td>
  </tr>
  <tr>
    <td><code>WSAEWOULDBLOCK</code></td>
    <td>Indicates that a resource is temporarily unavailable.</td>
  </tr>
  <tr>
    <td><code>WSAEINPROGRESS</code></td>
    <td>Indicates that an operation is currently in progress.</td>
  </tr>
  <tr>
    <td><code>WSAEALREADY</code></td>
    <td>Indicates that an operation is already in progress.</td>
  </tr>
  <tr>
    <td><code>WSAENOTSOCK</code></td>
    <td>Indicates that the resource is not a socket.</td>
  </tr>
  <tr>
    <td><code>WSAEDESTADDRREQ</code></td>
    <td>Indicates that a destination address is required.</td>
  </tr>
  <tr>
    <td><code>WSAEMSGSIZE</code></td>
    <td>Indicates that the message size is too long.</td>
  </tr>
  <tr>
    <td><code>WSAEPROTOTYPE</code></td>
    <td>Indicates the wrong protocol type for the socket.</td>
  </tr>
  <tr>
    <td><code>WSAENOPROTOOPT</code></td>
    <td>Indicates a bad protocol option.</td>
  </tr>
  <tr>
    <td><code>WSAEPROTONOSUPPORT</code></td>
    <td>Indicates that the protocol is not supported.</td>
  </tr>
  <tr>
    <td><code>WSAESOCKTNOSUPPORT</code></td>
    <td>Indicates that the socket type is not supported.</td>
  </tr>
  <tr>
    <td><code>WSAEOPNOTSUPP</code></td>
    <td>Indicates that the operation is not supported.</td>
  </tr>
  <tr>
    <td><code>WSAEPFNOSUPPORT</code></td>
    <td>Indicates that the protocol family is not supported.</td>
  </tr>
  <tr>
    <td><code>WSAEAFNOSUPPORT</code></td>
    <td>Indicates that the address family is not supported.</td>
  </tr>
  <tr>
    <td><code>WSAEADDRINUSE</code></td>
    <td>Indicates that the network address is already in use.</td>
  </tr>
  <tr>
    <td><code>WSAEADDRNOTAVAIL</code></td>
    <td>Indicates that the network address is not available.</td>
  </tr>
  <tr>
    <td><code>WSAENETDOWN</code></td>
    <td>Indicates that the network is down.</td>
  </tr>
  <tr>
    <td><code>WSAENETUNREACH</code></td>
    <td>Indicates that the network is unreachable.</td>
  </tr>
  <tr>
    <td><code>WSAENETRESET</code></td>
    <td>Indicates that the network connection has been reset.</td>
  </tr>
  <tr>
    <td><code>WSAECONNABORTED</code></td>
    <td>Indicates that the connection has been aborted.</td>
  </tr>
  <tr>
    <td><code>WSAECONNRESET</code></td>
    <td>Indicates that the connection has been reset by the peer.</td>
  </tr>
  <tr>
    <td><code>WSAENOBUFS</code></td>
    <td>Indicates that there is no buffer space available.</td>
  </tr>
  <tr>
    <td><code>WSAEISCONN</code></td>
    <td>Indicates that the socket is already connected.</td>
  </tr>
  <tr>
    <td><code>WSAENOTCONN</code></td>
    <td>Indicates that the socket is not connected.</td>
  </tr>
  <tr>
    <td><code>WSAESHUTDOWN</code></td>
    <td>Indicates that data cannot be sent after the socket has been
    shutdown.</td>
  </tr>
  <tr>
    <td><code>WSAETOOMANYREFS</code></td>
    <td>Indicates that there are too many references.</td>
  </tr>
  <tr>
    <td><code>WSAETIMEDOUT</code></td>
    <td>Indicates that the connection has timed out.</td>
  </tr>
  <tr>
    <td><code>WSAECONNREFUSED</code></td>
    <td>Indicates that the connection has been refused.</td>
  </tr>
  <tr>
    <td><code>WSAELOOP</code></td>
    <td>Indicates that a name cannot be translated.</td>
  </tr>
  <tr>
    <td><code>WSAENAMETOOLONG</code></td>
    <td>Indicates that a name was too long.</td>
  </tr>
  <tr>
    <td><code>WSAEHOSTDOWN</code></td>
    <td>Indicates that a network host is down.</td>
  </tr>
  <tr>
    <td><code>WSAEHOSTUNREACH</code></td>
    <td>Indicates that there is no route to a network host.</td>
  </tr>
  <tr>
    <td><code>WSAENOTEMPTY</code></td>
    <td>Indicates that the directory is not empty.</td>
  </tr>
  <tr>
    <td><code>WSAEPROCLIM</code></td>
    <td>Indicates that there are too many processes.</td>
  </tr>
  <tr>
    <td><code>WSAEUSERS</code></td>
    <td>Indicates that the user quota has been exceeded.</td>
  </tr>
  <tr>
    <td><code>WSAEDQUOT</code></td>
    <td>Indicates that the disk quota has been exceeded.</td>
  </tr>
  <tr>
    <td><code>WSAESTALE</code></td>
    <td>Indicates a stale file handle reference.</td>
  </tr>
  <tr>
    <td><code>WSAEREMOTE</code></td>
    <td>Indicates that the item is remote.</td>
  </tr>
  <tr>
    <td><code>WSASYSNOTREADY</code></td>
    <td>Indicates that the network subsystem is not ready.</td>
  </tr>
  <tr>
    <td><code>WSAVERNOTSUPPORTED</code></td>
    <td>Indicates that the `winsock.dll` version is out of range.</td>
  </tr>
  <tr>
    <td><code>WSANOTINITIALISED</code></td>
    <td>Indicates that successful WSAStartup has not yet been performed.</td>
  </tr>
  <tr>
    <td><code>WSAEDISCON</code></td>
    <td>Indicates that a graceful shutdown is in progress.</td>
  </tr>
  <tr>
    <td><code>WSAENOMORE</code></td>
    <td>Indicates that there are no more results.</td>
  </tr>
  <tr>
    <td><code>WSAECANCELLED</code></td>
    <td>Indicates that an operation has been canceled.</td>
  </tr>
  <tr>
    <td><code>WSAEINVALIDPROCTABLE</code></td>
    <td>Indicates that the procedure call table is invalid.</td>
  </tr>
  <tr>
    <td><code>WSAEINVALIDPROVIDER</code></td>
    <td>Indicates an invalid service provider.</td>
  </tr>
  <tr>
    <td><code>WSAEPROVIDERFAILEDINIT</code></td>
    <td>Indicates that the service provider failed to initialized.</td>
  </tr>
  <tr>
    <td><code>WSASYSCALLFAILURE</code></td>
    <td>Indicates a system call failure.</td>
  </tr>
  <tr>
    <td><code>WSASERVICE_NOT_FOUND</code></td>
    <td>Indicates that a service was not found.</td>
  </tr>
  <tr>
    <td><code>WSATYPE_NOT_FOUND</code></td>
    <td>Indicates that a class type was not found.</td>
  </tr>
  <tr>
    <td><code>WSA_E_NO_MORE</code></td>
    <td>Indicates that there are no more results.</td>
  </tr>
  <tr>
    <td><code>WSA_E_CANCELLED</code></td>
    <td>Indicates that the call was canceled.</td>
  </tr>
  <tr>
    <td><code>WSAEREFUSED</code></td>
    <td>Indicates that a database query was refused.</td>
  </tr>
</table>

### dlopen Constants

If available on the operating system, the following constants are exported in `os.constants.dlopen`. See dlopen(3) for detailed information.

<table>
  <tr>
    <th>Constant</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>RTLD_LAZY</code></td>
    <td>Perform lazy binding. Node.js sets this flag by default.</td>
  </tr>
  <tr>
    <td><code>RTLD_NOW</code></td>
    <td>Resolve all undefined symbols in the library before dlopen(3)
    returns.</td>
  </tr>
  <tr>
    <td><code>RTLD_GLOBAL</code></td>
    <td>Symbols defined by the library will be made available for symbol
    resolution of subsequently loaded libraries.</td>
  </tr>
  <tr>
    <td><code>RTLD_LOCAL</code></td>
    <td>The converse of `RTLD_GLOBAL`. This is the default behavior if neither
    flag is specified.</td>
  </tr>
  <tr>
    <td><code>RTLD_DEEPBIND</code></td>
    <td>Make a self-contained library use its own symbols in preference to
    symbols from previously loaded libraries.</td>
  </tr>
</table>

### libuv Constants

<table>
  <tr>
    <th>Constant</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>UV_UDP_REUSEADDR</code></td>
    <td></td>
  </tr>
</table>
