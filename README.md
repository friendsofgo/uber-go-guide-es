<!--

Como editar este documento:

- Comenta todos los cambios primero creando una issue de Github.
- Actualice la tabla de contenido a medida que se agreguen o eliminen nuevas secciones.
- Usa tablas, una al lado de la otra para ejemplos de código. Como se explica a continuación.

Ejemplos de código:

Usar 2 espacios para indentar. Debes mantener un estado horizontal en las tablas una al lado de la otra.

Para el código de ejemplo, usar tablas una al lado de la otra, siguiendo el siguiente snippet.

~~~
<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
CÓDIGO INCORRECTO AQUÍ
```

</td><td>

```go
CÓDIGO CORRECTO AQUÍ
```

</td></tr>
</tbody></table>
~~~

(Necesitas dejar líneas vacías entre los <td> y los ejemplos de códigos para que Markdown pueda interpretarlo correctamente.)

Si necesitas añadir etiquetas o descripciones debajo de los ejemplos de código, debes añadir otra fila antes de la línea </tbody></table>.

~~~
<tr>
<td>DESCRIPCIÓN DE CÓDIGO INCORRECTO</td>
<td>DESCRIPCIÖN DE CÓDIGO CORRECTO</td>
</tr>
~~~

-->

# Guía de estilo de Go en Uber

Esta guía es la versión traducida al español de la versión original creada por [Uber](https://github.com/uber-go/guide). Intentaremos mantener esta versión actualizada casi al mismo tiempo que la original. Si queréis contribuir podéis dejar vuestros PRs, pero recordar que siempre tendrán que ser sobre correcciones sobre la traducción o añadir nuevos cambios que no hayamos actualizados.

Además iremos dejando todos los cambios que se vayan realizando en el fichero [CHANGELOG.md](https://github.com/friendsofgo/uber-go-guide-es/blob/master/CHANGELOG.md)

## Índice

- [Introducción](#introduccin)
- [Directrices](#directrices)
  - [Punteros a interfaces](#punteros-a-interfaces)
  - [Receptores e interfaces](#receptores-e-interfaces)
  - [No es necesario inicializar los Mutexs](#no-es-necesario-inicializar-los-mutexs)
  - [Limitaciones copiando Slices y Maps](#limitaciones-copiando-slices-y-maps)
  - [Defer para limpiar](#defer-para-limpiar)
  - [El tamaño de los Canales es Uno o Ninguno](#el-tamao-de-los-canales-es-uno-o-ninguno)
  - [Empezando Enums en Uno](#empezando-enums-en-uno)
  - [Error Types](#error-types)
  - [Error Wrapping](#error-wrapping)
  - [Manejando los errores de Type Assertion](#manejando-los-errores-de-type-assertion)
  - [No Panic](#no-panic)
  - [Usa go.uber.org/atomic](#usa-gouberorgatomic)
  - [Evite globales mutables](#evite-globales-mutables)
- [Rendimiento](#rendimiento)
  - [Usar strconv en lugar fmt](#usar-strconv-en-lugar-fmt)
  - [Evitar convertir strings a bytes](#evitar-convertir-strings-a-bytes)
  - [Especificar una capacidad aproximada al Map](#especificar-una-capacidad-aproximada-al-map)
- [Estilo](#estilo)
  - [Se consistente](#se-consistente)
  - [Agrupar declaraciones similares](#agrupar-declaraciones-similares)
  - [Agrupaciones y orden en los imports](#agrupaciones-y-orden-en-los-imports)
  - [Nombre de paquetes](#nombre-de-paquetes)
  - [Nombres de funciones](#nombres-de-funciones)
  - [Alias en los imports](#alias-en-los-imports)
  - [Agrupación de funciones y orden](#agrupacin-de-funciones-y-orden)
  - [Reducir el código anidado](#reducir-el-cdigo-anidado)
  - [Else innecesario](#else-innecesario)
  - [Declaración de variables globales](#declaracin-de-variables-globales)
  - [Prefijo _ para las globales privadas](#prefijo-_-para-las-globales-privadas)
  - [Composición en Structs](#composicin-en-structs)
  - [Usa el nombre de los campos al inicializar Structs](#usa-el-nombre-de-los-campos-al-inicializar-structs)
  - [Declaración de variables locales](#declaracin-de-variables-globales)
  - [nil es un slice válido](#nil-es-un-slice-vlido)
  - [Reducir el scope de las variables](#reducir-el-scope-de-las-variables)
  - [Evitar parámetros planos](#evitar-parmetros-planos)
  - [Evita declarar strings con escapados](#evita-declarar-strings-con-escapados)
  - [Inicializando referencias a Struct](#inicializando-referencias-a-struct)
  - [Inicializando Maps](#inicializando-maps)
  - [Formatos de Strings fuera del Printf](#formatos-de-strings-fuera-del-printf)
  - [Nombra funciones al estilo Printf](#nombra-funciones-al-estilo-printf)
- [Patrones](#patrones)
  - [Test Tables](#test-tables)
  - [Opciones funcionales](#opciones-funcionales)

## Introducción

Los estilos son las convenciones que gobiernan nuestro código. El término estilo aquí es un poco inapropiado, ya que estas convenciones cubren algo más que el formato de nuestros ficheros, del cual ya se encarga sobradamente `gofmt`.

La finalidad de esta guía es la de tratar de describir en detalle como se escribe código Go en Uber. Estas reglas han sido creadas para mantener un código manejable, haciendo que los desarrolladores puedan realizar nuevas tareas o funcionalidades de manera más productiva.

Esta guía fue creada originalmente por [Prashant Varanasi] y [Simon Newton] como una forma de poner al día a algunos compañeros en Go. Con los años se ha ido mejorando con el feedback que ha ido recibiendo.

  [Prashant Varanasi]: https://github.com/prashantv
  [Simon Newton]: https://github.com/nomis52

Este documento cubre las convenciones idiomáticas en Go que se siguen en Uber. Muchas de éstas son pautas generales para Go, mientras que otras se extienden a recursos externos:

1. [Effective Go](https://golang.org/doc/effective_go.html)
2. [The Go common mistakes guide](https://github.com/golang/go/wiki/CodeReviewComments)

Todo el código debe estar libre de errores cuando se ejecute tanto `golint` como `go vet`. Recomendamos configurar tu editor para ejecutar:

- `goimports` al guardar
- `golint` y `go vet` para comprobar errores.

Puedes encontrar más información, en la sección de soporte de editores aquí: <https://github.com/golang/go/wiki/IDEsAndTextEditorPlugins>

## Directrices

### Punteros a interfaces

Casi nunca necesitarás un puntero a una interfaz. Deberías pasar las interfaces como valor, el dato subyacente puede seguir siendo un puntero.

Una interfaz se compone de dos campos:

1. Un puntero a información específica del tipo. Puedes pensar en ello como un `type`.
2. Puntero a datos. Si el dato almacenado es un puntero, será almacenado directamente. Si el dato almacenado es un valor, entonces se almacenará un puntero a su valor.

Si quieres que tu interfaz contenga métodos que puedan modificar los datos subyacentes, tendrás que utilizar un puntero.

### Receptores e interfaces

Métodos que reciben un receptor por valor, pueden ser llamados tanto como punteros como por valor.

Por ejemplo,

```go
type S struct {
  data string
}

