# Informazioni riguardo questa Documentazione

<!--introduced_in=v0.10.0-->

<!-- type=misc -->

L'obiettivo di questa documentazione è di spiegare in modo esaustivo l'API Node.js, sia dal punto di vista di avere un riferimento su cui basarsi sia da un punto di vista concettuale. Ogni sezione descrive un modulo integrato o un concetto di alto livello.

Dove più appropriato, i tipi delle proprietà, gli argomenti dei metodi e gli argomenti forniti agli event handler sono descritti in modo dettagliato in un elenco al di sotto dell'intestazione del topic.

## Contribuire

Se vengono trovati degli errori all'interno di questa documentazione, siete pregati di [inviare un issue](https://github.com/nodejs/node/issues/new) o di vedere [la guida su come contribuire](https://github.com/nodejs/node/blob/master/CONTRIBUTING.md) per avere indicazioni su come inserire una patch.

Ogni file viene generato in base al corrispondente file `.md` nella cartella `doc/api/` nel source tree di Node.js. La documentazione viene generata utilizzando il programma `tools/doc/generate.js`. All'interno di `doc/template.html` c'è un template HTML.

## Indice di Stabilità

<!--type=misc-->

All'interno di tutta la documentazione ci sono indicazioni riguardo la stabilità di ogni sezione. L'API Node.js è ancora in qualche modo in evoluzione e, con il passare del tempo, alcune parti sono più affidabili e sicure di altre. Alcune sono talmente testate e affidabili che è improbabile possano mai cambiare. Altre invece sono nuove e sperimentali oppure note per essere pericolosi e in fase di ri-progettazione.

Gli indici di stabilità sono i seguenti:

```txt
Stabilità: 0 - Obsoleto. Questa funzione è nota per essere problematica ed è una funzione alla quale potrebbero essere pianificate delle modifiche. Non bisogna farne affidamento. L'utilizzo di questa funzione potrebbe causare l'emissione di avvisi. Per le versioni principali non è prevista la retrocompatibilità.
```

```txt
Stabilità: 1 - Sperimentale. Questa funzione è ancora in fase di sviluppo attivo e soggetta a modifiche non retrocompatibili, o addirittura alla rimozione, in qualsiasi versione futura. L'utilizzo di questa funzione non è raccomandato negli ambienti di produzione.
Le funzionalità sperimentali non sono soggette al modello di versione semantica di Node.js.
```

```txt
Stabilità: 2 - Stabile. L'API si è dimostrata soddisfacente. La compatibilità con l'ecosistema npm è una priorità elevata e non verrà interrotta se non strettamente necessario.
```

Caution must be used when making use of `Experimental` features, particularly within modules that may be used as dependencies (or dependencies of dependencies) within a Node.js application. End users may not be aware that experimental features are being used, and therefore may experience unexpected failures or behavior changes when API modifications occur. To help avoid such surprises, `Experimental` features may require a command-line flag to explicitly enable them, or may cause a process warning to be emitted. By default, such warnings are printed to [`stderr`][] and may be handled by attaching a listener to the [`'warning'`][] event.

## JSON Output

<!-- YAML
added: v0.6.12
-->

> Stability: 1 - Experimental

Every `.html` document has a corresponding `.json` document presenting the same information in a structured manner. This feature is experimental, and added for the benefit of IDEs and other utilities that wish to do programmatic things with the documentation.

## Syscalls and man pages

System calls like open(2) and read(2) define the interface between user programs and the underlying operating system. Node.js functions which simply wrap a syscall, like [`fs.open()`][], will document that. The docs link to the corresponding man pages (short for manual pages) which describe how the syscalls work.

Some syscalls, like lchown(2), are BSD-specific. That means, for example, that [`fs.lchown()`][] only works on macOS and other BSD-derived systems, and is not available on Linux.

Most Unix syscalls have Windows equivalents, but behavior may differ on Windows relative to Linux and macOS. For an example of the subtle ways in which it's sometimes impossible to replace Unix syscall semantics on Windows, see [Node issue 4760](https://github.com/nodejs/node/issues/4760).