# Child Process

<!--introduced_in=v0.10.0-->

<!--lint disable maximum-line-length-->

> Stabilità: 2 - Stabile

Il modulo `child_process` offre la possibilità di generare processi child in modo simile, ma non identico, a quello di popen(3). Questa capacità è fornita principalmente dalla funzione [`child_process.spawn()`][]:

```js
const { spawn } = require('child_process');
const ls = spawn('ls', ['-lh', '/usr']);

ls.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

ls.stderr.on('data', (data) => {
  console.log(`stderr: ${data}`);
});

ls.on('close', (code) => {
  console.log(`child process exited with code ${code}`);
});
```

Di default, i pipe per `stdin`, `stdout`, ed `stderr` vengono stabiliti tra il processo parent di Node.js e il child generato. Questi pipe hanno capacità limitata (e specifica per ogni piattaforma). Se il processo child scrive su stdout eccedendo quel limite senza che l'output venga catturato, il processo child si interromperà in attesa che il buffer del pipe accetti più dati. Questo è identico al comportamento dei pipe nella shell. Utilizza l'opzione `{ stdio: 'ignore' }` se l'output non verrà utilizzato.

Il metodo [`child_process.spawn()`][] genera il processo child in modo asincrono, senza bloccare il ciclo degli eventi Node.js. La funzione [`child_process.spawnSync()`][] fornisce funzionalità equivalenti in un modo sincrono che interrompe il ciclo di eventi finché il processo generato non si conclude o viene chiuso.

Per comodità, il modulo `child_process` fornisce una manciata di alternative sincrone e asincrone a [`child_process.spawn()`][] e [`child_process.spawnSync()`][]. *Da notare che ognuna di queste alternative è implementata in cima a [`child_process.spawn()`][] o [`child_process.spawnSync()`][].*

    * [`child_process.exec()`][]: genera una shell ed esegue un comando al suo interno, 
      passando lo `stdout` e lo `stderr` a una funzione di callback quando è completa.
    * [`child_process.execFile()`][]: simile a [`child_process.exec()`][] eccetto che 
      genera il comando direttamente senza prima generare una shell di default.
    * [`child_process.fork()`][]: genera un nuovo processo Node.js e invoca un 
      modulo specificato con un canale di comunicazione IPC che consente
      di inviare messaggi tra parent e child.
    * [`child_process.execSync()`][]: una versione sincrona di
      [`child_process.exec()`][] che *interromperà* il ciclo di eventi di Node.js.
    * [`child_process.execFileSync()`][]: una versione sincrona di
      [`child_process.execFile()`][] che *interromperà* il ciclo di eventi di Node.js.
    

