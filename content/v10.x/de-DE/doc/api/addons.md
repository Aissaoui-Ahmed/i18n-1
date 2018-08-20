# C++ Erweiterungen

<!--introduced_in=v0.10.0-->

<!-- type=misc -->

Node.js-Erweiterungen sind dynamisch-verknüpfte freigegebene Objekte, die in C++ geschrieben sind und in Node.js mit der [`require()`](modules.html#modules_require) Funktion geladen und verwendet werden können, als wären sie ein gewöhnliches Node.js Modul. Sie werden in erster Linie dazu eingesetzt, um eine Schnittstelle zwischen JavaScript in Node.js und C/C++ Bibliotheken zu bieten.

Im Moment ist die Methode zur Implementierung von Erweiterungen eher kompliziert, sie bedarf Kenntnisse über verschiedene Komponenten und Programmierschnittstellen:

* V8: die C++ Bibliothek Node.js verwendet diese zurzeit, um eine JavaScript-Implementierung bereitzustellen. V8 stellt den Mechanismus für die Erstellung von Objekten, das Abrufen von Funktionen etc. bereit. Die Programmierschnittstelle von V8 wird zum größten Teil in der `v8.h` header file (`deps/v8/include/v8.h` in dem Node.js Source-Tree) dokumentiert, die auch [online](https://v8docs.nodesource.com/) verfügbar ist.

* [libuv](https://github.com/libuv/libuv): Die C Bibliothek, die die Node.js Event-Schleife, Arbeiter-Threads und alle asynchronen Verhaltensweisen der Plattform implementiert. Sie dient auch als plattformübergreifende Abstraktionsbibliothek, die einen einfachen, POSIX-ähnlichen Zugriff über alle wichtigen Betriebssysteme hinweg auf viele gängige Systemaufgaben, wie z.B. Interaktion mit dem Dateisystem, Sockets, Timern und Systemereignissen gewährt. libuv bietet auch eine pthreads-ähnliche Threading-Abstraktion, die verwendet werden kann, um anspruchsvollere asynchrone Erweiterungen zu betreiben, die über die Standard-Event-Schleife hinausgehen müssen. Autoren von Erweiterungen werden ermutigt, darüber nachzudenken, wie man es vermeiden kann, die Event-Schleife mit I/O oder anderen zeitintensiven Aufgaben zu blockieren, indem Sie die Arbeit via libuv auf nicht-blockierende Systemoperationen, Arbeiter-Threads oder eine benutzerdefinierte Verwendung der libuv-Threads auslagern.

* Interne Node.js-Bibliotheken. Node.js selbst exportiert eine Reihe von C++ Programmierschnittstellen, die von Erweiterungen verwendet werden können &mdash; von denen die Wichtigste die `node::ObjectWrap` Klasse ist.

* Node.js enthält eine Reihe weiterer statisch verknüpfter Bibliotheken, darunter OpenSSL. Diese anderen Bibliotheken befinden sich in dem `deps/` Verzeichnis im Node.js Source-Tree. Nur die Symbole libuv, OpenSSL, V8 und zlib werden von Node.js gezielt wieder exportiert und können von Addons in unterschiedlichem Umfang verwendet werden. Weitere Informationen finden Sie unter [Verknüpfung zu Node.js' eigene Abhängigkeiten](#addons_linking_to_node_js_own_dependencies).

Alle der folgenden Beispiele stehen zum [Download](https://github.com/nodejs/node-addon-examples) bereit und können als Ausgangspunkt für eine Erweiterung verwendet werden.

## Hallo Welt

Dieses "Hallo Welt"-Beispiel ist eine einfache, in C++ geschriebene, Erweiterung, die dem folgenden JavaScript-Code entspricht:

```js
module.exports.hello = () => 'world';
```

Erstellen Sie zunächst die Datei `hello.cc`:

```cpp
// hello.cc
#include <node.h>

namespace demo {

using v8::FunctionCallbackInfo;
using v8::Isolate;
using v8::Local;
using v8::Object;
using v8::String;
using v8::Value;

void Method(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();
  args.GetReturnValue().Set(String::NewFromUtf8(isolate, "world"));
}

void init(Local<Object> exports) {
  NODE_SET_METHOD(exports, "hello", Method);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, init)

}  // namespace demo
```

Beachten Sie, dass alle Node.js-Erweiterungen eine Initialisierungsfunktion nach folgendem Muster exportieren müssen:

```cpp
void Initialize(Local<Object> exports);
NODE_MODULE(NODE_GYP_MODULE_NAME, Initialize)
```

Es gibt keinen Strichpunkt nach `NODE_MODULE`, da es keine Funktion ist (siehe `node.h`).

Der `module_name` muss mit dem Dateinamen der endgültigen Binärdatei übereinstimmen (außer dem Suffix `.node`).

Im Beispiel `hello.cc` ist die Initialisierungsfunktion `init` und der Erweiterungsmodulname `addon`.

### Aufbau

Nachdem der Quellcode geschrieben wurde, muss er in die Binärdatei `addon.node` kompiliert werden. Erstellen Sie dazu eine Datei mit dem Namen `binding.gyp` in der obersten Ebene des Projekts, die die Build-Konfiguration des Moduls in einem JSON-ähnlichen Format beschreibt. Diese Datei wird von [node-gyp](https://github.com/nodejs/node-gyp) verwendet — einem Tool, das speziell für die Kompilierung von Node.js-Erweiterungen geschrieben wurde.

```json
{
  "targets": [
    {
      "target_name": "addon",
      "sources": [ "hello.cc" ]
    }
  ]
}
```

Eine Version des Programmes `node-gyp` wird mit Node.js als Teil von `npm` gebündelt und verteilt. Diese Version wird den Entwicklern nicht direkt zur Verfügung gestellt und soll nur die Möglichkeit bieten, den Befehl `npm install` zum Kompilieren und Installieren von Erweiterungen zu verwenden. Entwickler, die `node-gyp` direkt verwenden möchten, können es mit dem Befehl `npm install -g node-gyp` installieren. Sehen Sie die `node-gyp` [Installationsanleitung](https://github.com/nodejs/node-gyp#installation), einschließlich plattformspezifischer Anforderungen, für weitere Informationen.

Nachdem die Datei `binding.gyp` erstellt wurde, verwenden Sie `node-gyp configure`, um die entsprechenden Projekt-Build-Dateien für die aktuelle Plattform zu erzeugen. Dies erzeugt entweder eine `Makefile` (auf Unix-Plattformen) oder eine `vcxproj` Datei (auf Windows) im Verzeichnis `build/`.

Als nächstes rufen Sie den Befehl `node-gyp build` auf, um die übersetzte Datei `addon.node` zu erzeugen. Diese wird in das Verzeichnis `build/Release/` abgelegt.

Wenn Sie `npm install` verwenden, um ein Node.js-Erweiterung zu installieren, verwendet npm seine eigene Paketversion von `node-gyp`, um das gleiche Set von Aktionen auszuführen und eine kompilierte Version der Erweiterung für die Plattform des Benutzers bei Bedarf zu erzeugen.

Einmal erstellt, kann die binäre Erweiterung aus Node.js heraus verwendet werden, indem man [`require()`](modules.html#modules_require) auf das gebaute `addon.node`-Modul verweist:

```js
// hello.js
const addon = require('./build/Release/addon');

console.log(addon.hello());
// Prints: 'world'
```

Weitere Informationen finden Sie in den folgenden Beispielen oder unter [https://github.com/arturadib/node qt](https://github.com/arturadib/node-qt) für ein Anwendungsbeispiel in der Praxis.

Da der genaue Pfad zur kompilierten Erweiterungs-Binary je nach Kompilierung variieren kann (d.h. manchmal in `./build/Debug/`), können Erweiterungen das Paket [bindings](https://github.com/TooTallNate/node-bindings) verwenden, um das kompilierte Modul zu laden.

Beachten Sie, dass die `bindings`-Paketimplementierung zwar anspruchsvoller ist, wie sie Erweiterungs-Module lokalisiert, aber im Wesentlichen ein Try-Catch-Muster nutzt, ähnlich dem:

```js
try {
  return require('./build/Release/addon.node');
} catch (err) {
  return require('./build/Debug/addon.node');
}
```

### Verknüpfung mit den eigenen Abhängigkeitsbeziehungen von Node.js

Node.js verwendet eine Reihe von statisch-gelinkten Bibliotheken wie V8, libuv und OpenSSL. Alle Erweiterungen müssen auf V8 verweisen und können auch auf andere Abhängigkeitsbeziehungen verweisen. Normalerweise ist dies so einfach wie das Einfügen der entsprechenden `#include <...>` Anweisungen (z.B. `#include <v8.h> `) und `node-gyp` findet die entsprechenden Überschriften automatisch. Es gibt jedoch einige Einschränkungen zu beachten:

* Wenn `node-gyp` läuft, erkennt es die spezifische Version von Node.js und lädt entweder den vollständigen Quelltext oder nur die Überschriften herunter. Wenn der vollständige Quellcode heruntergeladen wird, haben Erweiterungen vollständigen Zugriff auf das gesamte Set der Abhängigkeitsbeziehungen von Node.js. Wenn jedoch nur die Node.js-Überschriften heruntergeladen werden, stehen nur die von Node.js exportierten Symbole zur Verfügung.

* `node-gyp` kann mit dem `--nodedir` Flag ausgeführt werden, das auf ein lokales Node.js Quellbild verweist. Mit dieser Option hat die Erweiterung Zugriff auf das volle Set von Abhängigkeitsbeziehungen.

### Laden von Erweiterungen mit require()

Die Dateinamenerweiterung des kompilierten Erweiterungs-Binary ist `.node` (im Gegensatz zu `.dll` oder `.so`). Die Funktion [`require()`](modules.html#modules_require) wird geschrieben, um nach Dateien mit der Dateiendung `.node` zu suchen und diese als dynamisch verknüpfte Bibliotheken zu initialisieren.

Beim Aufruf von [`require()`](modules.html#modules_require) kann die Erweiterung `.node` in der Regel weggelassen werden und Node.js wird die Erweiterung trotzdem finden und initialisieren. Ein Nachteil ist jedoch, dass Node.js zuerst versucht, Module oder JavaScript-Dateien zu finden und zu laden, die den gleichen Basisnamen haben. Zum Beispiel, wenn es eine Datei namens `addon.js` im selben Verzeichnis wie die binäre `addon.node`-Datei gibt, dann wird [`require('addon')`](modules.html#modules_require) der Datei `addon.js` Vorrang geben und sie stattdessen laden.

## Native Abstraktionen für Node.js

Jedes der in diesem Dokument dargestellten Beispiele verwendet direkt die Node.js und die V8-Programmierschnittstellen für die Implementierung von Erweiterungen. Es ist wichtig zu verstehen, dass sich die V8-Programmierschnittstelle von einer V8-Version zur nächsten (und von einer großen Node.js-Version zur nächsten) grundlegend verändern kann und tatsächlich verändert hat. Bei jeder Änderung müssen Erweiterungen aktualisiert und neu kompiliert werden, damit sie weiter funktionieren. Der Zeitplan für die Veröffentlichung von Node.js wurde entwickelt, um die Häufigkeit und die Auswirkungen solcher Änderungen zu minimieren, aber es gibt wenig, was Node.js derzeit tun kann, um die Stabilität der V8-Programmierschnittstellen zu gewährleisten.

Die [Nativen Abstraktionen für Node.js](https://github.com/nodejs/nan) (oder `nan`) bieten eine Reihe von Tools, die Erweiterungs-Entwicklern empfohlen werden, um die Kompatibilität zwischen früheren und zukünftigen Versionen von V8 und Node.js zu gewährleisten. Siehe `nan` [Beispiele](https://github.com/nodejs/nan/tree/master/examples/) für eine Veranschaulichung, wie sie verwendet werden können.

## N-API

> Stabilität: 1 - Experimentell

N-API ist eine Programmierschnittstelle zum Erstellen von nativen Erweiterungen. Sie ist unabhängig von der zugrundeliegenden JavaScript-Runtime (z.B. V8) und wird als Teil von Node.js selbst gepflegt. Diese Programmierschnittstelle wird über alle Versionen von Node.js hinweg, Application-Binary-Interface-stabil (ABI) sein. Es ist vorgesehen, Erweiterungen von Änderungen in der zugrunde liegenden JavaScript-Engine zu isolieren und es zu ermöglichen, dass Module, die für eine Version kompiliert wurden, auf späteren Versionen von Node.js ohne Neukompilierung ausgeführt werden können. Erweiterungen werden mit den in diesem Dokument beschriebenen Methoden und Werkzeugen (node-gyp, etc.) erstellt. Der einzige Unterschied ist das Set an Programmierschnittstellen, die von dem ursprünglichen Code verwendet werden. Anstatt die V8 oder die [Nativen Abstraktionen für Node.js](https://github.com/nodejs/nan)-Programmierschnittstellen zu verwenden, werden die in der N-API verfügbaren Funktionen verwendet.

Um N-API im obigen Beispiel "Hello world" zu verwenden, ersetzen Sie den Inhalt von `hello.cc` durch den folgenden. Alle anderen Anweisungen bleiben dieselben.

```cpp
// hello.cc using N-API
#include <node_api.h>

namespace demo {

napi_value Method(napi_env env, napi_callback_info args) {
  napi_value greeting;
  napi_status status;

  status = napi_create_string_utf8(env, "hello", NAPI_AUTO_LENGTH, &greeting);
  if (status != napi_ok) return nullptr;
  return greeting;
}

napi_value init(napi_env env, napi_value exports) {
  napi_status status;
  napi_value fn;

  status = napi_create_function(env, nullptr, 0, Method, nullptr, &fn);
  if (status != napi_ok) return nullptr;

  status = napi_set_named_property(env, exports, "hello", fn);
  if (status != napi_ok) return nullptr;
  return exports;
}

NAPI_MODULE(NODE_GYP_MODULE_NAME, init)

}  // namespace demo
```

Die verfügbaren Funktionen und deren Verwendung sind im Abschnitt [C/C++ Erweiterungen - N-API](n-api.html) dokumentiert.

## Beispiele für Erweiterungen

Nachfolgend finden Sie einige Beispiele für Erweiterungen, die den Entwicklern den Einstieg erleichtern sollen. Die Beispiele nutzen die V8-Programmierschnittstellen. Für eine Erklärung verschiedener Konzepte wie Handles, Scopes, Funktionsvorlagen, etc. und für Hilfe bei den verschiedenen V8-Calls siehe die [V8-Referenz](https://v8docs.nodesource.com/) und V8's [Einbettungsanleitung](https://github.com/v8/v8/wiki/Embedder's%20Guide).

Jedes dieser Beispiele nutzt die folgende `binding.gyp`-Datei:

```json
{
  "targets": [
    {
      "target_name": "addon",
      "sources": [ "addon.cc" ]
    }
  ]
}
```

In Fällen, in denen es mehr als eine `.cc`-Datei gibt, fügen Sie einfach den zusätzlichen Dateinamen zum `sources`-Array hinzu:

```json
"sources": ["addon.cc", "myexample.cc"]
```

Sobald die Datei `binding.gyp` fertig ist, können die Beispielserweiterungen mit `node-gyp` konfiguriert und gebaut werden:

```console
$ node-gyp configure build
```

### Function arguments

Addons will typically expose objects and functions that can be accessed from JavaScript running within Node.js. When functions are invoked from JavaScript, the input arguments and return value must be mapped to and from the C/C++ code.

The following example illustrates how to read function arguments passed from JavaScript and how to return a result:

```cpp
// addon.cc
#include <node.h>

namespace demo {

using v8::Exception;
using v8::FunctionCallbackInfo;
using v8::Isolate;
using v8::Local;
using v8::Number;
using v8::Object;
using v8::String;
using v8::Value;

// This is the implementation of the "add" method
// Input arguments are passed using the
// const FunctionCallbackInfo<Value>& args struct
void Add(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();

  // Check the number of arguments passed.
  if (args.Length() < 2) {
    // Throw an Error that is passed back to JavaScript
    isolate->ThrowException(Exception::TypeError(
        String::NewFromUtf8(isolate, "Wrong number of arguments")));
    return;
  }

  // Check the argument types
  if (!args[0]->IsNumber() || !args[1]->IsNumber()) {
    isolate->ThrowException(Exception::TypeError(
        String::NewFromUtf8(isolate, "Wrong arguments")));
    return;
  }

  // Perform the operation
  double value = args[0]->NumberValue() + args[1]->NumberValue();
  Local<Number> num = Number::New(isolate, value);

  // Set the return value (using the passed in
  // FunctionCallbackInfo<Value>&)
  args.GetReturnValue().Set(num);
}

void Init(Local<Object> exports) {
  NODE_SET_METHOD(exports, "add", Add);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, Init)

}  // namespace demo
```

Once compiled, the example Addon can be required and used from within Node.js:

```js
// test.js
const addon = require('./build/Release/addon');

console.log('This should be eight:', addon.add(3, 5));
```

### Callbacks

It is common practice within Addons to pass JavaScript functions to a C++ function and execute them from there. The following example illustrates how to invoke such callbacks:

```cpp
// addon.cc
#include <node.h>

namespace demo {

using v8::Function;
using v8::FunctionCallbackInfo;
using v8::Isolate;
using v8::Local;
using v8::Null;
using v8::Object;
using v8::String;
using v8::Value;

void RunCallback(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();
  Local<Function> cb = Local<Function>::Cast(args[0]);
  const unsigned argc = 1;
  Local<Value> argv[argc] = { String::NewFromUtf8(isolate, "hello world") };
  cb->Call(Null(isolate), argc, argv);
}

void Init(Local<Object> exports, Local<Object> module) {
  NODE_SET_METHOD(module, "exports", RunCallback);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, Init)

}  // namespace demo
```

Note that this example uses a two-argument form of `Init()` that receives the full `module` object as the second argument. This allows the Addon to completely overwrite `exports` with a single function instead of adding the function as a property of `exports`.

To test it, run the following JavaScript:

```js
// test.js
const addon = require('./build/Release/addon');

addon((msg) => {
  console.log(msg);
// Prints: 'hello world'
});
```

Note that, in this example, the callback function is invoked synchronously.

### Object factory

Addons can create and return new objects from within a C++ function as illustrated in the following example. An object is created and returned with a property `msg` that echoes the string passed to `createObject()`:

```cpp
// addon.cc
#include <node.h>

namespace demo {

using v8::FunctionCallbackInfo;
using v8::Isolate;
using v8::Local;
using v8::Object;
using v8::String;
using v8::Value;

void CreateObject(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();

  Local<Object> obj = Object::New(isolate);
  obj->Set(String::NewFromUtf8(isolate, "msg"), args[0]->ToString());

  args.GetReturnValue().Set(obj);
}

void Init(Local<Object> exports, Local<Object> module) {
  NODE_SET_METHOD(module, "exports", CreateObject);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, Init)

}  // namespace demo
```

To test it in JavaScript:

```js
// test.js
const addon = require('./build/Release/addon');

const obj1 = addon('hello');
const obj2 = addon('world');
console.log(obj1.msg, obj2.msg);
// Prints: 'hello world'
```

### Function factory

Another common scenario is creating JavaScript functions that wrap C++ functions and returning those back to JavaScript:

```cpp
// addon.cc
#include <node.h>

namespace demo {

using v8::Function;
using v8::FunctionCallbackInfo;
using v8::FunctionTemplate;
using v8::Isolate;
using v8::Local;
using v8::Object;
using v8::String;
using v8::Value;

void MyFunction(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();
  args.GetReturnValue().Set(String::NewFromUtf8(isolate, "hello world"));
}

void CreateFunction(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();

  Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, MyFunction);
  Local<Function> fn = tpl->GetFunction();

  // omit this to make it anonymous
  fn->SetName(String::NewFromUtf8(isolate, "theFunction"));

  args.GetReturnValue().Set(fn);
}

void Init(Local<Object> exports, Local<Object> module) {
  NODE_SET_METHOD(module, "exports", CreateFunction);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, Init)

}  // namespace demo
```

To test:

```js
// test.js
const addon = require('./build/Release/addon');

const fn = addon();
console.log(fn());
// Prints: 'hello world'
```

### Wrapping C++ objects

It is also possible to wrap C++ objects/classes in a way that allows new instances to be created using the JavaScript `new` operator:

```cpp
// addon.cc
#include <node.h>
#include "myobject.h"

namespace demo {

using v8::Local;
using v8::Object;

void InitAll(Local<Object> exports) {
  MyObject::Init(exports);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, InitAll)

}  // namespace demo
```

Then, in `myobject.h`, the wrapper class inherits from `node::ObjectWrap`:

```cpp
// myobject.h
#ifndef MYOBJECT_H
#define MYOBJECT_H

#include <node.h>
#include <node_object_wrap.h>

namespace demo {

class MyObject : public node::ObjectWrap {
 public:
  static void Init(v8::Local<v8::Object> exports);

 private:
  explicit MyObject(double value = 0);
  ~MyObject();

  static void New(const v8::FunctionCallbackInfo<v8::Value>& args);
  static void PlusOne(const v8::FunctionCallbackInfo<v8::Value>& args);
  static v8::Persistent<v8::Function> constructor;
  double value_;
};

}  // namespace demo

#endif
```

In `myobject.cc`, implement the various methods that are to be exposed. Below, the method `plusOne()` is exposed by adding it to the constructor's prototype:

```cpp
// myobject.cc
#include "myobject.h"

namespace demo {

using v8::Context;
using v8::Function;
using v8::FunctionCallbackInfo;
using v8::FunctionTemplate;
using v8::Isolate;
using v8::Local;
using v8::Number;
using v8::Object;
using v8::Persistent;
using v8::String;
using v8::Value;

Persistent<Function> MyObject::constructor;

MyObject::MyObject(double value) : value_(value) {
}

MyObject::~MyObject() {
}

void MyObject::Init(Local<Object> exports) {
  Isolate* isolate = exports->GetIsolate();

  // Prepare constructor template
  Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, New);
  tpl->SetClassName(String::NewFromUtf8(isolate, "MyObject"));
  tpl->InstanceTemplate()->SetInternalFieldCount(1);

  // Prototype
  NODE_SET_PROTOTYPE_METHOD(tpl, "plusOne", PlusOne);

  constructor.Reset(isolate, tpl->GetFunction());
  exports->Set(String::NewFromUtf8(isolate, "MyObject"),
               tpl->GetFunction());
}

void MyObject::New(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();

  if (args.IsConstructCall()) {
    // Invoked as constructor: `new MyObject(...)`
    double value = args[0]->IsUndefined() ? 0 : args[0]->NumberValue();
    MyObject* obj = new MyObject(value);
    obj->Wrap(args.This());
    args.GetReturnValue().Set(args.This());
  } else {
    // Invoked as plain function `MyObject(...)`, turn into construct call.
    const int argc = 1;
    Local<Value> argv[argc] = { args[0] };
    Local<Context> context = isolate->GetCurrentContext();
    Local<Function> cons = Local<Function>::New(isolate, constructor);
    Local<Object> result =
        cons->NewInstance(context, argc, argv).ToLocalChecked();
    args.GetReturnValue().Set(result);
  }
}

void MyObject::PlusOne(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();

  MyObject* obj = ObjectWrap::Unwrap<MyObject>(args.Holder());
  obj->value_ += 1;

  args.GetReturnValue().Set(Number::New(isolate, obj->value_));
}

}  // namespace demo
```

To build this example, the `myobject.cc` file must be added to the `binding.gyp`:

```json
{
  "targets": [
    {
      "target_name": "addon",
      "sources": [
        "addon.cc",
        "myobject.cc"
      ]
    }
  ]
}
```

Test it with:

```js
// test.js
const addon = require('./build/Release/addon');

const obj = new addon.MyObject(10);
console.log(obj.plusOne());
// Prints: 11
console.log(obj.plusOne());
// Prints: 12
console.log(obj.plusOne());
// Prints: 13
```

### Factory of wrapped objects

Alternatively, it is possible to use a factory pattern to avoid explicitly creating object instances using the JavaScript `new` operator:

```js
const obj = addon.createObject();
// instead of:
// const obj = new addon.Object();
```

First, the `createObject()` method is implemented in `addon.cc`:

```cpp
// addon.cc
#include <node.h>
#include "myobject.h"

namespace demo {

using v8::FunctionCallbackInfo;
using v8::Isolate;
using v8::Local;
using v8::Object;
using v8::String;
using v8::Value;

void CreateObject(const FunctionCallbackInfo<Value>& args) {
  MyObject::NewInstance(args);
}

void InitAll(Local<Object> exports, Local<Object> module) {
  MyObject::Init(exports->GetIsolate());

  NODE_SET_METHOD(module, "exports", CreateObject);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, InitAll)

}  // namespace demo
```

In `myobject.h`, the static method `NewInstance()` is added to handle instantiating the object. This method takes the place of using `new` in JavaScript:

```cpp
// myobject.h
#ifndef MYOBJECT_H
#define MYOBJECT_H

#include <node.h>
#include <node_object_wrap.h>

namespace demo {

class MyObject : public node::ObjectWrap {
 public:
  static void Init(v8::Isolate* isolate);
  static void NewInstance(const v8::FunctionCallbackInfo<v8::Value>& args);

 private:
  explicit MyObject(double value = 0);
  ~MyObject();

  static void New(const v8::FunctionCallbackInfo<v8::Value>& args);
  static void PlusOne(const v8::FunctionCallbackInfo<v8::Value>& args);
  static v8::Persistent<v8::Function> constructor;
  double value_;
};

}  // namespace demo

#endif
```

The implementation in `myobject.cc` is similar to the previous example:

```cpp
// myobject.cc
#include <node.h>
#include "myobject.h"

namespace demo {

using v8::Context;
using v8::Function;
using v8::FunctionCallbackInfo;
using v8::FunctionTemplate;
using v8::Isolate;
using v8::Local;
using v8::Number;
using v8::Object;
using v8::Persistent;
using v8::String;
using v8::Value;

Persistent<Function> MyObject::constructor;

MyObject::MyObject(double value) : value_(value) {
}

MyObject::~MyObject() {
}

void MyObject::Init(Isolate* isolate) {
  // Prepare constructor template
  Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, New);
  tpl->SetClassName(String::NewFromUtf8(isolate, "MyObject"));
  tpl->InstanceTemplate()->SetInternalFieldCount(1);

  // Prototype
  NODE_SET_PROTOTYPE_METHOD(tpl, "plusOne", PlusOne);

  constructor.Reset(isolate, tpl->GetFunction());
}

void MyObject::New(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();

  if (args.IsConstructCall()) {
    // Invoked as constructor: `new MyObject(...)`
    double value = args[0]->IsUndefined() ? 0 : args[0]->NumberValue();
    MyObject* obj = new MyObject(value);
    obj->Wrap(args.This());
    args.GetReturnValue().Set(args.This());
  } else {
    // Invoked as plain function `MyObject(...)`, turn into construct call.
    const int argc = 1;
    Local<Value> argv[argc] = { args[0] };
    Local<Function> cons = Local<Function>::New(isolate, constructor);
    Local<Context> context = isolate->GetCurrentContext();
    Local<Object> instance =
        cons->NewInstance(context, argc, argv).ToLocalChecked();
    args.GetReturnValue().Set(instance);
  }
}

void MyObject::NewInstance(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();

  const unsigned argc = 1;
  Local<Value> argv[argc] = { args[0] };
  Local<Function> cons = Local<Function>::New(isolate, constructor);
  Local<Context> context = isolate->GetCurrentContext();
  Local<Object> instance =
      cons->NewInstance(context, argc, argv).ToLocalChecked();

  args.GetReturnValue().Set(instance);
}

void MyObject::PlusOne(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();

  MyObject* obj = ObjectWrap::Unwrap<MyObject>(args.Holder());
  obj->value_ += 1;

  args.GetReturnValue().Set(Number::New(isolate, obj->value_));
}

}  // namespace demo
```

Once again, to build this example, the `myobject.cc` file must be added to the `binding.gyp`:

```json
{
  "targets": [
    {
      "target_name": "addon",
      "sources": [
        "addon.cc",
        "myobject.cc"
      ]
    }
  ]
}
```

Test it with:

```js
// test.js
const createObject = require('./build/Release/addon');

const obj = createObject(10);
console.log(obj.plusOne());
// Prints: 11
console.log(obj.plusOne());
// Prints: 12
console.log(obj.plusOne());
// Prints: 13

const obj2 = createObject(20);
console.log(obj2.plusOne());
// Prints: 21
console.log(obj2.plusOne());
// Prints: 22
console.log(obj2.plusOne());
// Prints: 23
```

### Passing wrapped objects around

In addition to wrapping and returning C++ objects, it is possible to pass wrapped objects around by unwrapping them with the Node.js helper function `node::ObjectWrap::Unwrap`. The following examples shows a function `add()` that can take two `MyObject` objects as input arguments:

```cpp
// addon.cc
#include <node.h>
#include <node_object_wrap.h>
#include "myobject.h"

namespace demo {

using v8::FunctionCallbackInfo;
using v8::Isolate;
using v8::Local;
using v8::Number;
using v8::Object;
using v8::String;
using v8::Value;

void CreateObject(const FunctionCallbackInfo<Value>& args) {
  MyObject::NewInstance(args);
}

void Add(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();

  MyObject* obj1 = node::ObjectWrap::Unwrap<MyObject>(
      args[0]->ToObject());
  MyObject* obj2 = node::ObjectWrap::Unwrap<MyObject>(
      args[1]->ToObject());

  double sum = obj1->value() + obj2->value();
  args.GetReturnValue().Set(Number::New(isolate, sum));
}

void InitAll(Local<Object> exports) {
  MyObject::Init(exports->GetIsolate());

  NODE_SET_METHOD(exports, "createObject", CreateObject);
  NODE_SET_METHOD(exports, "add", Add);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, InitAll)

}  // namespace demo
```

In `myobject.h`, a new public method is added to allow access to private values after unwrapping the object.

```cpp
// myobject.h
#ifndef MYOBJECT_H
#define MYOBJECT_H

#include <node.h>
#include <node_object_wrap.h>

namespace demo {

class MyObject : public node::ObjectWrap {
 public:
  static void Init(v8::Isolate* isolate);
  static void NewInstance(const v8::FunctionCallbackInfo<v8::Value>& args);
  inline double value() const { return value_; }

 private:
  explicit MyObject(double value = 0);
  ~MyObject();

  static void New(const v8::FunctionCallbackInfo<v8::Value>& args);
  static v8::Persistent<v8::Function> constructor;
  double value_;
};

}  // namespace demo

#endif
```

The implementation of `myobject.cc` is similar to before:

```cpp
// myobject.cc
#include <node.h>
#include "myobject.h"

namespace demo {

using v8::Context;
using v8::Function;
using v8::FunctionCallbackInfo;
using v8::FunctionTemplate;
using v8::Isolate;
using v8::Local;
using v8::Object;
using v8::Persistent;
using v8::String;
using v8::Value;

Persistent<Function> MyObject::constructor;

MyObject::MyObject(double value) : value_(value) {
}

MyObject::~MyObject() {
}

void MyObject::Init(Isolate* isolate) {
  // Prepare constructor template
  Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, New);
  tpl->SetClassName(String::NewFromUtf8(isolate, "MyObject"));
  tpl->InstanceTemplate()->SetInternalFieldCount(1);

  constructor.Reset(isolate, tpl->GetFunction());
}

void MyObject::New(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();

  if (args.IsConstructCall()) {
    // Invoked as constructor: `new MyObject(...)`
    double value = args[0]->IsUndefined() ? 0 : args[0]->NumberValue();
    MyObject* obj = new MyObject(value);
    obj->Wrap(args.This());
    args.GetReturnValue().Set(args.This());
  } else {
    // Invoked as plain function `MyObject(...)`, turn into construct call.
    const int argc = 1;
    Local<Value> argv[argc] = { args[0] };
    Local<Context> context = isolate->GetCurrentContext();
    Local<Function> cons = Local<Function>::New(isolate, constructor);
    Local<Object> instance =
        cons->NewInstance(context, argc, argv).ToLocalChecked();
    args.GetReturnValue().Set(instance);
  }
}

void MyObject::NewInstance(const FunctionCallbackInfo<Value>& args) {
  Isolate* isolate = args.GetIsolate();

  const unsigned argc = 1;
  Local<Value> argv[argc] = { args[0] };
  Local<Function> cons = Local<Function>::New(isolate, constructor);
  Local<Context> context = isolate->GetCurrentContext();
  Local<Object> instance =
      cons->NewInstance(context, argc, argv).ToLocalChecked();

  args.GetReturnValue().Set(instance);
}

}  // namespace demo
```

Test it with:

```js
// test.js
const addon = require('./build/Release/addon');

const obj1 = addon.createObject(10);
const obj2 = addon.createObject(20);
const result = addon.add(obj1, obj2);

console.log(result);
// Prints: 30
```

### AtExit hooks

An `AtExit` hook is a function that is invoked after the Node.js event loop has ended but before the JavaScript VM is terminated and Node.js shuts down. `AtExit` hooks are registered using the `node::AtExit` API.

#### void AtExit(callback, args)

* `callback` <span class="type">&lt;void (\<em>)(void\</em>)&gt;</span> A pointer to the function to call at exit.
* `args` <span class="type">&lt;void\*&gt;</span> A pointer to pass to the callback at exit.

Registers exit hooks that run after the event loop has ended but before the VM is killed.

`AtExit` takes two parameters: a pointer to a callback function to run at exit, and a pointer to untyped context data to be passed to that callback.

Callbacks are run in last-in first-out order.

The following `addon.cc` implements `AtExit`:

```cpp
// addon.cc
#include <assert.h>
#include <stdlib.h>
#include <node.h>

namespace demo {

using node::AtExit;
using v8::HandleScope;
using v8::Isolate;
using v8::Local;
using v8::Object;

static char cookie[] = "yum yum";
static int at_exit_cb1_called = 0;
static int at_exit_cb2_called = 0;

static void at_exit_cb1(void* arg) {
  Isolate* isolate = static_cast<Isolate*>(arg);
  HandleScope scope(isolate);
  Local<Object> obj = Object::New(isolate);
  assert(!obj.IsEmpty());  // assert VM is still alive
  assert(obj->IsObject());
  at_exit_cb1_called++;
}

static void at_exit_cb2(void* arg) {
  assert(arg == static_cast<void*>(cookie));
  at_exit_cb2_called++;
}

static void sanity_check(void*) {
  assert(at_exit_cb1_called == 1);
  assert(at_exit_cb2_called == 2);
}

void init(Local<Object> exports) {
  AtExit(at_exit_cb2, cookie);
  AtExit(at_exit_cb2, cookie);
  AtExit(at_exit_cb1, exports->GetIsolate());
  AtExit(sanity_check);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, init)

}  // namespace demo
```

Test in JavaScript by running:

```js
// test.js
require('./build/Release/addon');
```