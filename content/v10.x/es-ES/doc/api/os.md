# SO

<!--introduced_in=v0.10.0-->

> Estabilidad: 2 - Estable

El módulo `so` provee un número de métodos de utilidad relacionados al sistema operativo. Puede ser accesado usando:

```js
const os = require('os');
```

## os.EOL

<!-- YAML
added: v0.7.8
-->

* {string}

Una constante de linea definiendo el marcador de fin de linea específico del sistema operativo:

* `\n` en POSIX
* `\r\n\r\n` en Windows

## os.arch()

<!-- YAML
added: v0.5.0
-->

* Retorna: {string}

El método `os.arch()` retorna una línea identificando la arquitectura del CPU del sistema operativo para el cual el binario de Node.js fue compilado.

Los actuales valores posibles son: `'arm'`, `'arm64'`, `'ia32'`, `'mips'`, `'mipsel'`, `'ppc'`, `'ppc64'`, `'s390'`, `'s390x'`, `'x32'`, and `'x64'`.

Equivalente para [`process.arch`][].

## os.constants

<!-- YAML
added: v6.3.0
-->

* {Object}

Retorna un objeto que contiene constantes específicas del sistema operativo comúnmente usadas para códigos de error, señales de proceso, y así sucesivamente. Las constantes específicas actualmente definidas son descritas como [Constantes SO](#os_os_constants_1).

## os.cpus()

<!-- YAML
added: v0.3.3
-->

* Returns: {Object[]}

El método `os.cpus()` retorna un conjunto de objetos conteniendo información acerca de cada core lógico del CPU.

Las propiedades incluidas en cada objeto incluyen:

* `model`{string}
* `velocidad`{number} (en MHz)
* `times` {Object} 
  * `user`{number} El número de milisegundos que el CPU ha estado en modo usuario.
  * `nice` {number} El número de milisegundos que el CPU ha estado en modo nice.
  * `sys` {number} El número de milisegundos que el CPU ha estado en modo sys.
  * `idle` {number} El número de milisegundos que el CPU ha estado en modo idle.
  * `irq` {number} El número de milisegundos que el CPU ha estado en modo irq.

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

Porque los valores `nice` son específicos de UNIX, en Windows los valores `nice` todos los procesos son siempre 0.

## os.endianness()

<!-- YAML
added: v0.9.4
-->

* Retorna: {string}

El método `os.endianness()` retorna una línea identificando la endianidad del CPU* para la cual el binario Node.js fue compilado*.

Posibles valores son:

* `'BE'` para gran Endian
* `'LE'` para pequeño Endian.

## os.freemem()

<!-- YAML
added: v0.3.3
-->

* Retorna: {integer}

El método `os.freemem()` retorna la cantidad de memoria libre del sistema en bytes como un entero.

## os.homedir()

<!-- YAML
added: v2.3.0
-->

* Retorna: {string}

El método `os.homedir()` devuelve el directorio hogar del usuario actual como una línea.

## os.hostname()

<!-- YAML
added: v0.3.3
-->

* Retorna: {string}

El método `os.hostname()` devuelve el nombre del dueño del sistema operativo como una string.

## os.loadavg()

<!-- YAML
added: v0.3.3
-->

* Retorna: {number[]}

El método `os.loadavg()` devuelve un conjunto conteniendo los promedios de carga de 1,5 y 15 minutos.

El promedio de carga es una medida de la actividad del sistema, calculado por el sistema operativo y expresado como un número fraccionario. Como regla general, la carga promedio debería ser idealmente menor que el número de CPUs lógicos en el sistema.

La carga promedio es un concepto específico de UNIX con ningún equivalente real en las plataformas Windows. En Windows, el valor de retorno siempre es `[0, 0, 0]`.

## os.networkInterfaces()

<!-- YAML
added: v0.6.0
-->

* Retorna: {Object}

El método `os.networkInterfaces()` retorna un objeto conteniendo solamente interfaces en la red que han sido asignados a la dirección de la red.

Cada tecla en el objeto retornado identifica la interfaz de red. El valor asociado es un conjunto de objetos donde cada uno describe una dirección asignada a la red.

Las propiedades disponibles en el objeto de dirección de red asignado incluyen:

* `address` {string} La dirección IPv4 o IPv6 asignada
* `netmask` {string} La máscara de red IPv4 o IPv6
* `family` {string} Para `IPv4` o `IPv6`
* `mac` {string} La dirección MAC de la interfaz de red
* `internal` {boolean} `true` si la interfaz de la red es un loopback o es una interfaz similar que no es remotamente accesible; de otra forma `false`
* `scopeid` {number} El identificador numérico IPv6 (solamente especificado cuando `family` es `IPv6`)
* `cidr` {string} The assigned IPv4 or IPv6 address with the routing prefix in CIDR notation. Si la `netmask` es invalida, esta propiedad está colocada a `null`.

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

* Retorna: {string}

El método `os.platform()` retorna un string identificando la plataforma del sistema operativo como fue colocada durante el tiempo de compilación de Node.js.

Los posibles valores actuales son:

* `'aix'`
* `'darwin'`
* `'freebsd'`
* `'linux'`
* `'openbsd'`
* `'sunos'`
* `'win32'`

Equivalente para [`process.platform`][].

El valor `'android'` puede también ser devuelto si Node.js está construido en un sistema operativo Android. Sin embargo, el soporte de Android es considerado[ experimental](https://github.com/nodejs/node/blob/master/BUILDING.md#androidandroid-based-devices-eg-firefox-os) actualmente.

## os.release()

<!-- YAML
added: v0.3.3
-->

* Retorna: {string}

El método `os.release()` retorna un string identificando la versión del sistema operativo.

En sistemas POSIX, la versión del sistema operativo es determinada llamando [uname(3)](https://linux.die.net/man/3/uname). En Windows, `GetVersionExW()` es usado. Por favor vea https://en.wikipedia.org/wiki/Uname#Examples para mayor información.

## os.tmpdir()

<!-- YAML
added: v0.9.9
changes:

  - version: v2.0.0
    pr-url: https://github.com/nodejs/node/pull/747
    description: This function is now cross-platform consistent and no longer
                 returns a path with a trailing slash on any platform
-->

* Retorna: {string}

El método `os.tmpdir()` retorna una string que especifica el directorio predeterminado del sistema operativo para los archivos temporales.

## os.totalmem()

<!-- YAML
added: v0.3.3
-->

* Retorna: {integer}

El método `os.totalmem()` retorna la cantidad total de memoria del sistema en bytes como un entero.

## os.type()

<!-- YAML
added: v0.3.3
-->

* Retorna: {string}

El método `os.type()` retorna una string identificando el nombre del sistema operativo como es retornado por [uname(3)](https://linux.die.net/man/3/uname). Por ejemplo`'Linux'` en Linux, `'Darwin'` en macOS y `'Windows_NT'` en Windows.

Por favor vea https://en.wikipedia.org/wiki/Uname#Examples para información adicional sobre los resultados de correr [uname(3)](https://linux.die.net/man/3/uname) en distintos sistemas operativos.

## os.uptime()

<!-- YAML
added: v0.3.3
changes:

  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/20129
    description: The result of this function no longer contains a fraction
                 component on Windows.
-->

* Retorna: {integer}

El método de `os.uptime()` retorna el tiempo de operación del sistema en segundos.

## os.userInfo([options])

<!-- YAML
added: v6.0.0
-->

* `opciones` {Object} 
  * `encoding` {string} Cifrado de caracteres usado para interpretar strings resultantes. Si `encoding` esta colocado en `'buffer'`, el `username`, `shell`, y `homedir` valores serán instancias `Buffer`. **Predeterminado:** `'utf8'`.
* Retorna: {Object}

El método `os.userInfo()` retorna información acerca del actual usuario efectivo -- en plataformas POSIX, esto es tipicamente un subconjunto en el archivo de contraseñas. El objeto devuelto incluye el `username`, `uid`, `gid`, `shell`, y `homedir`. En Windows, el `uid` y `gid` campos son `-1`, y `shell` es `null`.

El valor del `homedir` deveulto por `os.userInfo()` es provisto por el sistema operativo. Esto difiere del resultado de `os.homedir()` el cual consulta distintas variables del entorno para el directorio hogar antes de recurrir a la respuesta del sistema operativo.

## Constantes del SO

Las siguientes constantes son exportadas por `os.constants`.

No todas las constantes estarán disponibles en todos los sistemas operativos.

### Constantes Señal

<!-- YAML
changes:

  - version: v5.11.0
    pr-url: https://github.com/nodejs/node/pull/6093
    description: Added support for `SIGINFO`.
-->

Las siguientes constantes son exportadas por `os.constants.signals`:

<table>
  <tr>
    <th>Constante</th>
    <th>Descripción</th>
  </tr>
  <tr>
    <td><code>SIGHUP</code></td>
    <td>Enviada para indicar cuando una terminal de control es cerrada o un proceso primario cierra.</td>
  </tr>
  <tr>
    <td><code>SIGINT</code></td>
    <td>Enviada para indicar cuando un usuario desea interrumpir un proceso (`(Ctrl+C)`).</td>
  </tr>
  <tr>
    <td><code>SIGQUIT</code></td>
    <td>Enviada para indicar cuando un usuario desea terminar un proceso y realizar un volcado de memoria.</td>
  </tr>
  <tr>
    <td><code>SIGILL</code></td>
    <td>Enviada a un proceso para notificar que ha intentado realizar una instrucción ilegal, malformada, desconocida, o priviligeada.</td>
  </tr>
  <tr>
    <td><code>SIGTRAP</code></td>
    <td>Enviada a un proceso cuando ha ocurrido una excepción.</td>
  </tr>
  <tr>
    <td><code>SIGABRT</code></td>
    <td>Enviada a un proceso para solicitar que aborte.</td>
  </tr>
  <tr>
    <td><code>SIGIOT</code></td>
    <td>Sinónimo para <code>SIGABRT</code></td>
  </tr>
  <tr>
    <td><code>SIGBUS</code></td>
    <td>Enviada a un proceso para notificar que ha causada un error de bus.</td>
  </tr>
  <tr>
    <td><code>SIGFPE</code></td>
    <td>Enviada a un proceso para notificar que ha realizado una operación aritmética ilegal.</td>
  </tr>
  <tr>
    <td><code>SIGKILL</code></td>
    <td>Enviada a un proceso para terminarlo inmediatamente.</td>
  </tr>
  <tr>
    <td><code>SIGUSR1</code> <code>SIGUSR2</code></td>
    <td>Enviada a un proceso para identificar las condiciones definidas por los usuarios.</td>
  </tr>
  <tr>
    <td><code>SIGSEGV</code></td>
    <td>Enviada a un proceso para notificar de un fallo de segmentación.</td>
  </tr>
  <tr>
    <td><code>SIGPIPE</code></td>
    <td>Enviada a un proceso cuando ha intentado escribir a un pipe desconectado.</td>
  </tr>
  <tr>
    <td><code>SIGALRM</code></td>
    <td>Enviada a un proceso cuando ha transcurrido un temporizador del sistema.</td>
  </tr>
  <tr>
    <td><code>SIGTERM</code></td>
    <td>Enviada a un proceso para solicitar su terminación.</td>
  </tr>
  <tr>
    <td><code>SIGCHLD</code></td>
    <td>Enviada a un proceso cuando un proceso secundario termina.</td>
  </tr>
  <tr>
    <td><code>SIGSTKFLT</code></td>
    <td>Enviada a un proceso para indicar un error de pila en un coprocesador.</td>
  </tr>
  <tr>
    <td><code>SIGCONT</code></td>
    <td>Enviada para indicar al sistema operativo continuar un proceso pausado.</td>
  </tr>
  <tr>
    <td><code>SIGSTOP</code></td>
    <td>Enviada para indicar al sistema operativo detener un proceso.</td>
  </tr>
  <tr>
    <td><code>SIGTSTP</code></td>
    <td>Enviada a un proceso para solicitar que se detenga.</td>
  </tr>
  <tr>
    <td><code>SIGBREAK</code></td>
    <td>Enviada para indicar cuando un usuario desea interrumpir un proceso.</td>
  </tr>
  <tr>
    <td><code>SIGTTIN</code></td>
    <td>Enviada a un proceso cuando lee desde el TTY en segundo plano.</td>
  </tr>
  <tr>
    <td><code>SIGTTOU</code></td>
    <td>Enviada a un proceso cuando escribe desde el TTY estando en segundo plano.</td>
  </tr>
  <tr>
    <td><code>SIGURG</code></td>
    <td>Enviada a un proceso cuando un socket tiene datos urgentes a leer.</td>
  </tr>
  <tr>
    <td><code>SIGXCPU</code></td>
    <td>Enviada a un proceso cuando ha excedido su límite de uso de la CPU.</td>
  </tr>
  <tr>
    <td><code>SIGXFSZ</code></td>
    <td>Enviada a un proceso cuando el archivo crece más grande que el máximo permitido.</td>
  </tr>
  <tr>
    <td><code>SIGVTALRM</code></td>
    <td>Enviada a un proceso cuando el temporizador virtual ha transcurrido.</td>
  </tr>
  <tr>
    <td><code>SIGPROF</code></td>
    <td>Enviada a un proceso cuando un temporizador del sistema ha transcurrido.</td>
  </tr>
  <tr>
    <td><code>SIGWINCH</code></td>
    <td>Enviada a un proceso cuando la terminal de control ha cambiado su tamaño.</td>
  </tr>
  <tr>
    <td><code>SIGIO</code></td>
    <td>Enviada a un proceso cuando la opción E/A está disponible.</td>
  </tr>
  <tr>
    <td><code>SIGPOLL</code></td>
    <td>Sinónimo para <code>SIGIO</code></td>
  </tr>
  <tr>
    <td><code>SIGLOST</code></td>
    <td>Enviada a un proceso cuando un bloqueo de archivo se ha perdido.</td>
  </tr>
  <tr>
    <td><code>SIGPWR</code></td>
    <td>Enviada a un proceso para notificar de un fallo de alimentación.</td>
  </tr>
  <tr>
    <td><code>SIGINFO</code></td>
    <td>Sinónimo para <code>SIGPWR</code></td>
  </tr>
  <tr>
    <td><code>SIGSYS</code></td>
    <td>Enviada a un proceso para notificar un argumento erróneo.</td>
  </tr>
  <tr>
    <td><code>SIGUNUSED</code></td>
    <td>Sinónimo para <code>SIGSYS</code></td>
  </tr>
</table>

### Constantes de Error

Las siguientes constantes de error son exportadas por `os.constants.errno`:

#### Constantes de Error POSIX

<table>
  <tr>
    <th>Constante</th>
    <th>Descripción</th>
  </tr>
  <tr>
    <td><code>E2BIG</code></td>
    <td>Indica que la lista de argumentos es mas larga de lo esperado.</td>
  </tr>
  <tr>
    <td><code>EACCES</code></td>
    <td>Indica que la operación no tiene los permisos suficientes.</td>
  </tr>
  <tr>
    <td><code>EADDRINUSE</code></td>
    <td>Indica que la dirección de red se encuentra en uso.</td>
  </tr>
  <tr>
    <td><code>EADDRNOTAVAIL</code></td>
    <td>Indica que la dirección de red se encuentra no disponible para su uso.</td>
  </tr>
  <tr>
    <td><code>EAFNOSUPPORT</code></td>
    <td>Indica que la familia de la dirección de red no es compatible.</td>
  </tr>
  <tr>
    <td><code>EAGAIN</code></td>
    <td>Indica que no hay datos disponibles y que se debe intentar la operación más tarde.</td>
  </tr>
  <tr>
    <td><code>EALREADY</code></td>
    <td>Indica que el socket ya tiene una conexión pendiente en proceso.</td>
  </tr>
  <tr>
    <td><code>EBADF</code></td>
    <td>Indica que un descriptor del archivo no es válido.</td>
  </tr>
  <tr>
    <td><code>EBADMSG</code></td>
    <td>Indica un mensaje de datos inválido.</td>
  </tr>
  <tr>
    <td><code>EBUSY</code></td>
    <td>Indica que un dispositivo o recurso está ocupado.</td>
  </tr>
  <tr>
    <td><code>ECANCELED</code></td>
    <td>Indica que una operación fue cancelada.</td>
  </tr>
  <tr>
    <td><code>ECHILD</code></td>
    <td>Indica que no no hay procesos secundarios.</td>
  </tr>
  <tr>
    <td><code>ECONNABORTED</code></td>
    <td>Indica que la conexión de red ha sido abortada.</td>
  </tr>
  <tr>
    <td><code>ECONNREFUSED</code></td>
    <td>Indica que la conexión de red fue rechazada.</td>
  </tr>
  <tr>
    <td><code>ECONNRESET</code></td>
    <td>Indica que la conexión de red fue reiniciada.</td>
  </tr>
  <tr>
    <td><code>EDEADLK</code></td>
    <td>Indica que un bloqueo de recursos ha sido evadido.</td>
  </tr>
  <tr>
    <td><code>EDESTADDRREQ</code></td>
    <td>Indica que una dirección destino es requerida.</td>
  </tr>
  <tr>
    <td><code>EDOM</code></td>
    <td>Indica que un argumento está fuera del dominio de la función.</td>
  </tr>
  <tr>
    <td><code>EDQUOT</code></td>
    <td>Indica que se ha excedido la cuota del disco.</td>
  </tr>
  <tr>
    <td><code>EEXIST</code></td>
    <td>Indica que el archivo existe.</td>
  </tr>
  <tr>
    <td><code>EFAULT</code></td>
    <td>Indica una dirección de puntero no válido.</td>
  </tr>
  <tr>
    <td><code>EFBIG</code></td>
    <td>Indica que el archivo es muy grande.</td>
  </tr>
  <tr>
    <td><code>EHOSTUNREACH</code></td>
    <td>Indica que el host es inalcanzable.</td>
  </tr>
  <tr>
    <td><code>EIDRM</code></td>
    <td>Indica que el identificador ha sido removido.</td>
  </tr>
  <tr>
    <td><code>EILSEQ</code></td>
    <td>Indica una secuencia ilegal de bytes.</td>
  </tr>
  <tr>
    <td><code>EINPROGRESS</code></td>
    <td>Indica que una operación está en progreso.</td>
  </tr>
  <tr>
    <td><code>EINTR</code></td>
    <td>Indica que un llamado a la función fue interrumpido.</td>
  </tr>
  <tr>
    <td><code>EINVAL</code></td>
    <td>Indica que un argumento inválido fue proporcionado.</td>
  </tr>
  <tr>
    <td><code>EIO</code></td>
    <td>Indica un error E/A no especificado.</td>
  </tr>
  <tr>
    <td><code>EISCONN</code></td>
    <td>Indica que el socket está conectado.</td>
  </tr>
  <tr>
    <td><code>EISDIR</code></td>
    <td>Indica que la ruta es un directorio.</td>
  </tr>
  <tr>
    <td><code>ELOOP</code></td>
    <td>Indica muchos niveles de enlaces simbólicos en la ruta.</td>
  </tr>
  <tr>
    <td><code>EMFILE</code></td>
    <td>Indica que hay demasiados archivos abiertos.</td>
  </tr>
  <tr>
    <td><code>EMLINK</code></td>
    <td>Indica que hay muchos enlaces duros a un archivo.</td>
  </tr>
  <tr>
    <td><code>EMSGSIZE</code></td>
    <td>Indica que el mensaje provisto es muy largo.</td>
  </tr>
  <tr>
    <td><code>EMULTIHOP</code></td>
    <td>Indica que un multisalto fue intentado.</td>
  </tr>
  <tr>
    <td><code>ENAMETOOLONG</code></td>
    <td>Indica que el nombre del archivo es muy largo.</td>
  </tr>
  <tr>
    <td><code>ENETDOWN</code></td>
    <td>Indica que la red esta caída.</td>
  </tr>
  <tr>
    <td><code>ENETRESET</code></td>
    <td>Indica que la conexión fue abortada por la red.</td>
  </tr>
  <tr>
    <td><code>ENETUNREACH</code></td>
    <td>Indica que la red es inalcanzable.</td>
  </tr>
  <tr>
    <td><code>ENFILE</code></td>
    <td>Indica que hay demasiados archivos abiertos en el sistema.</td>
  </tr>
  <tr>
    <td><code>ENOBUFS</code></td>
    <td>Indica que no hay espacio de búfer disponible.</td>
  </tr>
  <tr>
    <td><code>ENODATA</code></td>
    <td>Indicates that no message is available on the stream head read
    queue.</td>
  </tr>
  <tr>
    <td><code>ENODEV</code></td>
    <td>Indicates that there is no such device.</td>
  </tr>
  <tr>
    <td><code>ENOENT</code></td>
    <td>Indicates that there is no such file or directory.</td>
  </tr>
  <tr>
    <td><code>ENOEXEC</code></td>
    <td>Indicates an exec format error.</td>
  </tr>
  <tr>
    <td><code>ENOLCK</code></td>
    <td>Indicates that there are no locks available.</td>
  </tr>
  <tr>
    <td><code>ENOLINK</code></td>
    <td>Indications that a link has been severed.</td>
  </tr>
  <tr>
    <td><code>ENOMEM</code></td>
    <td>Indicates that there is not enough space.</td>
  </tr>
  <tr>
    <td><code>ENOMSG</code></td>
    <td>Indicates that there is no message of the desired type.</td>
  </tr>
  <tr>
    <td><code>ENOPROTOOPT</code></td>
    <td>Indicates that a given protocol is not available.</td>
  </tr>
  <tr>
    <td><code>ENOSPC</code></td>
    <td>Indicates that there is no space available on the device.</td>
  </tr>
  <tr>
    <td><code>ENOSR</code></td>
    <td>Indicates that there are no stream resources available.</td>
  </tr>
  <tr>
    <td><code>ENOSTR</code></td>
    <td>Indicates that a given resource is not a stream.</td>
  </tr>
  <tr>
    <td><code>ENOSYS</code></td>
    <td>Indicates that a function has not been implemented.</td>
  </tr>
  <tr>
    <td><code>ENOTCONN</code></td>
    <td>Indicates that the socket is not connected.</td>
  </tr>
  <tr>
    <td><code>ENOTDIR</code></td>
    <td>Indicates that the path is not a directory.</td>
  </tr>
  <tr>
    <td><code>ENOTEMPTY</code></td>
    <td>Indicates that the directory is not empty.</td>
  </tr>
  <tr>
    <td><code>ENOTSOCK</code></td>
    <td>Indicates that the given item is not a socket.</td>
  </tr>
  <tr>
    <td><code>ENOTSUP</code></td>
    <td>Indicates that a given operation is not supported.</td>
  </tr>
  <tr>
    <td><code>ENOTTY</code></td>
    <td>Indicates an inappropriate I/O control operation.</td>
  </tr>
  <tr>
    <td><code>ENXIO</code></td>
    <td>Indicates no such device or address.</td>
  </tr>
  <tr>
    <td><code>EOPNOTSUPP</code></td>
    <td>Indicates that an operation is not supported on the socket.
    Note that while `ENOTSUP` and `EOPNOTSUPP` have the same value on Linux,
    according to POSIX.1 these error values should be distinct.)</td>
  </tr>
  <tr>
    <td><code>EOVERFLOW</code></td>
    <td>Indicates that a value is too large to be stored in a given data
    type.</td>
  </tr>
  <tr>
    <td><code>EPERM</code></td>
    <td>Indicates that the operation is not permitted.</td>
  </tr>
  <tr>
    <td><code>EPIPE</code></td>
    <td>Indicates a broken pipe.</td>
  </tr>
  <tr>
    <td><code>EPROTO</code></td>
    <td>Indicates a protocol error.</td>
  </tr>
  <tr>
    <td><code>EPROTONOSUPPORT</code></td>
    <td>Indicates that a protocol is not supported.</td>
  </tr>
  <tr>
    <td><code>EPROTOTYPE</code></td>
    <td>Indicates the wrong type of protocol for a socket.</td>
  </tr>
  <tr>
    <td><code>ERANGE</code></td>
    <td>Indicates that the results are too large.</td>
  </tr>
  <tr>
    <td><code>EROFS</code></td>
    <td>Indicates that the file system is read only.</td>
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
