# Pętla zdarzeń programu Node.js, Zegary i `process.nextTick()`

## Co to jest Pętla Zdarzeń?

Pętla zdarzeń umożliwia Node.js wykonywanie nieblokujących operacji wej/wyj operacje - pomimo tego, że JavaScript jest jednowątkowy - przez przeładowywanie operacji na jądro systemu, gdy tylko jest to możliwe.

Ponieważ większość nowoczesnych jąder jest wielowątkowych, mogą obsługiwać wiele operacji wykonywanych w tle. Kiedy jedna z tych operacji zakończy się, jądro mówi Node.js, aby odpowiednie wywołanie zwrotne mogły zostać dodane do kolejki **sondażu**, aby ostatecznie został wykonany. Wyjaśnimy to bardziej szczegółowo w dalszej części tego tematu.

## Objaśnienie Pętli Zdarzeń

Po uruchomieniu Node.js inicjuje pętlę zdarzeń, przetwarza dostarczony skrypt wejściowy (lub wpada w [REPL](https://nodejs.org/api/repl.html#repl_repl), który nie jest uwzględniony w ten dokument), który może wykonywać asynchroniczne wywołania API, planować zegary lub wywoływać `process.nextTick()`, a następnie rozpoczyna przetwarzanie pętli zdarzeń.

Poniższy diagram przedstawia uproszczony przegląd kolejności operacji pętli zdarzeń.

```txt
   ┌───────────────────────┐
┌─>│ timery │
│ └──────────┬────────────┘
│ ┌──────────┴────────────┐
│ │ I/O wywołania zwrotne │
│ └──────────┬────────────┘
│ ┌──────────┴────────────┐
│ │ bezczynność, przygotowanie │
│ └──────────┬────────────┘ ┌───────────────┐
│ ┌──────────┴────────────┐ │ przychodzące: │
│ │ sonda │<─────┤  połączenia, │ │
└──────────┬────────────┘ │ dane, itp.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        weryfikacja          │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    zamknij wywołania zwrotne    │
   └───────────────────────┘
```

*uwaga: każde pole będzie określane jako "faza" pętli zdarzeń.*

Każda faza ma kolejkę wywołania zwrotnego FIFO do wykonania. Podczas gdy każda faza jest wyjątkowa na swój sposób, ogólnie, gdy pętla zdarzeń wchodzi w daną fazę W takim wypadku wykona ona wszystkie operacje właściwe dla tej fazy, następnie wykona wywołania zwrotne w kolejce tej fazy, aż do momentu gdy kolejka wyczerpie się lub zostanie wykonana maksymalna liczba wywołań zwrotnych. Kiedy kolejka została wyczerpana lub osiągnięto limit wywołań zwrotnych, pętla zdarzeń przejdzie do następnej fazy i tak dalej.

Ponieważ każda z tych operacji może zaplanować *więcej* operacji i nowych zdarzeń przetwarzanych w **ankieta** są ustawiane w kolejce przez jądro, ankieta zdarzeń może stać w kolejce, podczas gdy zdarzenia ankietowania są przetwarzane. Jak W rezultacie długotrwałe wywoływania zwrotne mogą znacznie przyspieszyć fazę odpytywania dłużej niż próg timera. Zobacz [**timery**](#timers) i **odpytywanie</​​1>, aby uzyskać więcej informacji.</p> 

***UWAGA:** Występuje niewielka rozbieżność między Windowsem i Implementacją systemu Unix/Linux, ale to nie ma znaczenia dla tej demonstracji. Najważniejsze części są tutaj. Istnieje faktycznie siedem albo osiem kroków, ale te którymi się przejmujemy - te które obecnie wykorzystuje - są powyżej.*

## Przegląd Faz

* **timery**: faza ta wykonuje wywołania zwrotne zaplanowane przez `ustawKoniecCzasu()`i `ustawiinterwał()`.
* **Wej/Wyj wywołania zwrotne**: wykonuje prawie wszystkie wywołania zwrotne z wyjątkiem zamkniętych wywołań zwrotnych, te zaplanowane przez timery i `ustawnatychmiastowo()`.
* **bezczynność, przygotuj**: używane tylko wewnętrznie.
* **odpytywanie**: odzyskaj nowe zdarzenia Wej/Wyj; węzeł zostanie tutaj zablokowany, gdy będzie to właściwe.
* **sprawdź**: `ustawNatychmiastowo()` wywołania zwrotne są wywoływane tutaj.
* **zamknięte wywołania zwrotne**: np `socket.on('zamknij',...)`.

Między każdym uruchomieniem pętli zdarzeń Node.js sprawdza, czy oczekuje dowolne asynchroniczne operacje wej/wyj lub timerów i wyłącza się, jeśli nie.

## Fazy w Szczegółach

### timery

Timer określa **próg***, po którym * jest zapewnione wywołanie zwrotne *może być wykonywane*zamiast **dokładnego**czasu, gdy osoba* chce, aby było to wykonane*. Połączenia zwrotne timerów działają tak wcześnie, jak tylko mogą zaplanowane po upływie określonego czasu; jednak, planowanie Systemu Operacyjnego lub uruchamianie innych wywołań zwrotnych może się opóźnić im.

***Uwaga**: Z technicznego punktu widzenia [**odpytywanie**faza](#poll)kontroluje timery, które są wykonywane.*

Na przykład powiedzmy, że planujesz czas oczekiwania na wykonanie po próg 100 ms, wtedy twój skrypt zaczyna asynchronicznie odczytywać plik, który trwa 95 ms:

```js
const fs = require('fs');

function someAsyncOperation(callback) {
  // Assume this takes 95ms to complete
  fs.readFile('/path/to/file', callback);
}

const timeoutScheduled = Date.now();

setTimeout(function() {

  const delay = Date.now() - timeoutScheduled;

  console.log(delay + 'ms have passed since I was scheduled');
}, 100);


// do someAsyncOperation which takes 95 ms to complete
someAsyncOperation(function() {

  const startCallback = Date.now();

  // do something that will take 10ms...
  while (Date.now() - startCallback < 10) {
    // do nothing
  }

});
```

Kiedy pętla zdarzeń wchodzi w fazę **odpytywania**, ma pustą kolejkę (`fs.readFile()` nie zostało zakończone), więc będzie czekać na liczbę ms pozostałych do ​​osiągnięcia progu jak najszybszego timera. Podczas gdy jest oczekiwanie 95 ms przejścia, `fs.readFile()` kończy czytanie pliku i jego wywołanie zwrotne, które trwa 10 ms, jest dodawane do kolejki **odpytywania** i wykonany. Po zakończeniu wywołania zwrotnego nie ma więcej wywołań zwrotnych w kolejce, więc pętla zdarzeń zobaczy, że próg najwcześniejszego timera został osiągnięty, a następnie zawinięty do fazy ** timerów** w celu wykonania wywołania zwrotnego timera. W tym przykładzie zobaczysz całkowite opóźnienie pomiędzy zaplanowanym timerem a jego wywoływaniem zwrotnym wykonywanym przez 105ms.

Uwaga: Aby nie dopuścić do fazy **odpytywania** z powodu zagłodzenia pętli zdarzeń, \[libuv\] (http://libuv.org/) (biblioteka C, która implementuje Node.js pętlę zdarzeń i wszystkie asynchroniczne zachowania platformy) ma również twarde maksimum (zależne od systemu), zanim przestanie odpytywać dla większej ilości wydarzeń.

### Wej/Wyj wywołania zwrotne

Ta faza wykonuje wywołania zwrotne dla niektórych operacji systemowych, takich jak typy błędów TCP. Na przykład, jeśli gniazdo TCP otrzymuje `POŁĄCZENIE ODRZUCONE` kiedy próbując się połączyć, niektóre systemy \* nix chcą czekają na zgłoszenie błędu. Zostanie on umieszczony w kolejce do wykonania w fazie **wywołania zwrotne**.

### odpytywanie

Faza **odpytywania** ma dwie główne funkcje:

1. Wykonywanie skryptów dla timerów, których próg upłynął, a następnie
2. Przetwarzanie zdarzeń w kolejce **odpytywania **.

Kiedy pętla zdarzeń wchodzi w fazę* **odpytywania** i nie ma zaplanowanych timerów *, nastąpi jedna z dwóch rzeczy:

* *jeśli **odpytywania**kolejka**nie jest pusta***, pętla zdarzeń zostanie powtórzona poprzez kolejkę wywołań zwrotnych, synchronicznie do czasu albo kolejka została wyczerpana, albo zależny od systemu surowy limit został osiągnięty.

* *jeśli **odpytywania** kolejka **jest pusta***, staną się jedna lub dwie rzeczy:
    
    * If scripts have been scheduled by `setImmediate()`, the event loop will end the **poll** phase and continue to the **check** phase to execute those scheduled scripts.
    
    * If scripts **have not** been scheduled by `setImmediate()`, the event loop will wait for callbacks to be added to the queue, then execute them immediately.

Once the **poll** queue is empty the event loop will check for timers *whose time thresholds have been reached*. If one or more timers are ready, the event loop will wrap back to the **timers** phase to execute those timers' callbacks.

### check

This phase allows a person to execute callbacks immediately after the **poll** phase has completed. If the **poll** phase becomes idle and scripts have been queued with `setImmediate()`, the event loop may continue to the **check** phase rather than waiting.

`setImmediate()` is actually a special timer that runs in a separate phase of the event loop. It uses a libuv API that schedules callbacks to execute after the **poll** phase has completed.

Generally, as the code is executed, the event loop will eventually hit the **poll** phase where it will wait for an incoming connection, request, etc. However, if a callback has been scheduled with `setImmediate()` and the **poll** phase becomes idle, it will end and continue to the **check** phase rather than waiting for **poll** events.

### close callbacks

If a socket or handle is closed abruptly (e.g. `socket.destroy()`), the `'close'` event will be emitted in this phase. Otherwise it will be emitted via `process.nextTick()`.

## `setImmediate()` vs `setTimeout()`

`setImmediate` and `setTimeout()` are similar, but behave in different ways depending on when they are called.

* `setImmediate()` is designed to execute a script once the current **poll** phase completes.
* `setTimeout()` schedules a script to be run after a minimum threshold in ms has elapsed.

The order in which the timers are executed will vary depending on the context in which they are called. If both are called from within the main module, then timing will be bound by the performance of the process (which can be impacted by other applications running on the machine).

For example, if we run the following script which is not within an I/O cycle (i.e. the main module), the order in which the two timers are executed is non-deterministic, as it is bound by the performance of the process:

```js
// timeout_vs_immediate.js
setTimeout(function timeout() {
  console.log('timeout');
}, 0);

setImmediate(function immediate() {
  console.log('immediate');
});
```

```console
$ node timeout_vs_immediate.js
timeout
immediate

$ node timeout_vs_immediate.js
immediate
timeout
```

However, if you move the two calls within an I/O cycle, the immediate callback is always executed first:

```js
// timeout_vs_immediate.js
const fs = require('fs');

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});
```

```console
$ node timeout_vs_immediate.js
immediate
timeout

$ node timeout_vs_immediate.js
immediate
timeout
```

The main advantage to using `setImmediate()` over `setTimeout()` is `setImmediate()` will always be executed before any timers if scheduled within an I/O cycle, independently of how many timers are present.

## `process.nextTick()`

### Understanding `process.nextTick()`

You may have noticed that `process.nextTick()` was not displayed in the diagram, even though it's a part of the asynchronous API. This is because `process.nextTick()` is not technically part of the event loop. Instead, the `nextTickQueue` will be processed after the current operation completes, regardless of the current phase of the event loop.

Looking back at our diagram, any time you call `process.nextTick()` in a given phase, all callbacks passed to `process.nextTick()` will be resolved before the event loop continues. This can create some bad situations because **it allows you to "starve" your I/O by making recursive `process.nextTick()` calls**, which prevents the event loop from reaching the **poll** phase.

### Why would that be allowed?

Why would something like this be included in Node.js? Part of it is a design philosophy where an API should always be asynchronous even where it doesn't have to be. Take this code snippet for example:

```js
function apiCall(arg, callback) {
  if (typeof arg !== 'string')
    return process.nextTick(callback,
                            new TypeError('argument should be string'));
}
```

The snippet does an argument check and if it's not correct, it will pass the error to the callback. The API updated fairly recently to allow passing arguments to `process.nextTick()` allowing it to take any arguments passed after the callback to be propagated as the arguments to the callback so you don't have to nest functions.

What we're doing is passing an error back to the user but only *after* we have allowed the rest of the user's code to execute. By using `process.nextTick()` we guarantee that `apiCall()` always runs its callback *after* the rest of the user's code and *before* the event loop is allowed to proceed. To achieve this, the JS call stack is allowed to unwind then immediately execute the provided callback which allows a person to make recursive calls to `process.nextTick()` without reaching a `RangeError: Maximum call stack size exceeded from v8`.

This philosophy can lead to some potentially problematic situations. Take this snippet for example:

```js
let bar;

// this has an asynchronous signature, but calls callback synchronously
function someAsyncApiCall(callback) { callback(); }

// the callback is called before `someAsyncApiCall` completes.
someAsyncApiCall(() => {

  // since someAsyncApiCall has completed, bar hasn't been assigned any value
  console.log('bar', bar); // undefined

});

bar = 1;
```

The user defines `someAsyncApiCall()` to have an asynchronous signature, but it actually operates synchronously. When it is called, the callback provided to `someAsyncApiCall()` is called in the same phase of the event loop because `someAsyncApiCall()` doesn't actually do anything asynchronously. As a result, the callback tries to reference `bar` even though it may not have that variable in scope yet, because the script has not been able to run to completion.

By placing the callback in a `process.nextTick()`, the script still has the ability to run to completion, allowing all the variables, functions, etc., to be initialized prior to the callback being called. It also has the advantage of not allowing the event loop to continue. It may be useful for the user to be alerted to an error before the event loop is allowed to continue. Here is the previous example using `process.nextTick()`:

```js
let bar;

function someAsyncApiCall(callback) {
  process.nextTick(callback);
}

someAsyncApiCall(() => {
  console.log('bar', bar); // 1
});

bar = 1;
```

Here's another real world example:

```js
const server = net.createServer(() => {}).listen(8080);

server.on('listening', () => {});
```

When only a port is passed the port is bound immediately. So the `'listening'` callback could be called immediately. Problem is that the `.on('listening')` will not have been set by that time.

To get around this the `'listening'` event is queued in a `nextTick()` to allow the script to run to completion. Which allows the user to set any event handlers they want.

## `process.nextTick()` vs `setImmediate()`

We have two calls that are similar as far as users are concerned, but their names are confusing.

* `process.nextTick()` fires immediately on the same phase
* `setImmediate()` fires on the following iteration or 'tick' of the event loop

In essence, the names should be swapped. `process.nextTick()` fires more immediately than `setImmediate()` but this is an artifact of the past which is unlikely to change. Making this switch would break a large percentage of the packages on npm. Every day more new modules are being added, which mean every day we wait, more potential breakages occur. While they are confusing, the names themselves won't change.

*We recommend developers use `setImmediate()` in all cases because it's easier to reason about (and it leads to code that's compatible with a wider variety of environments, like browser JS.)*

## Why use `process.nextTick()`?

There are two main reasons:

1. Allow users to handle errors, cleanup any then unneeded resources, or perhaps try the request again before the event loop continues.

2. At times it's necessary to allow a callback to run after the call stack has unwound but before the event loop continues.

One example is to match the user's expectations. Simple example:

```js
const server = net.createServer();
server.on('connection', function(conn) { });

server.listen(8080);
server.on('listening', function() { });
```

Say that `listen()` is run at the beginning of the event loop, but the listening callback is placed in a `setImmediate()`. Now, unless a hostname is passed binding to the port will happen immediately. Now for the event loop to proceed it must hit the **poll** phase, which means there is a non-zero chance that a connection could have been received allowing the connection event to be fired before the listening event.

Another example is running a function constructor that was to, say, inherit from `EventEmitter` and it wanted to call an event within the constructor:

```js
const EventEmitter = require('events');
const util = require('util');

function MyEmitter() {
  EventEmitter.call(this);
  this.emit('event');
}
util.inherits(MyEmitter, EventEmitter);

const myEmitter = new MyEmitter();
myEmitter.on('event', function() {
  console.log('an event occurred!');
});
```

You can't emit an event from the constructor immediately because the script will not have processed to the point where the user assigns a callback to that event. So, within the constructor itself, you can use `process.nextTick()` to set a callback to emit the event after the constructor has finished, which provides the expected results:

```js
const EventEmitter = require('events');
const util = require('util');

function MyEmitter() {
  EventEmitter.call(this);

  // use nextTick to emit the event once a handler is assigned
  process.nextTick(function() {
    this.emit('event');
  }.bind(this));
}
util.inherits(MyEmitter, EventEmitter);

const myEmitter = new MyEmitter();
myEmitter.on('event', function() {
  console.log('an event occurred!');
});
```