Per alcuni casi d'utilizzo, come l'automazione degli script della shell, le [controparti sincrone](#child_process_synchronous_process_creation) possono essere più convenienti. Tuttavia, in molti casi, i metodi sincroni possono avere un impatto significativo sulle prestazioni a causa dell'interruzione del ciclo degli eventi durante il completamento dei processi generati.

## Creazione di Processi Asincroni

I metodi [`child_process.spawn()`][], [`child_process.fork()`][], [`child_process.exec()`][], e [`child_process.execFile()`][] seguono tutti il modello di programmazione asincrona idiomatica tipico di altre API di Node.js.

Ciascun metodo restituisce un'istanza [`ChildProcess`][]. Questi objects implementano l'API Node.js [`EventEmitter`][], consentendo al processo parent di registrare le funzioni listener chiamate quando si verificano determinati eventi durante il ciclo del processo child.

I metodi [`child_process.exec()`][] e [`child_process.execFile()`][] consentono inoltre di specificare una funzione `callback` opzionale invocata al termine del processo child.

### Generare i file `.bat` e `.cmd` su Windows

L'importanza della distinzione tra [`child_process.exec()`][] e [`child_process.execFile()`][] può variare in base alla piattaforma. Sui sistemi operativi di tipo Unix (Unix, Linux, macOS) [`child_process.execFile()`][] può essere più efficiente perché non genera una shell di default. Su Windows, tuttavia, i file `.bat` e `.cmd` non sono eseguibili da soli senza un terminale e, pertanto, non possono essere avviati utilizzando [`child_process.execFile()`][]. Quando si eseguono su Windows, i file `.bat` e `.cmd` possono essere invocati usando [`child_process.spawn()`][] con il set di opzioni `shell`, con [`child_process.exec()`][], oppure generando `cmd.exe` e passando i file `.bat` o `.cmd` come argomento (che è ciò che fanno le opzioni `shell` e [`child_process.exec()`][]). In ogni caso, se il filename dello script contiene spazi, deve essere racchiuso tra le virgolette.

```js
// Solo su Windows ...
const { spawn } = require('child_process');
const bat = spawn('cmd.exe', ['/c', 'my.bat']);

bat.stdout.on('data', (data) => {
  console.log(data.toString());
});

bat.stderr.on('data', (data) => {
  console.log(data.toString());
});

bat.on('exit', (code) => {
  console.log(`Child exited with code ${code}`);
});
```

```js
// OPPURE...
const { exec } = require('child_process');
exec('my.bat', (err, stdout, stderr) => {
  if (err) {
    console.error(err);
    return;
  }
  console.log(stdout);
});

// Script con spazi nel filename:
const bat = spawn('"my script.cmd"', ['a', 'b'], { shell: true });
// oppure:
exec('"my script.cmd" a b', (err, stdout, stderr) => {
  // ...
});
```

### child_process.exec(command\[, options\]\[, callback\])

<!-- YAML
added: v0.1.90
changes:

  - version: v8.8.0
    pr-url: https://github.com/nodejs/node/pull/15380
    description: The `windowsHide` option is supported now.
-->

* `command` {string} Il comando da eseguire, con argomenti separati dallo spazio.
* `options` {Object} 
  * `cwd` {string} Attuale directory di lavoro del processo child. **Default:** `null`.
  * `env` {Object} Coppie key-value dell'ambiente. **Default:** `null`.
  * `encoding` {string} **Default:** `'utf8'`
  * `shell` {string} La Shell con la quale eseguire il comando. Vedi [Requisiti della Shell](#child_process_shell_requirements) e [Shell Default di Windows](#child_process_default_windows_shell). **Default:** `'/bin/sh'` su UNIX, `process.env.ComSpec` su Windows.
  * `timeout` {number} **Default:** `0`
  * `maxBuffer` {number} La quantità maggiore di dati in byte consentiti su stdout o stderr. Se superata, il processo child viene concluso. Vedi avvertenze su [`maxBuffer` and Unicode][]. **Default:** `200 * 1024`.
  * `killSignal` {string|integer} **Default:** `'SIGTERM'`
  * `uid` {number} Imposta l'identità dell'utente (user identity) del processo (vedi setuid(2)).
  * `gid` {number} Imposta l'identità di gruppo (group identity) del processo (vedi setgid(2)).
  * `windowsHide` {boolean} Nasconde la finestra della console di sottoprocesso che verrebbe normalmente creata sui sistemi Windows. **Default:** `false`.
* `callback` {Function} chiamata con l'output al termine del processo. 
  * `error` {Error}
  * `stdout` {string|Buffer}
  * `stderr` {string|Buffer}
* Restituisce: {ChildProcess}

Genera una shell e successivamente esegue il `command` all'interno della shell stessa, eseguendo il buffer di qualsiasi output generato. La stringa `command` passata alla funzione exec viene elaborata direttamente dalla shell e i caratteri speciali (che variano in base alla [shell](https://en.wikipedia.org/wiki/List_of_command-line_interpreters)) devono essere trattati di conseguenza:

```js
exec('"/path/to/test file/test.sh" arg1 arg2');
// Le virgolette vengono utilizzate in modo che lo spazio nel percorso (path) non venga interpretato come
// più argomenti

exec('echo "The \\$HOME variable is $HOME"');
// La variabile $HOME è sottoposta all'escape nella prima istanza, ma non nella seconda
```

**Non passare mai l'input unsanitized user a questa funzione. Qualsiasi input contenente metacaratteri della shell può essere utilizzato per attivare l'esecuzione arbitraria dei comandi.**

```js
const { exec } = require('child_process');
exec('cat *.js bad_file | wc -l', (error, stdout, stderr) => {
  if (error) {
    console.error(`exec error: ${error}`);
    return;
  }
  console.log(`stdout: ${stdout}`);
  console.log(`stderr: ${stderr}`);
});
```

Se viene fornita una funzione `callback`, viene chiamata con gli argomenti `(error, stdout, stderr)`. In caso di successo, `error` sarà `null`. In caso di errore, `error` sarà un'istanza di [`Error`][]. La proprietà `error.code` sarà il valore di uscita del processo child mentre `error.signal` sarà impostato sul segnale che ha interrotto il processo. Qualsiasi valore di uscita diverso da `0` è considerato un errore.

Gli argomenti `stdout` e `stderr` passati al callback conterranno gli output stdout e stderr del processo child. Di default, Node.js decodificherà l'output come UTF-8 e passerà le stringhe al callback. L'opzione `encoding` può essere usata per specificare l'encoding (codifica) dei caratteri utilizzato per decodificare gli output stdout e stderr. Se `encoding` è `'buffer'`, oppure un econding dei caratteri non riconosciuto, allora i `Buffer` object saranno passati al callback.

Se `timeout` è maggiore di `0`, il parent invierà il segnale identificato dalla proprietà `killSignal` (il valore predefinito è `'SIGTERM'`) se il child esegue più di `timeout` millisecondi.

A differenza della chiamata di sistema POSIX exec(3), `child_process.exec()` non sostituisce il processo esistente e utilizza una shell per eseguire il comando.

Se questo metodo viene invocato come sua versione [`util.promisify()`][], restituisce un `Promise` per un `Object` con le proprietà `stdout` e `stderr`. In caso di errore (incluso qualsiasi errore che dia come risultato un valore di uscita diverso da 0), viene restituito un promise respinto, con lo stesso `error` object specificato nel callback, ma con altre due proprietà `stdout` e `stderr`.

```js
const util = require('util');
const exec = util.promisify(require('child_process').exec);

async function lsExample() {
  const { stdout, stderr } = await exec('ls');
  console.log('stdout:', stdout);
  console.log('stderr:', stderr);
}
lsExample();
```

### child_process.execFile(file\[, args\]\[, options\][, callback])

<!-- YAML
added: v0.1.91
changes:

  - version: v8.8.0
    pr-url: https://github.com/nodejs/node/pull/15380
    description: The `windowsHide` option is supported now.
-->

* `file` {string} Il nome o il percorso del file eseguibile da avviare.
* `args` {string[]} Elenco degli argomenti di string.
* `options` {Object} 
  * `cwd` {string} Attuale directory di lavoro del processo child.
  * `env` {Object} Coppie key-value dell'ambiente.
  * `encoding` {string} **Default:** `'utf8'`
  * `timeout` {number} **Default:** `0`
  * `maxBuffer` {number} La quantità massima di dati in byte consentiti su stdout o stderr. Se superata, il processo child viene concluso. Vedi avvertenze su [`maxBuffer` and Unicode][]. **Default:** `200 * 1024`.
  * `killSignal` {string|integer} **Default:** `'SIGTERM'`
  * `uid` {number} Imposta l'identità dell'utente (user identity) del processo (vedi setuid(2)).
  * `gid` {number} Imposta l'identità di gruppo (group identity) del processo (vedi setgid(2)).
  * `windowsHide` {boolean} Nasconde la finestra della console di sottoprocesso che verrebbe normalmente creata sui sistemi Windows. **Default:** `false`.
  * `windowsVerbatimArguments` {boolean} Non viene eseguita nessuna citazione od escaping degli argomenti su Windows. Ignorato su Unix. **Default:** `false`.
  * `shell` {boolean|string} Se `true`, esegue `command` all'interno di una shell. Utilizza `'/bin/sh'` su UNIX, e `process.env.ComSpec` su Windows. Una shell diversa può essere specificata come una stringa. Vedi [Requisiti della Shell](#child_process_shell_requirements) e [Shell Default di Windows](#child_process_default_windows_shell). **Default:** `false` (nessuna shell).
* `callback` {Function} Chiamata con l'output al termine del processo. 
  * `error` {Error}
  * `stdout` {string|Buffer}
  * `stderr` {string|Buffer}
* Restituisce: {ChildProcess}

La funzione `child_process.execFile()` è simile a [`child_process.exec()`][] eccetto per il fatto che non genera una shell di default. Piuttosto, il `file` eseguibile specificato viene generato direttamente come un nuovo processo rendendolo leggermente più efficiente di [`child_process.exec()`][].

Sono supportate le stesse opzioni di [`child_process.exec()`][]. Dal momento che non viene generata una shell, i comportamenti come il reindirizzamento I/O e il file globbing non sono supportati.

```js
const { execFile } = require('child_process');
const child = execFile('node', ['--version'], (error, stdout, stderr) => {
  if (error) {
    throw error;
  }
  console.log(stdout);
});
```

Gli argomenti `stdout` e `stderr` passati al callback conterranno gli output stdout e stderr del processo child. Di default, Node.js decodificherà l'output come UTF-8 e passerà le stringhe al callback. L'opzione `encoding` può essere usata per specificare l'encoding (codifica) dei caratteri utilizzato per decodificare gli output stdout e stderr. Se `encoding` è `'buffer'`, oppure un econding dei caratteri non riconosciuto, allora i `Buffer` object saranno passati al callback.

Se questo metodo viene invocato come sua versione [`util.promisify()`][], restituisce un `Promise` per un `Object` con le proprietà `stdout` e `stderr`. In caso di errore (incluso qualsiasi errore che dia come risultato un valore di uscita diverso da 0), viene restituito un promise respinto, con lo stesso `error` object specificato nel callback, ma con altre due proprietà `stdout` e `stderr`.

```js
const util = require('util');
const execFile = util.promisify(require('child_process').execFile);
async function getVersion() {
  const { stdout } = await execFile('node', ['--version']);
  console.log(stdout);
}
getVersion();
```

**Se l'opzione `shell` è abilitata, non passare l'input unsanitized user a questa funzione. Qualsiasi input contenente metacaratteri della shell può essere utilizzato per attivare l'esecuzione arbitraria dei comandi.**

### child_process.fork(modulePath\[, args\]\[, options\])

<!-- YAML
added: v0.5.0
changes:

  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10866
    description: The `stdio` option can now be a string.
  - version: v6.4.0
    pr-url: https://github.com/nodejs/node/pull/7811
    description: The `stdio` option is supported now.
-->

* `modulePath` {string} Il modulo da eseguire nel child.
* `args` {string[]} Elenco degli argomenti di string.
* `options` {Object} 
  * `cwd` {string} Attuale directory di lavoro del processo child.
  * `env` {Object} Coppie key-value dell'ambiente.
  * `execPath` {string} Eseguibile utilizzato per creare il processo child.
  * `execArgv` {string[]} Elenco degli argomenti di string passati all'eseguibile. **Default:** `process.execArgv`.
  * `silent` {boolean} Se `true`, stdin, stdout e stderr del child verranno reindirizzati al parent tramite il piping, in caso contrario verranno ereditati dal parent, per maggiori dettagli vedi le opzioni `'pipe'` e `'inherit'` per lo [`stdio`][] di [`child_process.spawn()`][]. **Default:** `false`.
  * `stdio` {Array|string} Vedi lo [`stdio`][] di [`child_process.spawn()`][]. Quando viene fornita quest'opzione, esegue l'override di `silent`. Se viene utilizzata la variante dell'array, deve contenere esattamente un elemento con valore `'ipc'` oppure verrà generato un errore. Per esempio `[0, 1, 2, 'ipc']`.
  * `windowsVerbatimArguments` {boolean} Non viene eseguita nessuna citazione od escaping degli argomenti su Windows. Ignorato su Unix. **Default:** `false`.
  * `uid` {number} Imposta l'identità dell'utente (user identity) del processo (vedi setuid(2)).
  * `gid` {number} Imposta l'identità di gruppo (group identity) del processo (vedi setgid(2)).
* Restituisce: {ChildProcess}

Il metodo `child_process.fork()` è un caso speciale di [`child_process.spawn()`][] utilizzato specificamente per generare nuovi processi Node.js. Così come con [`child_process.spawn()`][], viene restituito un [`ChildProcess`][] object. Il [`ChildProcess`][] restituito avrà un canale di comunicazione aggiuntivo integrato che consente di passare i messaggi avanti e indietro tra parent e child. Vedi [`subprocess.send()`][] per i dettagli.

È importante tenere presente che i processi child di Node.js generati sono indipendenti dal parent con l'eccezione del canale di comunicazione IPC stabilito tra i due. Ogni processo ha la propria memoria, con le proprie istanze V8. A causa delle allocazioni aggiuntive richieste, non è consigliato generare un numero elevato di processi child di Node.js.

Di default, `child_process.fork()` genererà nuove istanze Node.js utilizzando il [`process.execPath`][] del processo parent. La proprietà `execPath` nell'`options` object consente di utilizzare un percorso di esecuzione alternativo.

I processi Node.js avviati con un `execPath` personalizzato comunicano con il processo parent utilizzando il file descriptor (fd) identificato utilizzando la variabile di ambiente `NODE_CHANNEL_FD` sul processo child.

A differenza della chiamata di sistema POSIX fork(2), `child_process.fork()` non clona il processo corrente.

L'opzione `shell` disponibile in [`child_process.spawn()`][] non è supportata da `child_process.fork()` e verrà ignorata se impostata.

### child_process.spawn(command\[, args\]\[, options\])

<!-- YAML
added: v0.1.90
changes:

  - version: v8.8.0
    pr-url: https://github.com/nodejs/node/pull/15380
    description: The `windowsHide` option is supported now.
  - version: v6.4.0
    pr-url: https://github.com/nodejs/node/pull/7696
    description: The `argv0` option is supported now.
  - version: v5.7.0
    pr-url: https://github.com/nodejs/node/pull/4598
    description: The `shell` option is supported now.
-->

* `command` {string} Il comando da eseguire.
* `args` {string[]} Elenco degli argomenti di string.
* `options` {Object} 
  * `cwd` {string} Attuale directory di lavoro del processo child.
  * `env` {Object} Coppie key-value dell'ambiente.
  * `argv0` {string} Imposta esplicitamente il valore di `argv[0]` inviato al processo child. Questo sarà impostato su `command` se non specificato.
  * `stdio` {Array|string} Configurazione stdio del child (vedi [`options.stdio`][`stdio`]).
  * `detached` {boolean} Prepara il child all'avvio indipendentemente dal suo processo parent. Il comportamento specifico dipende dalla piattaforma, vedi [`options.detached`][].
  * `uid` {number} Imposta l'identità dell'utente (user identity) del processo (vedi setuid(2)).
  * `gid` {number} Imposta l'identità di gruppo (group identity) del processo (vedi setgid(2)).
  * `shell` {boolean|string} Se `true`, esegue `command` all'interno di una shell. Utilizza `'/bin/sh'` su UNIX, e `process.env.ComSpec` su Windows. Una shell diversa può essere specificata come una stringa. Vedi [Requisiti della Shell](#child_process_shell_requirements) e [Shell Default di Windows](#child_process_default_windows_shell). **Default:** `false` (nessuna shell).
  * `windowsVerbatimArguments` {boolean} Non viene eseguita nessuna citazione od escaping degli argomenti su Windows. Ignorato su Unix. È impostato su `true` automaticamente quando la `shell` è specificata. **Default:** `false`.
  * `windowsHide` {boolean} Nasconde la finestra della console di sottoprocesso che verrebbe normalmente creata sui sistemi Windows. **Default:** `false`.
* Restituisce: {ChildProcess}

Il metodo `child_process.spawn()` genera un nuovo processo utilizzando il `command` dato, con gli argomenti della command line in `args`. Se omesso, `args` si imposta automaticamente su un array vuoto.

**Se l'opzione `shell` è abilitata, non passare l'input unsanitized user a questa funzione. Qualsiasi input contenente metacaratteri della shell può essere utilizzato per attivare l'esecuzione arbitraria dei comandi.**

Un terzo argomento può essere usato per specificare ulteriori opzioni, con questi valori di default:

```js
const defaults = {
  cwd: undefined,
  env: process.env
};
```

Utilizza `cwd` per specificare la directory di lavoro dalla quale viene generato il processo. Se non specificata, l'impostazione predefinita consiste nell'ereditare l'attuale directory di lavoro.

Utilizza `env` per specificare le variabili di ambiente che saranno visibili per il nuovo processo, il valore predefinito è [`process.env`][].

I valori `undefined` in `env` saranno ignorati.

Esempio di esecuzione di `ls -lh /usr`, acquisizione di `stdout`, `stderr`, e del valore di uscita:

```js
const { spawn } = require('child_process');
const ls = spawn('ls', ['-lh', '/usr']);

ls.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

ls.stderr.on('data', (data) => {
  console.log(`stderr: ${data}`);
});

ls.on('close', (code) => {
  console.log(`child process exited with code ${code}`);
});
```

Esempio: Un modo molto elaborato per eseguire `ps ax | grep ssh`

```js
const { spawn } = require('child_process');
const ps = spawn('ps', ['ax']);
const grep = spawn('grep', ['ssh']);

ps.stdout.on('data', (data) => {
  grep.stdin.write(data);
});

ps.stderr.on('data', (data) => {
  console.log(`ps stderr: ${data}`);
});

ps.on('close', (code) => {
  if (code !== 0) {
    console.log(`ps process exited with code ${code}`);
  }
  grep.stdin.end();
});

grep.stdout.on('data', (data) => {
  console.log(data.toString());
});

grep.stderr.on('data', (data) => {
  console.log(`grep stderr: ${data}`);
});

grep.on('close', (code) => {
  if (code !== 0) {
    console.log(`grep process exited with code ${code}`);
  }
});
```

Esempio di controllo per `spawn` fallito:

```js
const { spawn } = require('child_process');
const subprocess = spawn('bad_command');

subprocess.on('error', (err) => {
  console.log('Failed to start subprocess.');
});
```

Alcune piattaforme (macOS, Linux) utilizzeranno il valore di `argv[0]` per il titolo del processo, mentre altri (Windows, SunOS) useranno `command`.

Attualmente Node.js sovrascrive `argv[0]` con `process.execPath` all'avvio, quindi il `process.argv[0]` all'interno di un processo child di Node.js non corrisponderà al parametro `argv0` passato a `spawn` dal parent, perciò va recuperato con la proprietà `process.argv0`.

#### options.detached

<!-- YAML
added: v0.7.10
-->

Su Windows, l'impostazione di `options.detached` su `true` consente al processo child di continuare ad essere eseguito dopo la chiusura del parent. Il child avrà la propria finestra della console. *Una volta abilitato per un processo child, non può essere disabilitato*.

Su piattaforme diverse da Windows, se `options.detached` è impostato su `true`, allora il processo child sarà reso il leader di un nuovo gruppo di processi e di sessione. Da notare che i processi child possono continuare ad essere eseguiti dopo che il parent si è concluso indipendentemente dal fatto che siano stati distaccati o meno. Vedi setsid(2) per maggiori informazioni.

Di default, il parent aspetterà che il child distaccato si concluda. Per prevenire che il parent attenda un dato `subprocess`, utilizza il metodo `subprocess.unref()`. Così facendo, il ciclo di eventi del parent non includerà il child nel suo reference count, consentendo al parent di concludere indipendentemente dal child, a meno che non vi sia un canale IPC stabilito tra child e parent.

Quando si utilizza l'opzione `detached` per avviare un processo di lunga durata, il processo non verrà eseguito in background dopo la conclusione del parent a meno che non sia fornito di una configurazione `stdio` che non sia collegata al parent. Se lo `stdio` del parent è ereditato, il child rimarrà collegato al terminale di controllo.

Esempio di un processo ad esecuzione prolungata, scollegando ed ignorando anche i file descriptor `stdio` del suo parent, in modo da ignorare la terminazione del parent stesso:

```js
const { spawn } = require('child_process');

const subprocess = spawn(process.argv[0], ['child_program.js'], {
  detached: true,
  stdio: 'ignore'
});

subprocess.unref();
```

In alternativa, è possibile reindirizzare l'output del processo child all'interno dei file:

```js
const fs = require('fs');
const { spawn } = require('child_process');
const out = fs.openSync('./out.log', 'a');
const err = fs.openSync('./out.log', 'a');

const subprocess = spawn('prg', [], {
  detached: true,
  stdio: [ 'ignore', out, err ]
});

subprocess.unref();
```

#### options.stdio

<!-- YAML
added: v0.7.10
changes:

  - version: v3.3.1
    pr-url: https://github.com/nodejs/node/pull/2727
    description: The value `0` is now accepted as a file descriptor.
-->

L'opzione `options.stdio` viene utilizzata per configurare i pipe stabiliti tra i processi parent e child. Di default, lo stdin, lo stdout e lo stderr del child vengono reindirizzati ai corrispondenti stream [`subprocess.stdin`][], [`subprocess.stdout`][], e [`subprocess.stderr`][] sul [`ChildProcess`][] object. Ciò è equivalente all'impostazione di `options.stdio` uguale a `['pipe', 'pipe', 'pipe']`.

Per comodità, `options.stdio` potrebbe essere una delle seguenti stringhe:

* `'pipe'` - equivalente a `['pipe', 'pipe', 'pipe']` (il valore predefinito)
* `'ignore'` - equivalente a `['ignore', 'ignore', 'ignore']`
* `'inherit'` - equivalente a `[process.stdin, process.stdout, process.stderr]` oppure a `[0,1,2]`

In caso contrario, il valore di `options.stdio` è un array in cui ogni indice corrisponde a un file descriptor (fd) all'interno del child. I file descriptor (fd) 0, 1 e 2 corrispondono rispettivamente a stdin, stdout e stderr. È possibile specificare ulteriori file descriptor (fd) per creare pipe aggiuntivi tra parent e child. Il valore è uno dei seguenti:

1. `'pipe'` - Crea un pipe tra i processi child e parent. L'estremità parent del pipe è esposta al parent come una proprietà sul `child_process` object come un [`subprocess.stdio[fd]`][`stdio`]. I pipe creati per i file descriptor (fd) 0 - 2 sono anche disponibili, rispettivamente, come [`subprocess.stdin`][], [`subprocess.stdout`][] e [`subprocess.stderr`][].
2. `'ipc'` - Crea un canale IPC per far passare i messaggi/file descriptor tra parent e child. Un [`ChildProcess`][] può avere al massimo *un* file descriptor IPC stdio. L'impostazione di quest'opzione abilita il metodo [`subprocess.send()`][]. Se il child è un processo Node.js, la presenza di un canale IPC abiliterà i metodi [`process.send()`][] e [`process.disconnect()`][], nonché gli eventi [`'disconnect'`][] e [`'message'`][] all'interno del child stesso.
  
  L'accesso al file descriptor del canale IPC in un modo diverso dall'utilizzo di [`process.send()`][] oppure l'utilizzo del canale IPC con un processo child che non è un'istanza Node.js, non sono supportati.

3. `'ignore'` - Dà a Node.js l'istruzione di ignorare il file descriptor all'interno del child. Sebbene Node.js aprirà sempre i file descriptor 0 - 2 per i processi che genera, l'impostazione del file descriptor su `'ignore'` farà in modo che Node.js apra `/dev/null` e lo colleghi al file descriptor del child.

4. {Stream} object - Condivide uno stream readable o writable che fa riferimento ad un tty, un file, un socket o un pipe con il processo child. Il file descriptor sottostante allo stream è duplicato nel processo child nel file descriptor corrispondente all'indice nell'array `stdio`. Da notare che lo stream deve avere un descriptor sottostante (gli stream dei file non entrano in azione finché non si verifica l'evento `'open'`).
5. Positive integer - Il valore integer (intero) viene interpretato come un file descriptor attualmente aperto nel processo parent. È condiviso con il processo child, in modo simile alla modalità di condivisione degli {Stream} object.
6. `null`, `undefined` - Utilizzano il valore predefinito. Per i file descriptor stdio 0, 1 e 2 (in altre parole, stdin, stdout e stderr) viene creato un pipe. Per i file descriptor 3 o superiori, l'impostazione predefinita è `'ignore'`.

Esempio:

```js
const { spawn } = require('child_process');

// Il child utilizzerà gli stdio del parent
spawn('prg', [], { stdio: 'inherit' });

// Genera un child che condivide solo stderr
spawn('prg', [], { stdio: ['pipe', 'pipe', process.stderr] });

// Apre un extra fd = 4, per interagire con i programmi che presentano 
// un'interfaccia in stile startd.
spawn('prg', [], { stdio: ['pipe', null, null, null, 'pipe'] });
```

*Vale la pena notare che quando viene stabilito un canale IPC tra i processi parent e child, e il child è un processo Node.js, allora esso viene avviato con il canale IPC unreferenced (ovvero senza riferimento, utilizzando `unref()`) fino a quando registra un event handler per l'evento [`'disconnect'`][] o per l'evento [`'message'`][]. Ciò consente al child di concludersi normalmente senza che il processo venga tenuto aperto dal canale IPC aperto.*

Vedi anche: [`child_process.exec()`][] e [`child_process.fork()`][].

## Creazione di Processi Sincroni

I metodi [`child_process.spawnSync()`][], [`child_process.execSync()`][], e [`child_process.execFileSync()`][] sono **sincroni** e **bloccheranno** il ciclo di eventi di Node.js, interrompendo l'esecuzione di qualsiasi codice aggiuntivo fino alla chiusura del processo generato.

Le chiamate di blocco come queste sono per lo più utili per semplificare attività di scripting generiche e per semplificare il caricamento/l'elaborazione della configurazione dell'applicazione all'avvio.

### child_process.execFileSync(file\[, args\]\[, options\])

<!-- YAML
added: v0.11.12
changes:

  - version: v8.8.0
    pr-url: https://github.com/nodejs/node/pull/15380
    description: The `windowsHide` option is supported now.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10653
    description: The `input` option can now be a `Uint8Array`.
  - version: v6.2.1, v4.5.0
    pr-url: https://github.com/nodejs/node/pull/6939
    description: The `encoding` option can now explicitly be set to `buffer`.
-->

* `file` {string} Il nome o il percorso del file eseguibile da avviare.
* `args` {string[]} Elenco degli argomenti di string.
* `options` {Object} 
  * `cwd` {string} Attuale directory di lavoro del processo child.
  * `input` {string|Buffer|Uint8Array} Il valore che verrà passato come stdin al processo generato. Fornendo questo valore `stdio[0]` verrà sottoposto all'override.
  * `stdio` {string|Array} Configurazione stdio del child. Di default `stderr` sarà l'output dello stderr del processo parent a meno che non sia specificato `stdio`. **Default:** `'pipe'`.
  * `env` {Object} Coppie key-value dell'ambiente.
  * `uid` {number} Imposta l'identità dell'utente (user identity) del processo (vedi setuid(2)).
  * `gid` {number} Imposta l'identità di gruppo (group identity) del processo (vedi setgid(2)).
  * `timeout` {number} La quantità massima di tempo in millisecondi in cui il processo può essere eseguito. **Default:** `undefined`.
  * `killSignal` {string|integer} Il valore del segnale da utilizzare quando il processo generato verrà arrestato. **Default:** `'SIGTERM'`.
  * `maxBuffer` {number} La quantità massima di dati in byte consentiti su stdout o stderr. Se superata, il processo child viene concluso. Vedi avvertenze su [`maxBuffer` and Unicode][]. **Default:** `200 * 1024`.
  * `encoding` {string} L'encoding (codifica) utilizzata per tutti gli input e gli output stdio. **Default:** `'buffer'`.
  * `windowsHide` {boolean} Nasconde la finestra della console di sottoprocesso che verrebbe normalmente creata sui sistemi Windows. **Default:** `false`.
  * `shell` {boolean|string} Se `true`, esegue `command` all'interno di una shell. Utilizza `'/bin/sh'` su UNIX, e `process.env.ComSpec` su Windows. Una shell diversa può essere specificata come una stringa. Vedi [Requisiti della Shell](#child_process_shell_requirements) e [Shell Default di Windows](#child_process_default_windows_shell). **Default:** `false` (nessuna shell).
* Restituisce: {Buffer|string} Lo stdout dal comando.

Il metodo `child_process.execFileSync()` è generalmente identico a [`child_process.execFile()`][] con l'eccezione che il metodo non restituirà nulla finché il processo child non sarà completamente chiuso. Quando si verifica un timeout e viene inviato `killSignal`, il metodo non restituirà nulla finché il processo non sarà completamente concluso.

Se il processo child intercetta e gestisce il segnale `SIGTERM` e non si conclude, il processo parent aspetterà ancora fino alla conclusione del processo child.

Se il processo è scaduto (timeout) o ha un valore di uscita diverso da zero, questo metodo ***genererà*** un [`Error`][] che includerà il risultato completo del [`child_process.spawnSync()`][] sottostante.

**Se l'opzione `shell` è abilitata, non passare l'input unsanitized user a questa funzione. Qualsiasi input contenente metacaratteri della shell può essere utilizzato per attivare l'esecuzione arbitraria dei comandi.**

### child_process.execSync(command[, options])

<!-- YAML
added: v0.11.12
changes:

  - version: v8.8.0
    pr-url: https://github.com/nodejs/node/pull/15380
    description: The `windowsHide` option is supported now.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10653
    description: The `input` option can now be a `Uint8Array`.
-->

* `command` {string} Il comando da eseguire.
* `options` {Object} 
  * `cwd` {string} Attuale directory di lavoro del processo child.
  * `input` {string|Buffer|Uint8Array} Il valore che verrà passato come stdin al processo generato. Fornendo questo valore `stdio[0]` verrà sottoposto all'override.
  * `stdio` {string|Array} Configurazione stdio del child. Di default `stderr` sarà l'output dello stderr del processo parent a meno che non sia specificato `stdio`. **Default:** `'pipe'`.
  * `env` {Object} Coppie key-value dell'ambiente.
  * `shell` {string} La Shell con la quale eseguire il comando. Vedi [Requisiti della Shell](#child_process_shell_requirements) e [Shell Default di Windows](#child_process_default_windows_shell). **Default:** `'/bin/sh'` su UNIX, `process.env.ComSpec` su Windows.
  * `uid` {number} Imposta l'identità dell'utente (user identity) del processo. (Vedi setuid(2)).
  * `gid` {number} Imposta l'identità di gruppo (group identity) del processo. (Vedi setgid(2)).
  * `timeout` {number} La quantità massima di tempo in millisecondi in cui il processo può essere eseguito. **Default:** `undefined`.
  * `killSignal` {string|integer} Il valore del segnale da utilizzare quando il processo generato verrà arrestato. **Default:** `'SIGTERM'`.
  * `maxBuffer` {number} La quantità massima di dati in byte consentiti su stdout o stderr. Se superata, il processo child viene concluso. Vedi avvertenze su [`maxBuffer` and Unicode][]. **Default:** `200 * 1024`.
  * `encoding` {string} L'encoding (codifica) utilizzata per tutti gli input e gli output stdio. **Default:** `'buffer'`.
  * `windowsHide` {boolean} Nasconde la finestra della console di sottoprocesso che verrebbe normalmente creata sui sistemi Windows. **Default:** `false`.
* Restituisce: {Buffer|string} Lo stdout dal comando.

Il metodo `child_process.execSync()` è generalmente identico a [`child_process.exec()`][] con l'eccezione che il metodo restituirà nulla finché il processo child non sarà completamente chiuso. Quando si verifica un timeout e viene inviato `killSignal`, il metodo non restituirà nulla finché il processo non sarà completamente concluso. *Da notare che se il processo child intercetta e gestisce il segnale `SIGTERM` e non si conclude, il processo parent aspetterà fino alla conclusione del processo child.*

Se il processo è scaduto (timeout) oppure ha un valore di uscita diverso da zero, questo metodo ***verrà*** generato. L'[`Error`][] object conterrà l'intero risultato proveniente da [`child_process.spawnSync()`][].

**Non passare mai l'input unsanitized user a questa funzione. Qualsiasi input contenente metacaratteri della shell può essere utilizzato per attivare l'esecuzione arbitraria dei comandi.**

### child_process.spawnSync(command\[, args\]\[, options\])

<!-- YAML
added: v0.11.12
changes:

  - version: v8.8.0
    pr-url: https://github.com/nodejs/node/pull/15380
    description: The `windowsHide` option is supported now.
  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10653
    description: The `input` option can now be a `Uint8Array`.
  - version: v6.2.1, v4.5.0
    pr-url: https://github.com/nodejs/node/pull/6939
    description: The `encoding` option can now explicitly be set to `buffer`.
  - version: v5.7.0
    pr-url: https://github.com/nodejs/node/pull/4598
    description: The `shell` option is supported now.
-->

* `command` {string} Il comando da eseguire.
* `args` {string[]} Elenco degli argomenti di string.
* `options` {Object} 
  * `cwd` {string} Attuale directory di lavoro del processo child.
  * `input` {string|Buffer|Uint8Array} Il valore che verrà passato come stdin al processo generato. Fornendo questo valore `stdio[0]` verrà sottoposto all'override.
  * `stdio` {string|Array} Configurazione stdio del child.
  * `env` {Object} Coppie key-value dell'ambiente.
  * `uid` {number} Imposta l'identità dell'utente (user identity) del processo (vedi setuid(2)).
  * `gid` {number} Imposta l'identità di gruppo (group identity) del processo (vedi setgid(2)).
  * `timeout` {number} La quantità massima di tempo in millisecondi in cui il processo può essere eseguito. **Default:** `undefined`.
  * `killSignal` {string|integer} Il valore del segnale da utilizzare quando il processo generato verrà arrestato. **Default:** `'SIGTERM'`.
  * `maxBuffer` {number} La quantità massima di dati in byte consentiti su stdout o stderr. Se superata, il processo child viene concluso. Vedi avvertenze su [`maxBuffer` and Unicode][]. **Default:** `200 * 1024`.
  * `encoding` {string} L'encoding (codifica) utilizzata per tutti gli input e gli output stdio. **Default:** `'buffer'`.
  * `shell` {boolean|string} Se `true`, esegue `command` all'interno di una shell. Utilizza `'/bin/sh'` su UNIX, e `process.env.ComSpec` su Windows. Una shell diversa può essere specificata come una stringa. Vedi [Requisiti della Shell](#child_process_shell_requirements) e [Shell Default di Windows](#child_process_default_windows_shell). **Default:** `false` (nessuna shell).
  * `windowsVerbatimArguments` {boolean} Non viene eseguita nessuna citazione od escaping degli argomenti su Windows. Ignorato su Unix. È impostato su `true` automaticamente quando la `shell` è specificata. **Default:** `false`.
  * `windowsHide` {boolean} Nasconde la finestra della console di sottoprocesso che verrebbe normalmente creata sui sistemi Windows. **Default:** `false`.
* Restituisce: {Object} 
  * `pid` {number} Pid (Process Identifier) del processo child.
  * `output` {Array} Array dei risultati provenienti dall'output di stdio.
  * `stdout` {Buffer|string} Il contenuto di `output[1]`.
  * `stderr` {Buffer|string} Il contenuto di `output[2]`.
  * `status` {number} Il valore di uscita del processo child.
  * `signal` {string} Il segnale utilizzato per arrestare il processo child.
  * `error` {Error} L'error object se il processo child ha avuto esito negativo oppure è scaduto (timeout).

Il metodo `child_process.spawnSync()` è generalmente identico a [`child_process.spawn()`][] con l'eccezione che la funzione non restituirà nulla finché il processo child non sarà completamente chiuso. Quando si verifica un timeout e viene inviato `killSignal`, il metodo non restituirà nulla finché il processo non sarà completamente concluso. Da notare che se il processo child intercetta e gestisce il segnale `SIGTERM` e non si conclude, il processo parent aspetterà fino alla conclusione del processo child.

**Se l'opzione `shell` è abilitata, non passare l'input unsanitized user a questa funzione. Qualsiasi input contenente metacaratteri della shell può essere utilizzato per attivare l'esecuzione arbitraria dei comandi.**

## Class: ChildProcess

<!-- YAML
added: v2.2.0
-->

Le istanze della classe `ChildProcess`, che rappresentano i processi child generati, sono [`EventEmitters`][`EventEmitter`].

Le istanze di `ChildProcess` non devono essere create direttamente. Piuttosto, per creare istanze di `ChildProcess`, utilizza i metodi [`child_process.spawn()`][], [`child_process.exec()`][], [`child_process.execFile()`][], oppure [`child_process.fork()`][].

### Event: 'close'

<!-- YAML
added: v0.7.7
-->

* `code` {number} Il valore di uscita se il child si è concluso autonomamente.
* `signal` {string} Il segnale con cui è stato terminato il processo child.

L'evento `'close'` viene emesso quando gli stream stdio di un processo child sono stati chiusi. E' diverso dall'evento [`'exit'`][], in quanto più processi potrebbero condividere gli stessi stream stdio.

### Event: 'disconnect'

<!-- YAML
added: v0.7.2
-->

L'evento `'disconnect'` viene emesso dopo aver chiamato il metodo [`subprocess.disconnect()`][] nel processo parent oppure [`process.disconnect()`][] nel processo child. Dopo averlo disconnesso, non è più possibile inviare o ricevere messaggi e la proprietà [`subprocess.connected`][] diventa `false`.

### Event: 'error'

* `err` {Error} L'errore.

L'evento `'error'` viene emesso ogni volta che:

1. Non è stato possibile generare il processo, oppure
2. Non è stato possibile arrestare il processo, oppure
3. Non è andato a buon fine l'invio di un messaggio al processo child.

L'evento `'exit'` può o non può funzionare dopo che si è verificato un errore. Quando si ascoltano entrambi gli eventi `'exit'` ed `'error'`, è importante stare in guardia contro molteplici invocazioni accidentali delle funzioni dell'handler.

Vedi anche [`subprocess.kill()`][] e [`subprocess.send()`][].

### Event: 'exit'

<!-- YAML
added: v0.1.90
-->

* `code` {number} Il valore di uscita se il child si è concluso autonomamente.
* `signal` {string} Il segnale con cui è stato terminato il processo child.

L'evento `'exit'` viene emesso al termine del processo child. Se il processo è concluso, `code` è il valore di uscita finale del processo, in caso contrario `null`. Se il processo termina a causa della ricezione di un segnale, `signal` è il nome della stringa del segnale, in caso contrario `null`. Uno dei due sarà sempre non nullo.

Da notare che quando viene attivato l'evento `'exit'`, gli stream stdio del processo child potrebbero essere ancora aperti.

Da notare inoltre che Node.js stabilisce gli handler dei segnali per `SIGINT` e `SIGTERM` e i processi Node.js non terminano immediatamente a causa della ricezione di quei segnali. Anzi, Node.js eseguirà una sequenza di azioni di pulizia e di conseguenza rileverà nuovamente il segnale gestito dagli handler.

Vedi waitpid(2).

### Event: 'message'

<!-- YAML
added: v0.5.9
-->

* `message` {Object} Un JSON object analizzato tramite il parsing oppure un valore primitivo.
* `sendHandle` {Handle} Un [`net.Socket`][] object o un [`net.Server`][] object, oppure un valore undefined (indefinito).

L'evento `'message'` viene attivato quando un processo child utilizza [`process.send()`][] per inviare messaggi.

Il messaggio passa attraverso la serializzazione e il parsing. Il messaggio risultante potrebbe non essere uguale a quello che è stato inviato originariamente.

### subprocess.channel

<!-- YAML
added: v7.1.0
-->

* {Object} Un pipe che rappresenta il canale IPC per il processo child.

La proprietà `subprocess.channel` è un riferimento al canale IPC del child. Se al momento non esiste alcun canale IPC, questa proprietà è `undefined`.

### subprocess.connected

<!-- YAML
added: v0.7.2
-->

* {boolean} Impostata su `false` dopo che viene chiamato `subprocess.disconnect()`.

La proprietà `subprocess.connected` indica se è ancora possibile inviare e ricevere messaggi da un processo child. Quando `subprocess.connected` è `false`, non è più possibile inviare o ricevere messaggi.

### subprocess.disconnect()

<!-- YAML
added: v0.7.2
-->

Chiude il canale IPC tra parent e child, consentendo al child di concludersi facilmente una volta che non ci sono altre connessioni che lo mantengano in funzione. Dopo aver chiamato questo metodo, le proprietà `subprocess.connected` e `process.connected`, rispettivamente all'interno del parent e del child, saranno impostate su `false` e non sarà più possibile far passare messaggi tra i due processi.

L'evento `'disconnect'` verrà emesso quando non ci saranno messaggi in fase di ricezione. Questo verrà spesso attivato immediatamente dopo aver chiamato `subprocess.disconnect()`.

Da notare che quando il processo child è un'istanza di Node.js (ad esempio generata con [`child_process.fork()`]), il metodo `process.disconnect()` può essere invocato all'interno del processo child per chiudere anche il canale IPC.

### subprocess.kill([signal])

<!-- YAML
added: v0.1.90
-->

* `signal` {string}

Il metodo `subprocess.kill()` invia un segnale al processo child. Se non viene fornito alcun argomento, al processo verrà inviato il segnale `'SIGTERM'`. Vedi signal(7) per la lista dei segnali disponibili.

```js
const { spawn } = require('child_process');
const grep = spawn('grep', ['ssh']);

grep.on('close', (code, signal) => {
  console.log(
    `child process terminated due to receipt of signal ${signal}`);
});

// Invia SIGHUP al processo
grep.kill('SIGHUP');
```

Il [`ChildProcess`][] object potrebbe emettere un evento [`'error'`][] se il segnale non può essere consegnato. L'invio di un segnale ad un processo child che si è già concluso non è un errore ma potrebbe comportare conseguenze impreviste. In particolare, se il process identifier (PID) è stato riassegnato a un altro processo, allora il segnale verrà consegnato a quel processo che può avere risultati imprevisti.

Da notare che nonostante la funzione venga chiamata `kill`, il segnale inviato al processo child effettivamente potrebbe non terminare il processo.

Vedi kill(2) come riferimento.

Da notare anche che: su Linux, i processi child dei processi child non verranno terminati quando si tenta di arrestare i loro parent. È probabile che ciò accada quando si esegue un nuovo processo in una shell oppure con l'utilizzo dell'opzione `shell` di `ChildProcess`, come in questo esempio:

```js
'use strict';
const { spawn } = require('child_process');

const subprocess = spawn(
  'sh',
  [
    '-c',
    `node -e "setInterval(() => {
      console.log(process.pid, 'is alive')
    }, 500);"`
  ], {
    stdio: ['inherit', 'inherit', 'inherit']
  }
);