func (s S) Read() string {
  return s.data
}

func (s *S) Write(str string) {
  s.data = str
}

sVals := map[int]S{1: {"A"}}

// Puedes llamar a Read usando un valor
sVals[1].Read()

// Producirá un error de compilación:
//  sVals[1].Write("test")

sPtrs := map[int]*S{1: {"A"}}

// Puedes llamar a ambos Read y Write usando un puntero
sPtrs[1].Read()
sPtrs[1].Write("test")
```

Del mismo modo, un puntero puede satisfacer una interfaz, incluso si el método tiene un receptor por valor.

```go
type F interface {
  f()
}

type S1 struct{}

func (s S1) f() {}

type S2 struct{}

func (s *S2) f() {}

s1Val := S1{}
s1Ptr := &S1{}
s2Val := S2{}
s2Ptr := &S2{}

var i F
i = s1Val
i = s1Ptr
i = s2Ptr

// El siguiente código no compilará, ya que s2Val es un valor, y no hay método f() que esperen ser llamados por un valor.
//   i = s2Val
```

Effective Go tiene muy buena documentación sobre [Punteros vs. Valor].

  [Punteros vs. Valor]: https://golang.org/doc/effective_go.html#pointers_vs_values

### No es necesario inicializar los Mutexs

El valor por defecto de `sync.Mutex` y `sync.RWMutex` es suficiente, no necesitarás en la mayoría de los casos crear un puntero hacia un `mutex`.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
mu := new(sync.Mutex)
mu.Lock()
```

</td><td>

```go
var mu sync.Mutex
mu.Lock()
```

</td></tr>
</tbody></table>

Si usas un `struct` como puntero, entonces el `mutex` no necesita ser un puntero.

Los `structs` privados que usen `mutex` para proteger campos del `struct` deberían estar compuestos por `mutex`.

<table>
<tbody>
<tr><td>

```go
type smap struct {
  sync.Mutex // sólo para tipos privados

  data map[string]string
}

func newSMap() *smap {
  return &smap{
    data: make(map[string]string),
  }
}

func (m *smap) Get(k string) string {
  m.Lock()
  defer m.Unlock()

  return m.data[k]
}
```

</td><td>

```go
type SMap struct {
  mu sync.Mutex

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.mu.Lock()
  defer m.mu.Unlock()

  return m.data[k]
}
```

</td></tr>

</tr>
<tr>
<td>Embed mutex para tipos privados o tipos que necesitan implementar la interfaz Mutex.</td>
<td>Para tipos públicos, usa un campo privado.</td>
</tr>

</tbody></table>

### Limitaciones copiando Slices y Maps

Slices y maps contienen punteros que apuntan a datos subyacentes, así que hay tener cuidado cuando tengamos que copiar sus datos.

#### Recibiendo Slices y Maps

Ten presente que un usuario puede modificar un `map` o un `slice` que recibas como argumento si guardas su referencia.

<table>
<thead><tr><th>Incorrecto</th> <th>Correcto</th></tr></thead>
<tbody>
<tr>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = trips
}

trips := ...
d1.SetTrips(trips)

// ¿Querías modificar d1.trips?
trips[0] = ...
```

</td>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = make([]Trip, len(trips))
  copy(d.trips, trips)
}

trips := ...
d1.SetTrips(trips)

// Ahora podemos modificar trips[0] sin que d1.trips se vea afectado.
trips[0] = ...
```

</td>
</tr>

</tbody>
</table>

#### Devolviendo Slices y Maps

Del mismo modo, ten cuidado con las modificaciones que se realicen a los `maps` o `slice` que exponen su estado interno.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

// Snapshot devuelve el valor actual de s.counters.
func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  return s.counters
}

// snapshot ya no está protegido por el mutex, por eso
// cualquier acceso al snapshot está sujeto a condiciones de carrera.
snapshot := stats.Snapshot()
```

</td><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  result := make(map[string]int, len(s.counters))
  for k, v := range s.counters {
    result[k] = v
  }
  return result
}

// Snapshot es ahora una copia.
snapshot := stats.Snapshot()
```

</td></tr>
</tbody></table>

### Defer para limpiar

Utiliza `defer` para limpiar y cerrar recursos como ficheros y `locks`.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
p.Lock()
if p.count < 10 {
  p.Unlock()
  return p.count
}

p.count++
newCount := p.count
p.Unlock()

return newCount