setTimeout(() => {
  subprocess.kill(); // non termina il processo del nodo nella shell
}, 2000);
```

### subprocess.killed

<!-- YAML
added: v0.5.10
-->

* {boolean} Impostata su `true` dopo aver utilizzato `subprocess.kill()` per inviare correttamente un segnale al processo child.

La proprietà `subprocess.killed` indica se il processo child ha ricevuto correttamente un segnale da `subprocess.kill()`. La proprietà `killed` non sta ad indicare che il processo child è stato arrestato.

### subprocess.pid

<!-- YAML
added: v0.1.90
-->

* {integer}

Restituisce il process identifier (PID) del processo child.

Esempio:

```js
const { spawn } = require('child_process');
const grep = spawn('grep', ['ssh']);

console.log(`Spawned child pid: ${grep.pid}`);
grep.stdin.end();
```

### subprocess.send(message\[, sendHandle[, options]\]\[, callback\])

<!-- YAML
added: v0.5.9
changes:

  - version: v5.8.0
    pr-url: https://github.com/nodejs/node/pull/5283
    description: The `options` parameter, and the `keepOpen` option
                 in particular, is supported now.
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/3516
    description: This method returns a boolean for flow control now.
  - version: v4.0.0
    pr-url: https://github.com/nodejs/node/pull/2620
    description: The `callback` parameter is supported now.
-->

* `message` {Object}
* `sendHandle` {Handle}
* `options` {Object} L'argomento `options`, se presente, è un object utilizzato per parametrizzare l'invio di determinati tipi di handle. `options` supporta le seguenti proprietà: 
  * `keepOpen` {boolean} Un valore che può essere utilizzato durante il passaggio delle istanze di `net.Socket`. Quando è `true`, il socket viene mantenuto aperto nel processo di invio. **Default:** `false`.
* `callback` {Function}
* Restituisce: {boolean}

Quando viene stabilito un canale IPC tra parent e child (cioè quando si utilizza [`child_process.fork()`][]), il metodo `subprocess.send()` può essere utilizzato per inviare messaggi al processo child. Quando il processo child è un'istanza di Node.js, questi messaggi possono essere ricevuti tramite l'evento [`'message'`][].

Il messaggio passa attraverso la serializzazione e il parsing. Il messaggio risultante potrebbe non essere uguale a quello che è stato inviato originariamente.

Per esempio, nello script del parent:

```js
const cp = require('child_process');
const n = cp.fork(`${__dirname}/sub.js`);