// es sencillo olvidar realizar unlocks cuando tienes múltiples returns
```

</td><td>

```go
p.Lock()
defer p.Unlock()

if p.count < 10 {
  return p.count
}

p.count++
return p.count

// más legible
```

</td></tr>
</tbody></table>

El sobrecoste que tiene `defer` es extremadamente pequeño y sólo debe evitarse si puedes
asegurar que asegurar que el tiempo de ejecución de tu función es de nanosegundos. Se gana mucho más con la legibilidad obtenida utilizando `defer` que el minúsculo coste que tiene utilizarlos. Esto se puede observar claramente en métodos largos que tienen más que simples accesos a memoria, dónde los otros cálculos son mucho más significantes que el `defer`.

### El tamaño de los Canales es Uno o Ninguno

Los canales normalmente deberían ser de tamaño uno o ser `unbuffered`. Por defecto, los canales son
`unbuffered` y tienen un tamaño de cero. Cualquier otro tamaño debería de ser analizado con mucho detalle. Consider

Channels should usually have a size of one or be unbuffered. By default,
channels are unbuffered and have a size of zero. Any other size
must be subject to a high level of scrutiny. Considerar el tamaño de nuestro canal evitará que acabemos sufriendo problemas de carga y bloqueos.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
// ¡Esta inicialización debería ser suficiente para cualquiera!
c := make(chan int, 64)
```

</td><td>

```go
// Tamaño de uno
c := make(chan int, 1) // o
// Canal unbuffered, tamaño de cero
c := make(chan int)
```

</td></tr>
</tbody></table>

### Empezando Enums en Uno

La manera estándar de introducir `enums` en Go es declarar un tipo personalizado y un grupo de `const` empezando con el valor `iota`. Como las variables tienen un valor predeterminado de 0, deberías de empezar tus `enums` por un valor que sea distinto de 0.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota
  Subtract
  Multiply
)

// Add=0, Subtract=1, Multiply=2
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

// Add=1, Subtract=2, Multiply=3
```

</td></tr>
</tbody></table>

Hay casos donde usar el valor cero tiene sentido, como por ejemplo cuando el valor cero es el comportamiento deseado.

```go
type LogOutput int

const (
  LogToStdout LogOutput = iota
  LogToFile
  LogToRemote
)

// LogToStdout=0, LogToFile=1, LogToRemote=2
```

<!-- TODO: section on String methods for enums -->

### Error Types

Hay varias formas de declarar errores en Go:

- [`errors.New`] para errores con un simple `string` estático
- [`fmt.Errorf`] para errores con un `string` formateado
- Tipos personalizados que implementan la interfaz `error` es decir tienen el método `Error()`
- Errores envueltos (aka "wrapeados") usando [`"pkg/errors".Wrap`]

A la hora de devolver errores, hay que considerar ciertos aspecto para tomar la mejor decisión:

- ¿Es un simple error que no necesita información adicional? Entonces, [`errors.New`] debería ser suficiente.
- ¿Los clientes necesitan detectar y manejar este error? Entonces, deberías de utilizar un tipo personalizado que implemente la interfaz `error`.
- ¿Estás propagando un error que te ha devuelto otra función?Are you propagating an error returned by a downstream function? Entonces, salta a la sección de [error wrapping](#error-wrapping).
- En otros caso utilizaremos, [`fmt.Errorf`].

  [`errors.New`]: https://golang.org/pkg/errors/#New
  [`fmt.Errorf`]: https://golang.org/pkg/fmt/#Errorf
  [`"pkg/errors".Wrap`]: https://godoc.org/github.com/pkg/errors#Wrap

Si el cliente necesita detectar el error pero no necesita información adicional, utiliza errores sentinelas o `sentinel errors` en lugar de devolver la función con [`errors.New`], como se puede ver a continuación:

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open() error {
  return errors.New("could not open")
}

// package bar

func use() {
  if err := foo.Open(); err != nil {
    if err.Error() == "could not open" {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td><td>

```go
// package foo

var ErrCouldNotOpen = errors.New("could not open")

func Open() error {
  return ErrCouldNotOpen
}

// package bar