n.on('message', (m) => {
  console.log('PARENT got message:', m);
});

// Fa sì che il child stampi: CHILD got message: { hello: 'world' }
n.send({ hello: 'world' });
```

E poi lo script del child, che potrebbe assomigliare a `'sub.js'`:

```js
process.on('message', (m) => {
  console.log('CHILD got message:', m);
});

// Fa sì che il parent stampi: PARENT got message: { foo: 'bar', baz: null }
process.send({ foo: 'bar', baz: NaN });
```

I processi child di Node.js avranno un proprio metodo [`process.send()`][] che gli permette di inviare messaggi al parent.

Si presenta un caso particolare quando si invia un messaggio `{cmd: 'NODE_foo'}`. I messaggi contenenti un prefisso `NODE_` nella proprietà `cmd` sono riservati per l'uso nel Node.js core e non verranno emessi nell'evento [`'message'`][] del child. Anzi, tali messaggi vengono emessi utilizzando l'evento `'internalMessage'` e sono utilizzati internamente da Node.js. Le applicazioni dovrebbero evitare l'utilizzo di messaggi del genere o di ascoltare gli eventi `'internalMessage'` in quanto sono soggetti a modifiche senza alcun preavviso.

L'argomento facoltativo `sendHandle` che può essere passato a `subprocess.send()` serve a far passare un server TCP o un socket object al processo child. Il child riceverà l'object come secondo argomento passato alla funzione callback registrata sull'evento [`'message'`][]. Tutti i dati ricevuti e memorizzati tramite il buffering all'interno del socket non verranno inviati al child.

Il `callback` facoltativo è una funzione invocata dopo aver inviato il messaggio ma prima che il child l'abbia ricevuto. La funzione viene chiamata con un singolo argomento: `null` in caso di successo, oppure un [`Error`][] object se fallisce.

Se non viene fornita alcuna funzione `callback` e il messaggio non può essere inviato, verrà emesso un evento `'error'` dal [`ChildProcess`][] object. Ciò può accadere, ad esempio, quando il processo child si è già concluso.

`subprocess.send()` restituirà `false` se il canale è chiuso o quando il backlog dei messaggi non inviati supera una soglia oltre la quale è sconsigliato inviarne altri. Altrimenti, il metodo restituisce `true`. La funzione `callback` può essere utilizzata per implementare il flow control (controllo del flusso).

#### Esempio: invio di un server object

L'argomento `sendHandle` può essere utilizzato, ad esempio, per passare l'handle di un server object TCP al processo child come mostrato di seguito:

```js
const subprocess = require('child_process').fork('subprocess.js');

// Apri il server object ed invia l'handle.
const server = require('net').createServer();
server.on('connection', (socket) => {
  socket.end('handled by parent');
});
server.listen(1337, () => {
  subprocess.send('server', server);
});
```

Quindi il child riceverà il server object come:

```js
process.on('message', (m, server) => {
  if (m === 'server') {
    server.on('connection', (socket) => {
      socket.end('handled by child');
    });
  }
});
```

Una volta che il server viene condiviso tra parent e child, alcune connessioni possono essere gestite dal parent ed altre dal child.

Mentre l'esempio precedente utilizza un server creato tramite il modulo `net`, i server del modulo `dgram` usano esattamente lo stesso workflow fatta eccezione per l'ascolto dell'evento `'message'` al posto di `'connection'` e per l'utilizzo di `server.bind()` al posto di `server.listen()`. Tuttavia attualmente è supportato solo su piattaforme UNIX.

#### Esempio: invio di un socket object

Allo stesso modo, l'argomento `sendHandler` può essere utilizzato per passare l'handle di un socket al processo child. L'esempio seguente genera due children che gestiscono rispettivamente le connessioni con priorità "normal" o "special":

```js
const { fork } = require('child_process');
const normal = fork('subprocess.js', ['normal']);
const special = fork('subprocess.js', ['special']);