if err := foo.Open(); err != nil {
  if err == foo.ErrCouldNotOpen {
    // handle
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

Si tienes un error que los clientes necesitan detectar, y además quieres proporcionar más información, entonces deberías utilizar un tipo personalizado.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func open(file string) error {
  return fmt.Errorf("file %q not found", file)
}

func use() {
  if err := open(); err != nil {
    if strings.Contains(err.Error(), "not found") {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td><td>

```go
type errNotFound struct {
  file string
}

func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func open(file string) error {
  return errNotFound{file: file}
}

func use() {
  if err := open(); err != nil {
    if _, ok := err.(errNotFound); ok {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td></tr>
</tbody></table>

Ten cuidado con el scope de tus errores de tipos personalizados, ya que si los haces públicos pasaran a ser parte de la API pública del paquete. Es preferible solo exponer una función que compruebe si el error es del tipo deseado, en lugar de exponer tu `struct`.

```go
// package foo

type errNotFound struct {
  file string
}

func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func IsNotFoundError(err error) bool {
  _, ok := err.(errNotFound)
  return ok
}

func Open(file string) error {
  return errNotFound{file: file}
}

// package bar

if err := foo.Open("foo"); err != nil {
  if foo.IsNotFoundError(err) {
    // handle
  } else {
    panic("unknown error")
  }
}
```

<!-- TODO: Exposing the information to callers with accessor functions. -->

### Error Wrapping

Hay tres formas para propagar los errores si una llamada falla:

- Devolver el error original si no necesitas añadir un contexto adicional y quieres mantener el tipo del error original.
- Añadir contexto usando [`"pkg/errors".Wrap`] para que el mensaje de error proporcione más contexto y [`"pkg/errors".Cause`] pueda ser usado para extraer el error original.
- Usa [`fmt.Errorf`] si los métodos/funciones no necesitan detectar o manejar este caso específico de error.

Se recomienda añadir contexto donde sea posible para que en lugar de tener errores como "connection refused", tengamos algo más útil como "call service foo: connection refused".

Cuando añadimos contexto a los errores devueltos, intentemos mantener el contexto conciso evitando frases como "failed to", que indican lo obvio y se nos acumularán en la pila de mensajes:

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "failed to create new store: %s", err)
}
```

</td><td>

```go
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "new store: %s", err)
}
```

<tr><td>

```
failed to x: failed to y: failed to create new store: the error
```

</td><td>

```
x: y: new store: the error
```

</td></tr>
</tbody></table>

Sin embargo es enviado a otro sistema, deberá de dejar claro que el mensaje es un error (ej. un tag `err` o un prefijo como "Failed" en los logs).

Un artículo muy interesante sobre el tratamiento de errores en Go: [Don't just check errors, handle them gracefully].

  [`"pkg/errors".Cause`]: https://godoc.org/github.com/pkg/errors#Cause
  [Don't just check errors, handle them gracefully]: https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully

### Manejando los errores de Type Assertion

*Para claridad de esta parte mantendremos el término original en inglés, `Type Assertion` en lugar de validación de tipos, ya que está más extendido.*

Devolver en una sola linea un [type assertion] provocará `panic` en tipos incorrectos. Por lo tanto, siempre usaremos la anotación "comma ok".

  [type assertion]: https://golang.org/ref/spec#Type_assertions

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
t := i.(string)
```

</td><td>

```go
t, ok := i.(string)
if !ok {
  // handle the error gracefully
}
```

</td></tr>
</tbody></table>

<!-- TODO: There are a few situations where the single assignment form is
fine. -->

### No Panic

El código que funciona en producción no puede provocar `panics`. Los `panics` son la mayor fuente de [fallos en cascada]. Si un ocurre un error, la función tiene que devolver un error y permitir al que esta usando dicha función como manejarlo.

  [fallos en cascada]: https://en.wikipedia.org/wiki/Cascading_failure

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
func foo(bar string) {
  if len(bar) == 0 {
    panic("bar must not be empty")
  }
  // ...
}

func main() {
  if len(os.Args) != 2 {
    fmt.Println("USAGE: foo <bar>")
    os.Exit(1)
  }
  foo(os.Args[1])
}
```

</td><td>

```go
func foo(bar string) error {
  if len(bar) == 0 {
    return errors.New("bar must not be empty")
  }
  // ...
  return nil
}

func main() {
  if len(os.Args) != 2 {
    fmt.Println("USAGE: foo <bar>")
    os.Exit(1)
  }
  if err := foo(os.Args[1]); err != nil {
    panic(err)
  }
}
```

</td></tr>
</tbody></table>

Panic/recover no es una estrategia para controlar errores. Un programa tiene que devolver panic solo cuando sucede algo de lo que no puede recuperarse, como puede ser acceder aun método de un valor `nil`. La excepción a esta regla sólo se aplica cuando inicializamos el programa: aquello que haga que nuestro programa no pueda funcionar correctamente en el momento de inicialización deberá abortar la ejecución debido a un `panic`.

```go
var _statusTemplate = template.Must(template.New("name").Parse("_statusHTML"))
```
Incluso en los tests, es preferible usar `t.Fatal` o `t.FailNow` en lugar de `panics` para marcar el test como fallido.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestFoo(t *testing.T)

f, err := ioutil.TempFile("", "test")
if err != nil {
  panic("failed to set up test")
}
```

</td><td>

```go
// func TestFoo(t *testing.T)

f, err := ioutil.TempFile("", "test")
if err != nil {
  t.Fatal("failed to set up test")
}
```

</td></tr>
</tbody></table>

<!-- TODO: Explain how to use _test packages. -->

### Usa go.uber.org/atomic

Las operaciones atómicas con el paquete [sync/atomic] operan en los tipos sin procesar
(`int32`, `int64`, etc.) por lo que es fácil olvidar usar operaciones atómicas para leer o modificar las variables.

[go.uber.org/atomic] añade seguridad de tipo a estas operaciones ocultando el tipo subyacente. Además, incluye un tipo conveniente, el `atomic.Bool`.

  [go.uber.org/atomic]: https://godoc.org/go.uber.org/atomic
  [sync/atomic]: https://golang.org/pkg/sync/atomic/

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
type foo struct {
  running int32  // atomic
}

func (f* foo) start() {
  if atomic.SwapInt32(&f.running, 1) == 1 {
     // already running…
     return
  }
  // start the Foo
}

func (f *foo) isRunning() bool {
  return f.running == 1  // race!
}
```

</td><td>

```go
type foo struct {
  running atomic.Bool
}

func (f *foo) start() {
  if f.running.Swap(true) {
     // already running…
     return
  }
  // start the Foo
}

func (f *foo) isRunning() bool {
  return f.running.Load()
}
```

</td></tr>
</tbody></table>

### Evite globales mutables

Evite la mutación de variables globales, en su lugar opte por la inyección de dependencias.
Esto se aplica tanto a los punteros de función como a otros tipos de valores.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// sign.go
var _timeNow = time.Now
func sign(msg string) string {
  now := _timeNow()
  return signWithTime(msg, now)
}
```

</td><td>

```go
// sign.go
type signer struct {
  now func() time.Time
}
func newSigner() *signer {
  return &signer{
    now: time.Now,
  }
}
func (s *signer) Sign(msg string) string {
  now := s.now()
  return signWithTime(msg, now)
}
```

</td></tr>
<tr><td>

```go
// sign_test.go
func TestSign(t *testing.T) {
  oldTimeNow := _timeNow
  _timeNow = func() time.Time {
    return someFixedTime
  }
  defer func() { _timeNow = oldTimeNow }()
  assert.Equal(t, want, sign(give))
}
```

</td><td>

```go
// sign_test.go
func TestSigner(t *testing.T) {
  s := newSigner()
  s.now = func() time.Time {
    return someFixedTime
  }
  assert.Equal(t, want, s.Sign(give))
}
```

</td></tr>
</tbody></table>

## Rendimiento

Aqui se recogen las directrices específicas de rendimiento aplicadas 
a como realizar ciertas acciones o utilizar ciertas funciones.

### Usar strconv en lugar fmt

Cuando convertimos primitivos a/o strings, `strconv` es más rápido que `fmt`.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
for i := 0; i < b.N; i++ {
  s := fmt.Sprint(rand.Int())
}
```

</td><td>

```go
for i := 0; i < b.N; i++ {
  s := strconv.Itoa(rand.Int())
}
```

</td></tr>
<tr><td>

```
BenchmarkFmtSprint-4    143 ns/op    2 allocs/op
```

</td><td>

```
BenchmarkStrconv-4    64.2 ns/op    1 allocs/op
```

</td></tr>
</tbody></table>

### Evitar convertir strings a bytes

No se recomienda crear `slice` de `byte` a partir de un string de manera repetitiva. En su lugar, 
realiza la conversión una vez y captura el resultado.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
for i := 0; i < b.N; i++ {
  w.Write([]byte("Hello world"))
}
```

</td><td>

```go
data := []byte("Hello world")
for i := 0; i < b.N; i++ {
  w.Write(data)
}
```

</tr>
<tr><td>

```
BenchmarkBad-4   50000000   22.2 ns/op
```

</td><td>

```
BenchmarkGood-4  500000000   3.25 ns/op
```

</td></tr>
</tbody></table>

### Especificar una capacidad aproximada al Map

Cuando sea posible, inicializacermos el `map` indicando una capacidad aproximada, utilizando la función `make()`.

```go
make(map[T1]T2, hint)
```

Intenta siempre que puedas, proporcionar una capacidad aproximada utilizando `make()`, 
ajustando su tamaño a la hora de ser inicializado, esto reduce la necesidad de aumentar 
el tamaño y las asignaciones del `map` a medida que se vayan agregando nuevos elementos. 
Ten en cuenta que la capacidad aproximada no impedirá en ningún caso que se puedan asignar 
elementos de más aunque ésta haya sido proporcionada.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
m := make(map[string]os.FileInfo)

files, _ := ioutil.ReadDir("./files")
for _, f := range files {
    m[f.Name()] = f
}
```

</td><td>

```go

files, _ := ioutil.ReadDir("./files")

m := make(map[string]os.FileInfo, len(files))
for _, f := range files {
    m[f.Name()] = f
}
```

</td></tr>
<tr><td>

`m` es creado sin asignar una capacidad previa; 
esto provocará más asignaciones en el momento de añadir nuevos elementos.

</td><td>

`m` es creado utilizando una capacidad previa; habrá menos asignaciones en el momento de añadir
nuevos elementos.

</td></tr>
</tbody></table>

## Estilo

### Se consistente

Alguna de las directrices recogidas en este documente pueden ser tratadas de manera objetiva; sin embargo otras son 
situacionales, contextuales o subjetivas.

Por encima de todo, **se consistente**.

El código consistente es sencillo de mantener, sencillo de comprender, 
requiere menos esfuerzo cognitivo y es más sencillo de migrar o actualizar si aparecen nuevas 
convenciones o funcionalidades o bugs que solucionar.

Por el contrario, tener estilos dispares o conflictivos entre si en un solo repositorio de código, causará sin duda 
alguna un sobrecoste de mantenimiento, que afectará de manera significativa a la velocidad, code reviews y creación de 
nuevas funcionalidades y corrección de errores.

Cuando apliques estas directrices en tu código, es recomendable que los cambios sean hechos a nivel de paquete 
(o en todo el código), de otro modo la aplicación a nivel de sub paquetes violará la preocupación anterior al 
introducir múltiples estilos en el mismo código.

### Agrupar declaraciones similares

Go soporta la agrupación de declaraciones similares.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
import "a"
import "b"
```

</td><td>

```go
import (
  "a"
  "b"
)
```

</td></tr>
</tbody></table>

Esto también aplica a variables, constantes y declaraciones de tipo.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go

const a = 1
const b = 2



var a = 1
var b = 2



type Area float64
type Volume float64
```

</td><td>

```go
const (
  a = 1
  b = 2
)

var (
  a = 1
  b = 2
)

type (
  Area float64
  Volume float64
)
```

</td></tr>
</tbody></table>

Sólo agruparemos aquellas declaraciones que estén estrechamente cohesionadas.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
  ENV_VAR = "MY_ENV"
)
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

const ENV_VAR = "MY_ENV"
```

</td></tr>
</tbody></table>

Las agrupaciones no tienen limitaciones en cuanto a donde pueden ser usadas. Podemos utilizarlas incluso dentro de una función.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
func f() string {
  var red = color.New(0xff0000)
  var green = color.New(0x00ff00)
  var blue = color.New(0x0000ff)

  ...
}
```

</td><td>

```go
func f() string {
  var (
    red   = color.New(0xff0000)
    green = color.New(0x00ff00)
    blue  = color.New(0x0000ff)
  )

  ...
}
```

</td></tr>
</tbody></table>

### Agrupaciones y orden en los imports

Debería haber siempre sólo dos grupos de `imports`:

- Paquete estándar 
- Todo lo demás

Esta es la forma que aplica `goimports` por defecto.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"
  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

</td><td>

```go
import (
  "fmt"
  "os"

  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

</td></tr>
</tbody></table>

### Nombre de paquetes

Cuando nombramos un paquete, debemos seguir las siguientes normas:

- Será completamente en minúscula. No habrá ninguna mayúscula para separar palabras ni guiones bajo.
- No debería de necesitar el uso de alias cuando sea importado.
- Debe ser corto y conciso. Recuerda que ese nombre será el identificador del paquete en cada llamada.
- No usar pluares. Por ejemplo, `net\url`, no `net\urls`.
- No utilizar nombres de paquetes como, `common`, `util`, `shared`, o `lib`. Son nombres malos y que no dicen nada.

Ver también [Package Names] y [Style guideline for Go packages].

  [Package Names]: https://blog.golang.org/package-names
  [Style guideline for Go packages]: https://rakyll.org/style-packages/

### Nombres de funciones

Seguimos las pautas de la comunidad de Go [MixedCaps for function names]. La única excepción es para las funciones de los test, las cuales pueden contener guiones bajos con el propósito agrupar los casos de los tests. ej., `TestMyFunction_WhatIsBeingTested`.

  [MixedCaps for function names]: https://golang.org/doc/effective_go.html#mixed-caps

### Alias en los imports

Los alias para los imports sólo deben ser usados si el nombre no coincide con el último elemento de la ruta importada.

```go
import (
  "net/http"

  client "example.com/client-go"
  trace "example.com/trace/v2"
)
```
En todos los demás escenarios, los alias para los imports deben ser evitados a no ser que haya un conflicto directo entre los paquetes.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"


  nettrace "golang.net/x/trace"
)
```

</td><td>

```go
import (
  "fmt"
  "os"
  "runtime/trace"

  nettrace "golang.net/x/trace"
)
```

</td></tr>
</tbody></table>

### Agrupación de funciones y orden

- Las funciones deben agruparse por proximidad de llamada.
- Las funciones de un fichero deben agruparse por su receptor.

Por lo tanto, las funciones públicas aparecerán al principio del fichero, después de las definiciones de
Therefore, exported functions should appear first in a file, after
`struct`, `const`, `var`.

El constructor `newXYZ()`/`NewXYZ()` debe aparecer después de que el tipo sea definido, pero antes del resto de métodos del receptor.

Como las funciones están agrupadas por receptor, las funciones de tipo *helper* deberían aparecer al final del fichero.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
func (s *something) Cost() {
  return calcCost(s.weights)
}

type something struct{ ... }

func calcCost(n []int) int {...}

func (s *something) Stop() {...}

func newSomething() *something {
    return &something{}
}
```

</td><td>

```go
type something struct{ ... }

func newSomething() *something {
    return &something{}
}

func (s *something) Cost() {
  return calcCost(s.weights)
}

func (s *something) Stop() {...}

func calcCost(n []int) int {...}
```

</td></tr>
</tbody></table>

### Reducir el código anidado

Tenemos que tener un código con poca anidación, donde pondremos el control de errores y condiciones de validación al principio provocando un return temprano o continuar un bucle. Reduciendo el número de código anidado a varios niveles.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
for _, v := range data {
  if v.F1 == 1 {
    v = process(v)
    if err := v.Call(); err == nil {
      v.Send()
    } else {
      return err
    }
  } else {
    log.Printf("Invalid v: %v", v)
  }
}
```

</td><td>

```go
for _, v := range data {
  if v.F1 != 1 {
    log.Printf("Invalid v: %v", v)
    continue
  }

  v = process(v)
  if err := v.Call(); err != nil {
    return err
  }
  v.Send()
}
```

</td></tr>
</tbody></table>

### Else innecesario
Si una variable es asignada tanto en el `if` como en el `else`, esto podrá ser remplazado por un único `if`.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
var a int
if b {
  a = 100
} else {
  a = 10
}
```

</td><td>

```go
a := 10
if b {
  a = 100
}
```

</td></tr>
</tbody></table>

### Declaración de variables globales

Las variables globales usan la palabra reservada `var`. No es necesario especificar el tipo, a no ser que sea una declaración de un tipo diferente que la expresión.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
var _s string = F()

func F() string { return "A" }
```

</td><td>

```go
var _s = F()
// Since F already states that it returns a string, we don't need to specify
// the type again.

func F() string { return "A" }
```

</td></tr>
</tbody></table>

Especifica el tipo si la expresión no es exactamente el tipo que buscas.

```go
type myError struct{}

func (myError) Error() string { return "error" }

func F() myError { return myError{} }

var _e error = F()
// F returns an object of type myError but we want error.
```

### Prefijo _ para las globales privadas

Todas las variables, `var` y constantes, `const` globales irán acompañadas del prefijo `_` para clarificar cuando se usan que son globales.

Excepción: los errores privados, deben ir acompañados del prefijo `err`.

Justificación: Las variables y constantes globales tienen el scope del paquete. Usando un nombre muy genérico se pueden provocar accidentes muy fácilmente.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
// foo.go

const (
  defaultPort = 8080
  defaultUser = "user"
)

// bar.go

func Bar() {
  defaultPort := 9090
  ...
  fmt.Println("Default port", defaultPort)

  // No veremos error de compilación si la primera línea de 
  // Bar() es borrada.
}
```

</td><td>

```go
// foo.go

const (
  _defaultPort = 8080
  _defaultUser = "user"
)
```

</td></tr>
</tbody></table>

### Composición en Structs

*Embedded types* (como los mutexs) deben estar al principio de la lista del `struct`, y deben añadir un salto de línea para separarlos de los campos regulares del `struct`.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
type Client struct {
  version int
  http.Client
}
```

</td><td>

```go
type Client struct {
  http.Client

  version int
}
```

</td></tr>
</tbody></table>

### Usa el nombre de los campos al inicializar Structs

Debes especificar los campos del `struct` cuando vayas a inicializarlo. Además esto es ahora de cumplimiento obligatorio utilizando [`go vet`].

  [`go vet`]: https://golang.org/cmd/vet/

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
k := User{"John", "Doe", true}
```

</td><td>

```go
k := User{
    FirstName: "John",
    LastName: "Doe",
    Admin: true,
}
```

</td></tr>
</tbody></table>

Excepción: Puedes omitir inicializar un `struct` con el nombre de sus campos en los *test tables* cuando tengan 3 o menos campos.

```go
tests := []struct{
  op Operation
  want string
}{
  {Add, "add"},
  {Subtract, "subtract"},
}
```

### Declaración de variables locales

La forma corta de declaración de variables (`:=`) debe ser usada si una variable se le va a asignar un valor explícito.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
var s = "foo"
```

</td><td>

```go
s := "foo"
```

</td></tr>
</tbody></table>

Sin embargo, hay algunos casos donde el valor por defecto queda más claro utilizando `var`. [Declarando Slices vacíos], por ejemplo.

  [Declarando Slices vacíos]: https://github.com/golang/go/wiki/CodeReviewComments#declaring-empty-slices

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
func f(list []int) {
  filtered := []int{}
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td><td>

```go
func f(list []int) {
  var filtered []int
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td></tr>
</tbody></table>

### nil es un slice válido

`nil` es un `slice` válido de tamaño 0. Eso quiere decir que,

- No debes devolver un `slice` de tamaño 0 explícitamente. Devuelve `nil` en su lugar.

  <table>
  <thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  if x == "" {
    return []int{}
  }
  ```

  </td><td>

  ```go
  if x == "" {
    return nil
  }
  ```

  </td></tr>
  </tbody></table>

- Para comprobar si un `slice` está vacío, siempre usaremos `len(s) == 0`. No comprobaremos si es `nil`.
  <table>
  <thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  func isEmpty(s []string) bool {
    return s == nil
  }
  ```

  </td><td>

  ```go
  func isEmpty(s []string) bool {
    return len(s) == 0
  }
  ```

  </td></tr>
  </tbody></table>

- Cuando declaramos un slice con `var`, es decir declararlo a su valor inicial, éste es usable inmediatamente sin necesidad de `make()`.

  <table>
  <thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  nums := []int{}
  // or, nums := make([]int)

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td><td>

  ```go
  var nums []int

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td></tr>
  </tbody></table>

### Reducir el scope de las variables

Cuando sea posible, reducir el scope de las variables. No hay que reducir el scope si entramos en conflicto con la regla [Reducir el código anidado](#reducir-el-cdigo-anidado).

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
err := ioutil.WriteFile(name, data, 0644)
if err != nil {
 return err
}
```

</td><td>

```go
if err := ioutil.WriteFile(name, data, 0644); err != nil {
 return err
}
```

</td></tr>
</tbody></table>

Si necesitas un resultado de una función fuera del `if`, entonces no debes de intentar reducir el scope.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
if data, err := ioutil.ReadFile(name); err == nil {
  err = cfg.Decode(data)
  if err != nil {
    return err
  }

  fmt.Println(cfg)
  return nil
} else {
  return err
}
```

</td><td>

```go
data, err := ioutil.ReadFile(name)
if err != nil {
   return err
}

if err := cfg.Decode(data); err != nil {
  return err
}

fmt.Println(cfg)
return nil
```

</td></tr>
</tbody></table>

### Evitar parámetros planos

Los parámetros planos en una función pueden dificultar la legibilidad. Para solucionar esto añadiremos los comentarios al estilo `C`, (`/* ... */`), para añadir el nombre de los parámetros cuando estos no sean obvios.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true, true)
```

</td><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true /* isLocal */, true /* done */)
```

</td></tr>
</tbody></table>

Todavía mejor, si remplazamos el `bool` por un tipo personalizado más legible y seguro. Además esto nos permitirá añadir nuevos estados que sólo (`true`/`false`) en el futuro.

```go
type Region int

const (
  UnknownRegion Region = iota
  Local
)

type Status int

const (
  StatusReady = iota + 1
  StatusDone
  // Maybe we will have a StatusInProgress in the future.
)

func printInfo(name string, region Region, status Status)
```

### Evita declarar strings con escapados

Go soporta los llamados [raw string literals](https://golang.org/ref/spec#raw_string_lit),
los cuales permiten múltiples lineas y comillas, usa este tipo de `string` para evitar tener que escapar tu cadena y que la haga más complicada de leer.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
wantError := "unknown name:\"test\""
```

</td><td>

```go
wantError := `unknown error:"test"`
```

</td></tr>
</tbody></table>

### Inicializando referencias a Struct

Usa siempre `&T{}` en lugar de `new(T)` cuando realices una inicialización por referencia de un `struct` a modo de ser consistente con la inicialización del `struct`.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
sval := T{Name: "foo"}

// inconsistent
sptr := new(T)
sptr.Name = "bar"
```

</td><td>

```go
sval := T{Name: "foo"}

sptr := &T{Name: "bar"}
```

</td></tr>
</tbody></table>

### Inicializando Maps

Elige `make(..)` para `maps` vacíos, y `maps` que sean llenados programáticamente.
Esto hace que la inicialización del `map` sea distinta de la declaración visualmente, haciendo que sea más fácil agregar capacidad en un futuro si fuera necesario.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
var (
  // m1 is safe to read and write;
  // m2 will panic on writes.
  m1 = map[T1]T2{}
  m2 map[T1]T2
)
```

</td><td>

```go
var (
  // m1 is safe to read and write;
  // m2 will panic on writes.
  m1 = make(map[T1]T2)
  m2 map[T1]T2
)
```

</td></tr>
<tr><td>

La declaración y la inicialización es visualmente similar.

</td><td>

La declaración y la inicialización es visualmente distinta.

</td></tr>
</tbody></table>

Donde sea posible, añade la capacidad cuando incialices los `maps` con `make()`. 
Where possible, provide capacity hints when initializing
maps with `make()`. Más información:
[Especificar una capacidad aproximada al Map](#especificar-una-capacidad-aproximada-al-map).

Por otro lado, si el `map` tiene un tamaño fijo de elementos, declara literalmente el `map` al inicializarlo.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
m := make(map[T1]T2, 3)
m[k1] = v1
m[k2] = v2
m[k3] = v3
```

</td><td>

```go
m := map[T1]T2{
  k1: v1,
  k2: v2,
  k3: v3,
}
```

</td></tr>
</tbody></table>

La regla de oro es utilizar la declaración literal de los `maps`, cuando añadas un grupo fijo de elementos en el momento de inicialización, de otro modo usa `make` (y especifica la capacidad si es posible).

### Formatos de Strings fuera del Printf

Si declaras un formato de `string` con el estilo del `Printf` fuera de una cadena, entonces debes hacerlo con una constante.

Esto ayudará a `go vet` a realizar un análisis estático del formato del `string`.

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
msg := "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td><td>

```go
const msg = "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td></tr>
</tbody></table>

### Nombra funciones al estilo Printf

Cuando declares una función de estilo `Printf`, asegúrate de que `go vet` pueda detectarla y comprobar el formato del `string`.

Esto quiere decir que debes usar los nombres al estilo `Printf` si es posible. `go vet` comprobará esto por defecto. Más información: [Printf family]

  [Printf family]: https://golang.org/cmd/vet/#hdr-Printf_family

Si no puedes utilizar los nombres predifinidos, acaba tus funciones con `f`: `Wrapf`, no `Wrap`. A `go vet` se le puede especificar que compruebe como funciones de estilo `Printf`, aquellas funciones que su nombre acabe por `f`.

```shell
$ go vet -printfuncs=wrapf,statusf
```

Más información: [go vet: Printf family check].

  [go vet: Printf family check]: https://kuzminva.wordpress.com/2017/11/07/go-vet-printf-family-check/

## Patrones

### Test Tables

Usa *table-driven tests* con [subtests] para evitar la duplicación de código cuando la lógica del test es repetitiva.

  [subtests]: https://blog.golang.org/subtests

<table>
<thead><tr><th>Incorrecto</th><th>Correcto</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestSplitHostPort(t *testing.T)

host, port, err := net.SplitHostPort("192.0.2.0:8000")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("192.0.2.0:http")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "http", port)

host, port, err = net.SplitHostPort(":8000")
require.NoError(t, err)
assert.Equal(t, "", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("1:8")
require.NoError(t, err)
assert.Equal(t, "1", host)
assert.Equal(t, "8", port)
```

</td><td>

```go
// func TestSplitHostPort(t *testing.T)

tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  {
    give:     "192.0.2.0:8000",
    wantHost: "192.0.2.0",
    wantPort: "8000",
  },
  {
    give:     "192.0.2.0:http",
    wantHost: "192.0.2.0",
    wantPort: "http",
  },
  {
    give:     ":8000",
    wantHost: "",
    wantPort: "8000",
  },
  {
    give:     "1:8",
    wantHost: "1",
    wantPort: "8",
  },
}

for _, tt := range tests {
  t.Run(tt.give, func(t *testing.T) {
    host, port, err := net.SplitHostPort(tt.give)
    require.NoError(t, err)
    assert.Equal(t, tt.wantHost, host)
    assert.Equal(t, tt.wantPort, port)
  })
}
```

</td></tr>
</tbody></table>

*Test tables* hace mucho más sencillo añadir contexto a mensajes de errores, reduce la lógica duplicada y añade nuevos casos de test.s.

Seguimos la convención de el `slice` de `structs` se le llamará `tests` y cada caso de test `tt`. Además explicitamos los valores de *input*(entrada) y *output*(salida) de cada test con los prefijos `give` y `want`.

```go
tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  // ...
}

for _, tt := range tests {
  // ...
}
```

### Opciones funcionales

*Functional options* es un patrón donde declaras un tipo `Option` que almacena la información en algún `struct` interno. Puedes aceptar un número variable de estas opciones y actuar según la información almacenada por las `options` en el `struct` interno.

Es recomendable utilizar este patrón para argumentos opcionales en los constructores y otras funciones públicas de tu API cuando prevés la necesidad de expandirlos, especialmente si ya tienes tres o más argumentos en dicha función.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// package db

func Connect(
  addr string,
  timeout time.Duration,
  caching bool,
) (*Connection, error) {
  // ...
}

// Timeout and caching siempre serán proporcionados,
// incluso si el usuario quiere usar los valores por defecto.

db.Connect(addr, db.DefaultTimeout, db.DefaultCaching)
db.Connect(addr, newTimeout, db.DefaultCaching)
db.Connect(addr, db.DefaultTimeout, false /* caching */)
db.Connect(addr, newTimeout, false /* caching */)
```

</td><td>

```go
type options struct {
  timeout time.Duration
  caching bool
}

// Option sobreescribe el comportamiento de Connect.
type Option interface {
  apply(*options)
}

type optionFunc func(*options)

func (f optionFunc) apply(o *options) {
  f(o)
}

func WithTimeout(t time.Duration) Option {
  return optionFunc(func(o *options) {
    o.timeout = t
  })
}

func WithCaching(cache bool) Option {
  return optionFunc(func(o *options) {
    o.caching = cache
  })
}

// Connect creates a connection.
func Connect(
  addr string,
  opts ...Option,
) (*Connection, error) {
  options := options{
    timeout: defaultTimeout,
    caching: defaultCaching,
  }

  for _, o := range opts {
    o.apply(&options)
  }

  // ...
}

// Options serán proporcionadas sólo si son necesarias.

db.Connect(addr)
db.Connect(addr, db.WithTimeout(newTimeout))
db.Connect(addr, db.WithCaching(false))
db.Connect(
  addr,
  db.WithCaching(false),
  db.WithTimeout(newTimeout),
)
```

</td></tr>
</tbody></table>

Recomendamos mirar también,

- [Self-referential functions and the design of options]
- [Functional options for friendly APIs]

  [Self-referential functions and the design of options]: https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html
  [Functional options for friendly APIs]: https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis

<!-- TODO: replace this with parameter structs and functional options, when to
use one vs other -->