// Apri il server e invia i socket al child. Utilizza pauseOnConnect per impedire 
// la lettura dei socket prima che vengano inviati al processo child.
const server = require('net').createServer({ pauseOnConnect: true });
server.on('connection', (socket) => {

  // Se questa è una priorità special
  if (socket.remoteAddress === '74.125.127.100') {
    special.send('socket', socket);
    return;
  }
  // Questa è una priorità normal
  normal.send('socket', socket);
});
server.listen(1337);
```

Il `subprocess.js` riceverà l'handle del socket come secondo argomento passato alla funzione callback dell'evento:

```js
process.on('message', (m, socket) => {
  if (m === 'socket') {
    if (socket) {
      // Verifica che il client socket esista.
      // È possibile che il socket venga chiuso dal momento in cui viene
// inviato fino a quando non viene ricevuto nel processo child.
      socket.end(`Request handled with ${process.argv[2]} priority`);
    }
  }
});
```

Una volta che un socket viene passato a un child, il parent non è più in grado di rilevare quando tale socket viene distrutto. Per indicare ciò, la proprietà `.connections` diventa `null`. Si consiglia di non utilizzare `.maxConnections` quando questo si verifica.

Si consiglia inoltre che qualsiasi `'message'` handler nel processo child verifichi che il `socket` esista, in quanto la connessione potrebbe essere stata chiusa mentre veniva inviata al child.

### subprocess.stderr

<!-- YAML
added: v0.1.90
-->

* {stream.Readable}

Un `Readable Stream` che rappresenta lo `stderr` del processo child.

Se il child è stato generato con `stdio[2]` impostato su un valore diverso da `'pipe'`, allora questo sarà `null`.

`subprocess.stderr` è un alias per `subprocess.stdio[2]`. Entrambe le proprietà faranno riferimento allo stesso valore.

### subprocess.stdin

<!-- YAML
added: v0.1.90
-->

* {stream.Writable}

Un `Writable Stream` che rappresenta lo `stdin` del processo child.

*Da notare che se un processo child attende la lettura completa del proprio input, il processo non continuerà fino a quando questo stream non sarà stato chiuso tramite `end()`.*

Se il child è stato generato con `stdio[0]` impostato su un valore diverso da `'pipe'`, allora questo sarà `null`.

`subprocess.stdin` è un alias per `subprocess.stdio[0]`. Entrambe le proprietà faranno riferimento allo stesso valore.

### subprocess.stdio

<!-- YAML
added: v0.7.10
-->

* {Array}

Uno sparse array dei pipe per il processo child, corrispondenti alle posizioni nell'opzione [`stdio`][], passate a [`child_process.spawn()`][], che sono state impostate al valore `'pipe'`. Da notare che `subprocess.stdio[0]`, `subprocess.stdio[1]`, e `subprocess.stdio[2]` sono disponibili rispettivamente anche come `subprocess.stdin`, `subprocess.stdout`, e `subprocess.stderr`.

Nell'esempio seguente, solo il file descriptor `1` (stdout) del child è configurato come un pipe, quindi solo il `subprocess.stdio[1]` del parent è uno stream, tutti gli altri valori nell'array sono `null`.

```js
const assert = require('assert');
const fs = require('fs');
const child_process = require('child_process');

const subprocess = child_process.spawn('ls', {
  stdio: [
    0, // Usa lo stdin del parent per il child
    'pipe', // Esegue il piping dello stdout del child al parent
    fs.openSync('err.out', 'w') // Dirige lo stderr del child su un file
  ]
});

assert.strictEqual(subprocess.stdio[0], null);
assert.strictEqual(subprocess.stdio[0], subprocess.stdin);

assert(subprocess.stdout);
assert.strictEqual(subprocess.stdio[1], subprocess.stdout);

assert.strictEqual(subprocess.stdio[2], null);
assert.strictEqual(subprocess.stdio[2], subprocess.stderr);
```

### subprocess.stdout

<!-- YAML
added: v0.1.90
-->

* {stream.Readable}

Un `Readable Stream` che rappresenta lo `stdout` del processo child.

Se il child è stato generato con `stdio[1]` impostato su un valore diverso da `'pipe'`, allora questo sarà `null`.

`subprocess.stdout` è un alias per `subprocess.stdio[1]`. Entrambe le proprietà faranno riferimento allo stesso valore.

## `maxBuffer` e Unicode

L'opzione `maxBuffer` specifica il massimo numero di byte consentiti su `stdout` o su `stderr`. Se questo valore viene superato, allora il processo child viene concluso. Questo influisce sull'output che include gli encoding di caratteri multibyte come UTF-8 o UTF-16. Ad esempio, `console.log('中文测试')` invierà 13 byte con codifica UTF-8 a `stdout` sebbene ci siano solo 4 caratteri.

## Requisiti della Shell

La shell dovrebbe capire lo switch `-c` su UNIX oppure `/d /s /c` su Windows. Su Windows, il parsing (analisi) della command line dovrebbe essere compatibile con `'cmd.exe'`.

## Shell Default di Windows

Sebbene Microsoft specifichi che `%COMSPEC%` debba contenere il percorso per `'cmd.exe'` nell'ambiente del root, i processi child non sono sempre soggetti alla stessa condizione. Pertanto, nelle funzioni `child_process` in cui è possibile generare una shell, `'cmd.exe'` viene utilizzato come fallback se `process.env.ComSpec` non è disponibile.