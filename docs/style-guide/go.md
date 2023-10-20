<!--
For side-by-side code samples, use the following snippet.

~~~
<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
BAD CODE GOES HERE
```

</td><td>

```go
GOOD CODE GOES HERE
```

</td></tr>
</tbody></table>
~~~
Optionally add snippet bottom description before the </tbody></table> line.

~~~
<tr>
<td>DESCRIBE BAD CODE</td>
<td>DESCRIBE GOOD CODE</td>
</tr>
~~~

-->

# Golang code conventions

## Table of Contents

- [Introduction](#introduction)
- [Styles and guidelines](#styles-and-guidelines)
    - [Verify Interface Compliance](#verify-interface-compliance)
    - [Consider slices and maps copying](#consider-slices-and-maps-copying)
    - [Use `"time"` to handle time](#use-time-to-handle-time)
    - [Don't Panic](#dont-panic)
    - [Channel Size is One or None](#channel-size-is-one-or-none)
    - [Start Enums at One](#start-enums-at-one)
    - [Avoid Embedding Types in Public Structs](#avoid-embedding-types-in-public-structs)
    - [Exit in Main](#exit-in-main)
    - [Use field tags in marshaled structs](#use-field-tags-in-marshaled-structs)
    - [Package Names](#package-names)
    - [Function Names](#function-names)
    - [Imports](#imports)
    - [Reduce Nesting](#reduce-nesting)
    - [Unnecessary Else](#unnecessary-else)
    - [Prefix Unexported Globals with _](#prefix-unexported-globals-with)
    - [Local Variable Declarations](#local-variable-declarations)
    - [Interface naming](#interface-naming)
    - [Constants grouping](#constants-grouping)
    - [Prefer strconv over fmt](#prefer-strconv-over-fmt)
    - [Operations inside a cycle's body](#operations-inside-a-cycles-body)
    - [Prefer Specifying Container Capacity](#prefer-specifying-container-capacity)
    - [Functional Options](#functional-options)
    - [Avoid nested inline structs](#avoid-nested-inline-structs)
    - [Using context](#using-context)
    - [Go 1.18 features](#go-118-features)
- [Logging](#logging)
- [Error handling](#error-handling)
- [Unit tests](#unit-tests)
- [Code review](#code-review)
- [Automated tools](#automated-tools)
    - [Linter](#linter)
    - [Tools](#other)

## Introduction

It is assumed that you are familiar with the following resources:

- [Effective Go](https://golang.org/doc/effective_go.html)
- [Go Common Mistakes](https://github.com/golang/go/wiki/CommonMistakes)
- [50 Shades of Go](http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang/) 
- [Secure coding practices](https://github.com/OWASP/Go-SCP)

Mostly we rely one them and include in this document only the rules that are:

- specific inside our company
- extend the standard guidelines
- differs from the standard guidelines
- frequently violated


## Styles and guidelines

### Verify Interface Compliance

If struct intends to implement some interface verify compliance at compile time.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Handler struct {
  // ...
}



func (h *Handler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  ...
}
```

</td><td>

```go
type Handler struct {
  // ...
}

var _ http.Handler = (*Handler)(nil)

func (h *Handler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  // ...
}
```

</td></tr>
</tbody></table>

The statement `var _ http.Handler = (*Handler)(nil)` will fail to compile if
`*Handler` ever stops matching the `http.Handler` interface.

The right hand side of the assignment should be the zero value of the asserted
type. This is `nil` for pointer types (like `*Handler`), slices, and maps, and
an empty struct for struct types.

```go
type LogHandler struct {
  h   http.Handler
  log *zap.Logger
}

var _ http.Handler = LogHandler{}

func (h LogHandler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  // ...
}
```

### Consider slices and maps copying

Slices and maps contain pointers to the underlying data so be wary of scenarios
when they need to be copied.

#### Receiving slices and maps

Keep in mind that users can modify a map or slice you received as an argument
if you store a reference to it.

<table>
<thead><tr><th>Bad</th> <th>Good</th></tr></thead>
<tbody>
<tr>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = trips
}

trips := ...
d1.SetTrips(trips)

// Did you mean to modify d1.trips?
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

// We can now modify trips[0] without affecting d1.trips.
trips[0] = ...
```

</td>
</tr>

</tbody>
</table>

#### Returning Slices and Maps

Similarly, be wary of user modifications to maps or slices exposing internal
state.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

// Snapshot returns the current stats.
func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  return s.counters
}

// snapshot is no longer protected by the mutex, so any
// access to the snapshot is subject to data races.
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

// Snapshot is now a copy.
snapshot := stats.Snapshot()
```

</td></tr>
</tbody></table>

### Use `"time"` to handle time

Time is complicated. Incorrect assumptions often made about time include the
following.

1. A day has 24 hours
2. An hour has 60 minutes
3. A week has 7 days
4. A year has 365 days
5. [And a lot more](https://infiniteundo.com/post/25326999628/falsehoods-programmers-believe-about-time)

For example, *1* means that adding 24 hours to a time instant will not always
yield a new calendar day.

Therefore, always use the [`"time"`] package when dealing with time because it
helps deal with these incorrect assumptions in a safer, more accurate manner.

  [`"time"`]: https://golang.org/pkg/time/

#### Use `time.Duration` for periods of time

Use [`time.Duration`] when dealing with periods of time.

  [`time.Duration`]: https://golang.org/pkg/time/#Duration

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func poll(delay int) {
  for {
    // ...
    time.Sleep(time.Duration(delay) * time.Millisecond)
  }
}

poll(10) // was it seconds or milliseconds?
```

</td><td>

```go
func poll(delay time.Duration) {
  for {
    // ...
    time.Sleep(delay)
  }
}

poll(10*time.Second)
```

</td></tr>
</tbody></table>

Going back to the example of adding 24 hours to a time instant, the method we
use to add time depends on intent. If we want the same time of the day, but on
the next calendar day, we should use [`Time.AddDate`]. However, if we want an
instant of time guaranteed to be 24 hours after the previous time, we should
use [`Time.Add`].

  [`Time.AddDate`]: https://golang.org/pkg/time/#Time.AddDate
  [`Time.Add`]: https://golang.org/pkg/time/#Time.Add

```go
newDay := t.AddDate(0 /* years */, 0 /* months */, 1 /* days */)
maybeNewDay := t.Add(24 * time.Hour)
```

#### Use `time.Time` and `time.Duration` with external systems

Use `time.Duration` and `time.Time` in interactions with external systems when
possible. For example:

- Command-line flags: [`flag`] supports `time.Duration` via
  [`time.ParseDuration`]
- JSON: [`encoding/json`] supports encoding `time.Time` as an [RFC 3339]
  string via its [`UnmarshalJSON` method]
- SQL: [`database/sql`] supports converting `DATETIME` or `TIMESTAMP` columns
  into `time.Time` and back if the underlying driver supports it
- YAML: [`gopkg.in/yaml.v2`] supports `time.Time` as an [RFC 3339] string, and
  `time.Duration` via [`time.ParseDuration`].

  [`flag`]: https://golang.org/pkg/flag/
  [`time.ParseDuration`]: https://golang.org/pkg/time/#ParseDuration
  [`encoding/json`]: https://golang.org/pkg/encoding/json/
  [RFC 3339]: https://tools.ietf.org/html/rfc3339
  [`UnmarshalJSON` method]: https://golang.org/pkg/time/#Time.UnmarshalJSON
  [`database/sql`]: https://golang.org/pkg/database/sql/
  [`gopkg.in/yaml.v2`]: https://godoc.org/gopkg.in/yaml.v2

When it is not possible to use `time.Duration` or you have a strong reason to
bind to ceratin duration units (e.g. seconds), use `int` or `float64` and include
the unit in the name of the field.

For example, since `encoding/json` does not support `time.Duration`, the unit
is included in the name of the field.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// {"interval": 2}
type Config struct {
  Interval int `json:"interval"`
}
```

</td><td>

```go
// {"intervalMillis": 2000}
type Config struct {
  IntervalMillis int `json:"intervalMillis"`
}
```

</td></tr>
</tbody></table>

When it is not possible to use `time.Time` in these interactions, unless an
alternative is agreed upon, use `string` and format timestamps as defined in
[RFC 3339]. This format is used by default by [`Time.UnmarshalText`] and is
available for use in `Time.Format` and `time.Parse` via [`time.RFC3339`].

  [`Time.UnmarshalText`]: https://golang.org/pkg/time/#Time.UnmarshalText
  [`time.RFC3339`]: https://golang.org/pkg/time/#RFC3339

Although this tends to not be a problem in practice, keep in mind that the
`"time"` package does not support parsing timestamps with leap seconds
([8728]), nor does it account for leap seconds in calculations ([15190]). If
you compare two instants of time, the difference will not include the leap
seconds that may have occurred between those two instants.

  [8728]: https://github.com/golang/go/issues/8728
  [15190]: https://github.com/golang/go/issues/15190


### Don't Panic

Code running in production must avoid panics. Panics are a major source of
[cascading failures]. If an error occurs, the function must return an error and
allow the caller to decide how to handle it.

  [cascading failures]: https://en.wikipedia.org/wiki/Cascading_failure

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func run(args []string) {
  if len(args) == 0 {
    panic("an argument is required")
  }
  // ...
}

func main() {
  run(os.Args[1:])
}
```

</td><td>

```go
func run(args []string) error {
  if len(args) == 0 {
    return errors.New("an argument is required")
  }
  // ...
  return nil
}

func main() {
  if err := run(os.Args[1:]); err != nil {
    fmt.Fprintln(os.Stderr, err)
    os.Exit(1)
  }
}
```

</td></tr>
</tbody></table>

Panic/recover is not an error handling strategy. A program must panic only when
something irrecoverable happens such as a nil dereference. An exception to this is
program initialization: bad things at program startup that should abort the
program may cause panic.

```go
var _statusTemplate = template.Must(template.New("name").Parse("_statusHTML"))
```

Even in tests, prefer `t.Fatal` or `t.FailNow` over panics to ensure that the
test is marked as failed.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestFoo(t *testing.T)

f, err := os.CreateTemp("", "test")
if err != nil {
  panic("failed to set up test")
}
```

</td><td>

```go
// func TestFoo(t *testing.T)

f, err := os.CreateTemp("", "test")
if err != nil {
  t.Fatal("failed to set up test")
}
```

</td></tr>
</tbody></table>

#### Avoid potential panics

You should avoid panics wherever it is possible, even in tests:

- check structs for nil before taking their properties and calling methods on them
- use type assertions with the second bool parameter
- check maps for nil before writing to them

Since it is sometimes annoying it protects us from huge panic related issues.
Exceptions can be made in the following cases:

- receiving a pointer when it is cognitively clear that it is not nil

```go

func CreateUser(u *User) error {
  // it is safe
  Validate(u.Document)
  u.Age()

  // but it is not safe to use second level and higher properites and methods
  // println(u.Document.Period)
  if u.Document != nil {
    println(u.Document.Period)
  }
}

```

- returning value is a pointer and it is cognitively clear that it is not nil

```go

func NewDBClient() *Client {
  return &Client{}
}

func main() {
  c := NewDBClient()
  // it is safe
  c.Ping()
}

```

### Channel Size is One or None

Channels should usually have a size of one or be unbuffered. By default,
channels are unbuffered and have a size of zero. Any other size
must be subject to a high level of scrutiny.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
c := make(chan int, 64)
```

</td><td>

```go
c := make(chan int, 1)
// or
c := make(chan int)
```

</td></tr>
</tbody></table>

### Start Enums at One

The standard way of introducing enumerations in Go is to declare a custom type
and a `const` group with `iota`. Since variables have a 0 default value, you
should usually start your enums on a non-zero value.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota // 0
  Subtract // 1
  Multiply // 2
)

```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1 // 1
  Subtract // 2
  Multiply // 3
)

```

</td></tr>
</tbody></table>


### Avoid Embedding Types in Public Structs

These embedded types leak implementation details, inhibit type evolution, and
obscure documentation.

Assuming you have implemented a variety of list types using a shared
`AbstractList`, avoid embedding the `AbstractList` in your concrete list
implementations.
Instead, hand-write only the methods to your concrete list that will delegate
to the abstract list.

```go
type AbstractList struct {}

// Add adds an entity to the list.
func (l *AbstractList) Add(e Entity) {
  // ...
}

// Remove removes an entity from the list.
func (l *AbstractList) Remove(e Entity) {
  // ...
}
```

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// ConcreteList is a list of entities.
type ConcreteList struct {
  *AbstractList
}
```

</td><td>

```go
// ConcreteList is a list of entities.
type ConcreteList struct {
  list *AbstractList
}

// Add adds an entity to the list.
func (l *ConcreteList) Add(e Entity) {
  l.list.Add(e)
}

// Remove removes an entity from the list.
func (l *ConcreteList) Remove(e Entity) {
  l.list.Remove(e)
}
```

</td></tr>
</tbody></table>

Go allows [type embedding] as a compromise between inheritance and composition.
The outer type gets implicit copies of the embedded type's methods.
These methods, by default, delegate to the same method of the embedded
instance.

  [type embedding]: https://golang.org/doc/effective_go.html#embedding

The struct also gains a field by the same name as the type.
So, if the embedded type is public, the field is public.
To maintain backward compatibility, every future version of the outer type must
keep the embedded type.

An embedded type is rarely necessary.
It is a convenience that helps you avoid writing tedious delegate methods.

Even embedding a compatible AbstractList *interface*, instead of the struct,
would offer the developer more flexibility to change in the future, but still
leak the detail that the concrete lists use an abstract implementation.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// AbstractList is a generalized implementation
// for various kinds of lists of entities.
type AbstractList interface {
  Add(Entity)
  Remove(Entity)
}

// ConcreteList is a list of entities.
type ConcreteList struct {
  AbstractList
}
```

</td><td>

```go
// ConcreteList is a list of entities.
type ConcreteList struct {
  list AbstractList
}

// Add adds an entity to the list.
func (l *ConcreteList) Add(e Entity) {
  l.list.Add(e)
}

// Remove removes an entity from the list.
func (l *ConcreteList) Remove(e Entity) {
  l.list.Remove(e)
}
```

</td></tr>
</tbody></table>

Either with an embedded struct or an embedded interface, the embedded type
places limits on the evolution of the type.

- Adding methods to an embedded interface is a breaking change.
- Removing methods from an embedded struct is a breaking change.
- Removing the embedded type is a breaking change.
- Replacing the embedded type, even with an alternative that satisfies the same
  interface, is a breaking change.

Although writing these delegate methods is tedious, the additional effort hides
an implementation detail, leaves more opportunities for change, and also
eliminates indirection for discovering the full List interface in
documentation.


### Exit in Main

Go programs use [`os.Exit`] or [`log.Fatal*`] to exit immediately. (Panicking
is not a good way to exit programs, please [don't panic](#dont-panic).)

  [`os.Exit`]: https://golang.org/pkg/os/#Exit
  [`log.Fatal*`]: https://golang.org/pkg/log/#Fatal

Call one of `os.Exit` or `log.Fatal*` **only in `main()`**. All other
functions should return errors to signal failure.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func main() {
  body := readFile(path)
  fmt.Println(body)
}

func readFile(path string) string {
  f, err := os.Open(path)
  if err != nil {
    log.Fatal(err)
  }

  b, err := io.ReadAll(f)
  if err != nil {
    log.Fatal(err)
  }

  return string(b)
}
```

</td><td>

```go
func main() {
  body, err := readFile(path)
  if err != nil {
    log.Fatal(err)
  }
  fmt.Println(body)
}

func readFile(path string) (string, error) {
  f, err := os.Open(path)
  if err != nil {
    return "", err
  }

  b, err := io.ReadAll(f)
  if err != nil {
    return "", err
  }

  return string(b), nil
}
```

</td></tr>
</tbody></table>


### Use field tags in marshaled structs

Any struct field that is marshaled into JSON, YAML,
or other formats that support tag-based field naming
should be annotated with the relevant tag.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Stock struct {
  Price int
  Name  string
}

bytes, err := json.Marshal(Stock{
  Price: 137,
  Name:  "TV",
})
```

</td><td>

```go
type Stock struct {
  Price int    `json:"price"`
  Name  string `json:"name"`
  // Safe to rename Name to Symbol.
}

bytes, err := json.Marshal(Stock{
  Price: 137,
  Name:  "TV",
})
```

</td></tr>
</tbody></table>

Rationale:
The serialized form of the structure is a contract between different systems.
Changes to the structure of the serialized form--including field names--break
this contract. Specifying field names inside tags makes the contract explicit,
and it guards against accidentally breaking the contract by refactoring or
renaming fields.


### Package Names

When naming packages, choose a name that is:

- All lower-case. No capitals or underscores.
- Does not need to be renamed using named imports at most call sites.
- Short and succinct. Remember that the name is identified in full at every call
  site.
- Prefer non-plural. For example, `net/url`, not `net/urls`.
- Not "common", "util", "shared", or "lib". These are bad, uninformative names.

See also [Package Names] and [Style guideline for Go packages].

  [Package Names]: https://blog.golang.org/package-names
  [Style guideline for Go packages]: https://rakyll.org/style-packages/


### Function Names

- use [MixedCaps for function names]
- function name must answer the question "what to do ?" (except getters)
- getter functions must not contain the `Get` part (`Name()` instead of `GetName()`)
- setter functions must contain the `Set` part (`SetName(n string)` instead of `Name(s string)`)
- predicates must start with `Is`, `Can`, etc. prefix (`IsUserValid() bool` instead of `UserValid() bool`)


### Imports

There should be the following import groups:

- Standard
- 3rd party
- Current module

There should be one blank line between the groups.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "wasfaty.api/pkg/log"
  "os"
  "github.com/satori/go.uuid"
)
```

</td><td>

```go
import (
  "fmt"
  "os"

  "github.com/satori/go.uuid"

  "wasfaty.api/pkg/log"
)
```

</td></tr>
</tbody></table>

Use aliases only they are unavoidable. In case of collisions:

- 3rd party import must be without alias
- make alias name as concatenation of the two last segments (`wasfaty.api/pkg/log` => `pkgLog`)

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go

import (
  zLog "github.com/rs/zerolog/log"
  
  "wasfaty.api/pkg/log"
)

```

</td><td>

```go

import (
  "github.com/rs/zerolog/log"
  
  pkgLog"wasfaty.api/pkg/log"
)

```

</td></tr>
</tbody></table>

### Reduce Nesting

Code should reduce nesting where possible by handling error cases/special
conditions first and returning early or continuing the loop. Reduce the amount
of code that is nested multiple levels.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
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

### Unnecessary Else

If a variable is set in both branches of an if, it can be replaced with a single if.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
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

### Prefix Unexported Globals with _

Prefix unexported top-level `var`s and `const`s with `_` to make it clear when
they are used that they are global symbols.

Rationale: Top-level variables and constants have a package scope. Using a
generic name makes it easy to accidentally use the wrong value in a different
file.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
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

  // We will not see a compile error if the first line of
  // Bar() is deleted.
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


### Local Variable Declarations

Short variable declarations (`:=`) should be used if a variable is being set to
some value explicitly.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
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

However, there are cases where the default value is clearer when the `var`
keyword is used. [Declaring Empty Slices], for example.

  [Declaring Empty Slices]: https://github.com/golang/go/wiki/CodeReviewComments#declaring-empty-slices

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
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


### Interface naming

Don't use `I-prefix` (`IReader`). Wherever it is possible use the [go interface naming approach](https://golang.org/doc/effective_go#interface-names) with `-er` suffix (`Reader, Writer`).


### Constants grouping

Group and name constant by type. Consider how you would see them in intellisense window.
Split groups with blank lines.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
const (
  // 1) status constants don't have the common Status prefix
  // 2) gender constants don't have the common Gender prefix
  // 3) constants are not grouped
  AcceptedStatus = "accepted"
  FGender = "F"
  RejectedStatus = "rejected"
  MGender = "M"
)
```

</td><td>

```go
const (
  StatusAccepted = "accepted"
  StatusRejected = "rejected"
  
  GenderF = "F"
  GenderM = "M"
)
```

</td></tr>
</tbody></table>


### Prefer strconv over fmt

When converting primitives to/from strings, `strconv` is faster than
`fmt`.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
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


### Operations inside a cycle's body

Move repeatable operations outside a cycle's body.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
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

</td></tr>
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


### Prefer Specifying Container Capacity

Specify container capacity where possible in order to allocate memory for the
container up front. This minimizes subsequent allocations (by copying and
resizing of the container) as elements are added.

#### Specifying Map Capacity Hints

Where possible, provide capacity hints when initializing
maps with `make()`.

```go
make(map[T1]T2, hint)
```

Providing a capacity hint to `make()` tries to right-size the
map at initialization time, which reduces the need for growing
the map and allocations as elements are added to the map.

Note that, unlike slices, map capacity hints do not guarantee complete,
preemptive allocation, but are used to approximate the number of hashmap buckets
required. Consequently, allocations may still occur when adding elements to the
map, even up to the specified capacity.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
m := make(map[string]os.FileInfo)

files, _ := os.ReadDir("./files")
for _, f := range files {
    m[f.Name()] = f
}
```

</td><td>

```go

files, _ := os.ReadDir("./files")

m := make(map[string]os.DirEntry, len(files))
for _, f := range files {
    m[f.Name()] = f
}
```

</td></tr>
<tr><td>

`m` is created without a size hint; there may be more
allocations at assignment time.

</td><td>

`m` is created with a size hint; there may be fewer
allocations at assignment time.

</td></tr>
</tbody></table>

#### Specifying Slice Capacity

Where possible, provide capacity hints when initializing slices with `make()`,
particularly when appending.

```go
make([]T, length, capacity)
```

Unlike maps, slice capacity is not a hint: the compiler will allocate enough
memory for the capacity of the slice as provided to `make()`, which means that
subsequent `append()` operations will incur zero allocations (until the length
of the slice matches the capacity, after which any appends will require a resize
to hold additional elements).

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for n := 0; n < b.N; n++ {
  data := make([]int, 0)
  for k := 0; k < size; k++{
    data = append(data, k)
  }
}
```

</td><td>

```go
for n := 0; n < b.N; n++ {
  data := make([]int, size)
  for k := 0; k < size; k++{
    data[k] = k
  }
}

// or

for n := 0; n < b.N; n++ {
  data := make([]int, 0, size)
  for k := 0; k < size; k++{
    data = append(data, k)
  }
}
```

</td></tr>
<tr><td>

```
BenchmarkBad-4    100000000    2.48s
```

</td><td>

```
BenchmarkGood-4   100000000    0.21s
```

</td></tr>
</tbody></table>


### Functional Options

Functional options is a pattern in which you declare an opaque `Option` type
that records information in some internal struct. You accept a variadic number
of these options and act upon the full information recorded by the options on
the internal struct.

Use this pattern for optional arguments in constructors and other public APIs
that you foresee needing to expand, especially if you already have three or
more arguments on those functions.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// package db

func Open(
  addr string,
  cache bool,
  logger *zap.Logger
) (*Connection, error) {
  // ...
}
```

</td><td>

```go
// package db

type Option interface {
  // ...
}

func WithCache(c bool) Option {
  // ...
}

func WithLogger(log *zap.Logger) Option {
  // ...
}

// Open creates a connection.
func Open(
  addr string,
  opts ...Option,
) (*Connection, error) {
  // ...
}
```

</td></tr>
<tr><td>

The cache and logger parameters must always be provided, even if the user
wants to use the default.

```go
db.Open(addr, db.DefaultCache, zap.NewNop())
db.Open(addr, db.DefaultCache, log)
db.Open(addr, false, zap.NewNop())
db.Open(addr, false, log)
```

</td><td>

Options are provided only if needed.

```go
db.Open(addr)
db.Open(addr, db.WithLogger(log))
db.Open(addr, db.WithCache(false))
db.Open(
  addr,
  db.WithCache(false),
  db.WithLogger(log),
)
```

</td></tr>
</tbody></table>

We use the following pattern:
- `Option` interface that holds an unexported `apply(*options)` method
- interface implementors (concrete options) write their values passed to `apply` method `options`

```go
type options struct {
  cache  bool
  logger *zap.Logger
}

type Option interface {
  apply(*options)
}

type cacheOption bool

func (c cacheOption) apply(opts *options) {
  opts.cache = bool(c)
}

func WithCache(c bool) Option {
  return cacheOption(c)
}

type loggerOption struct {
  Log *zap.Logger
}

func (l loggerOption) apply(opts *options) {
  opts.logger = l.Log
}

func WithLogger(log *zap.Logger) Option {
  return loggerOption{Log: log}
}

// Open creates a connection.
func Open(
  addr string,
  opts ...Option,
) (*Connection, error) {
  options := options{
    cache:  defaultCache,
    logger: zap.NewNop(),
  }

  for _, o := range opts {
    o.apply(&options)
  }

  // ...
}
```

It may seem overhead but still worth our spent time. You should always prefer `functional arguments`
over `*pointer` optional arguments and `...interface{}`. Some times passing `Config` object feats
better. Here pros vs cons table you can consider deciding between `Config` and `functional arguments`:

<table>
<thead><tr><th>Config</th><th>Functional arguments</th></tr></thead>
<tbody>
<tr><td>

```
Pros:
  - Simple to use
  - Occupies the least number of symbols
    in function signature

Cons:
  - Doesn't clarify what fields are required
    (pointers serve just as pointer instead of optional fields)
  - It is possible to omit even required fields
    without compilation error
  - Config fields should be well commented 
  - Optional fields handling requires if != nil checks
    for setting defaults
```

</td><td>

```
Pros:
  - Fully resolves the required/optional ambiguity
  - Releases user from passing nil arguments for optional args
  - Allows to set the default config in constructor
    and then iterate over options to override defaults

Cons:
  - When there are a lot of required args (lets say >5)
    leads to a huge function signature. But still it is possible
    to combine all the required args to Config struct
    assuming that all the fields are required and implement
    optional values as `functional arguments`
  - Is complicated compared to Config
```

</td></tr>
</tbody></table>

There is one more option that may seem a nice tradeoff:
```go

type Client {
  host string
  password string
}

func NewClient(host string) *Client {
  return &Client{host: host}
}

func(c *Client) WithPassword(p string) *Client {
  c.password = p
  return c
}

func main() {
  _ = NewClient("host").WithPassword("password")
}

```

But sometimes arguments are required exactly in constructor. For example:

```go

type Client {
  conn db.Connection
}

func NewClient(host string, *password string) *Client {
  dsn := host
  if *password != nil {
    dsn += ":" + *password
  }
  return &Client{conn: db.NewConnection(dsn)}
}

```

This approach serves rather for dynamic changes of program's behavior than for
structure initialization and settings immutable fields:

```go

const _defaultColor = "black"

type Printer {
  color string
}

func NewPrinter() *Printer {
  return &Printer{color: _defaultColor}
}

func(p *Printer) WithColor(c string) *Printer {
  p.color = c
  return p
}

func main() {
  p := NewPrinter()
  for ... {
    p.WithColor(_defaultColor)
    if ... {
      p.WithColor(customColor)
    }
    p.Print()
  }
}

```

### Avoid nested inline structs

Always avoid inline structs especially for public structs even if they are small
and seem to be never used separately:

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type User struct {
  Name    string
  Address struct {
    City   string
    Street string
  }
}
```

</td><td>

```go
type User struct {
  Name    string
  Address *Address
}

type Address struct {
  City   string
  Street string
}
```

</td></tr>
</tbody></table>

### Using context

Don't forget to receive a context as the first argument. It is required for most
public methods because:

- most of external libraries are designed to work
  with context and we shouldn't always pass context.Background
- context may transport cancellation signal and
  we should have an opportunity to handle it
- context may transport values that you'll need in future

For example we actively use context in `pkg/log` and `pkg/cerror` to take
`requestID` from there. If we need some more end-to-end data we'll easily
add it to context and be able to read it every method that receives context.
So the only cases when we can omit context in our function's signature are:

- public functions that don't use context and don't pass it to other packages.
  But you must be absolutely sure that you'll don't need it in future
  because in the opposite case you'll have change the signature.
  Also you function should not write logs or produce errors because their
  methods receive context and `Background` is not suitable as soon as we
  loose end-to-end values in logs
- private functions that don't use context. As soon as they are private
  that won't be a great deal to change the signature without pain for
  function consumers

Here some examples of valid functions that use or don't use context:

<table>
<thead><tr><th>Context may be omitted</th><th>Context is required</th></tr></thead>
<tbody>
<tr><td>

```go
func Contains(list []string, value string) bool {
  for _, v := range list {
		if v == value {
			return true
		}
	}
	return false
}

func TimeToDateString(t time.Time) string {
  return t.Format("2006-01-02")
}

func ParseDate(s string) (time.Time, bool) {
  t, err := time.Parse("2006-01-02", s)
  return t, err == nil
}
```

</td><td>

```go
func (r *Repo) CreateUser(ctx context.Context, u *User) error {
  err := r.db.ModelContext(ctx, u).Insert()
  if err != nil {
    return cerror.New(ctx, cerror.KindInternal, err)
  }
  return nil
}

func ParseDate(ctx context.Context, s string) (time.Time, bool) {
  t, err := time.Parse("2006-01-02", s)
  if err != nil {
    log.Error(ctx, err)
  }
  return t, err == nil
}

func ParseDate(ctx context.Context, s string) (*time.Time, error) {
  t, err := time.Parse("2006-01-02", s)
  if err != nil {
    return nil, cerror.New(ctx, cerror.KindBadParams, err)
  }
  return &t, nil
}
```

</td></tr>
</tbody></table>

Be careful passing context to packages that listen cancellation signal
on the given context and interrupt operation execution. This will or will not
be a desired behavior depended on your case. For example we want to make
a graceful shutdown of HTTP server (wait until already received requests will be handled)
and pass context to db's transaction:

```go

func (r *Repo) CreateUser(ctx context.Context, u *User) error {
  // even small transaction will be rollbacked immediatelly on ctx.Done
  // tx, err := r.db.BeginContext(ctx)
  
  // we assume that we have small transaction that should not be canceled
  tCtx := context.Background()
  tx, err := r.db.BeginContext(tCtx)

  // or use context with timeout
  // tCtx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
  // defer cancel()
  // tx, err := r.db.BeginContext(tCtx)

  // ... do some transactional operations

  // but don't forget to use the original context for errors
  if err != nil {
    return cerror.New(ctx /* !!! NOT tCtx */, cerror.KindInternal, err)
  }
  return nil
}

```

### Go 1.18 features

At the moment we don't use the following go 1.18 features:

- [generics](https://go.dev/doc/tutorial/generics) because a lot of linters
  have issues with new syntax. We don't want to lose essential analyzers
  such as `staticcheck`. But you can consider using generics in your
  project when most of the linters will work with them correctly
- `any` type. Because existing projects already use `interface{}`
  and we don't wont to be inconsistent. Also some linters argue
  currently about `any`. If linters will be fixed at the moment when
  new project starts you can consider using `any` but you should
  have only one of `any` or `interface{}` in your code base


## Logging

- use the internal log package
- it is forbidden to use any 3rd party loggers outside the log package folder
- log levels are declared in the level.go file
- log field names are declared in the field.go file
- errors wrapped with cerror package already have their built-in
  log methods (that also use the internal log package) that
  should be used instead. See the [errors section](#error-logging)
- choose a log level wisely. It is assumed that we have `info` severity level
  for production and it should not have a huge amount of logs.
- write a meaningful message that would be clear even for those who is out of business context
- use `event.WithPayload` method to bind helpful information (e.g. recordID, actorID)
- avoid using `event.WithValue` with different from predefined keys
  to limit amount of different keys in logs. But still we left this opportunity and you
  can use it if you have strong reasons

Incoming http requests are logged by audit log middleware
that include common information (http method, status, etc.).

If http request was processed with error, it is logged by LogHTTPHandlerErrorCtx method.
It logs general error information just to avoid cases when somebody forgot
to log error in place as described in the [errors section](#error-logging).


## Error handling

We use the internal cerror package that provides a common approach for errors wrapping. 

For __EVERY__ component inside the repository (i.e. pkgs, services, etc.)
__IS REQUIRED__ to wrap errors with the help of this package (i.e. each error should be created as `cerror.New`).

### Main functions of cerror package

`New(ctx context.Context, kind Kind, err error) *CError` - is used as the wrapper for external errors.
Context serves as a storage for end-to-end values (e.g. `request_id`). It is safe to pass `context.Background`
or `nil` if there is no suitable context in an error propagator. But it is highly recommended to
pass meaningful context wherever possible.

`NewValidationError(ctx context.Context, errFields map[string]string) *ValidationError` - should be used
when it is needed to return detailed validation error info with description for each invalid field.
It is based on `CError` and extends/overrides it with some additional methods.


### Common rules

- error message should be clear even for those who don't know implementation details
- error message should contain debug useful information (i.e. attribute name/value that caused an error,
  collection name where record was not found, etc.). `*CError.WithPayload` method serves to provide such data
  Don't include this information directly in `error`
- no private data in log messages
- avoid `failed to` , `unable to`, `error when`, etc. prefixes in error messages
  unless they are really needed. The mess logs with extra information because error log level
  already means that something went wrong
- `not found` logging depends on case, often they are just the expected behavior

### Error logging

`*CError.Log***` method is used to log an error in common log structure. The following info is provided:
- `error` - original error
- `errorKind` - kind of error (e.g. NotFound, BadParams, ...)
- `operations` - N-function names before from the top of the stack. Serves as short a stack trace
- `requestID` - end-to-end value of request id if it is present in a provided context
- `errorPayload` - additional error related data
- `message` - additional description. Mostly should be avoided as soon as we have error field

Also, the stack trace is logged as additional log message with `TRACE` severity level.
It includes `error` and `errorStack` fields.


### Usage in code

Let's consider the following example of HTTP request handling by one of our services:

`Incoming HTTP request` -> `Handler layer` -> `Service layer` -> `Persistence layer`

An error may occur on each of those layers. It __IS REQUIRED__ to wrap the error
in `cerror` package __AS SOON AS POSSIBLE__. Other layers should consider the
error they have received as the already `cerror` wrapped error and just return
it as is. This approach serves to avoid `cerror` wrapping duplication:

__WRONG__: error was wrapped on `adapter` layer then on `usecase` layer
then on `controller` layer`. Finally, we have 3 log messages for the same error and 3-level nested cerror
that is marshalled as something like `{error: no rows{error no rows {error no rows}}}`

__OK__: error was wrapped on `adapter` layer. All other layers do just `return err`
when they received the error from the previous layer

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// controller/controller.go

type Controller struct {
  uc *UseCase
}

func(c *Controller) Create(ctx context.Context, u *User) error {
  err := c.uc.Create(ctx, u)
  if err != nil {
    // got error from internal package
    // wrapping and logging for the 3rd time
    // NOT OK
    return cerror.New(ctx, cerror.ErrKind(err), err).LogError()
  }
  return nil
}

// usecase/usecase.go

type UseCase struct {
  r *Repo
}

func(u *UseCase) Create(ctx context.Context, u *User) error {
  err := u.r.Create(ctx, u)
  if err != nil {
    // got error from internal package
    // wrapping and logging for the 2nd time
    // NOT OK
    return cerror.New(ctx, cerror.ErrKind(err), err).LogError()
  }
  return nil
}

// adapter/repo.go

import "third/party/db"

type Repo struct {
  d *db.DB
}

func(r *Repo) Create(ctx context.Context, u *User) error {
  err := r.d.Insert(ctx, u)
  if err != nil {
    // got error from 3rd party package
    // must wrap and log
    // GOOD
    return cerror.New(ctx, cerror.ErrKind(err), err).LogError()
  }
  return nil
}
```

</td><td>

```go
// controller/controller.go

type Controller struct {
  uc *UseCase
}

func(c *Controller) Create(ctx context.Context, u *User) error {
  // return error from internal package as is
  // GOOD
  return c.uc.Create(ctx, u)
}

// usecase/usecase.go

type UseCase struct {
  r *Repo
}

func(u *UseCase) Create(ctx context.Context, u *User) error {
  // return error from internal package as is
  // GOOD
  return u.r.Create(ctx, u)
}

// adapter/repo.go

import "third/party/db"

type Repo struct {
  d *db.DB
}

func(r *Repo) Create(ctx context.Context, u *User) error {
  err := r.d.Insert(ctx, u)
  if err != nil {
    // got error from 3rd party package
    // must wrap and log
    // GOOD
    return cerror.New(ctx, cerror.ErrKind(err), err).LogError()
  }
  return nil
}
```

</td></tr>
</tbody></table>

### Error Naming

For error values stored as global variables, use the prefix `err`:

```go
var (
  ErrBrokenLink = errors.New("link is broken")
  ErrCouldNotOpen = errors.New("could not open")

  errNotFound = errors.New("not found")
)
```

## Unit tests

We apply the following principles for our unit tests:

- test must be in `_test` package to limit access to private parts of the package
  and simulate a real end user behavior
- don't make methods that you want to test public just making them
  available in tests. You should consider only public interface that
  you want to expose to the end user. You should be able to test
  all the cases by calling public interface with different arguments or
  redesign your package's API 
- use the following libraries:
    - `github.com/tj/assert` - for simple scenario
    - `github.com/stretchr/testify/suite` - for tests that require some setup
      and use test lifecycle hooks
      (e.g. create db connection on start and close after all the tests)
- is required to cover public API of:
    - every package from /pkg
    - `adapter, controller, usecase, workflow` service layers

The preferred approach to mock dependencies is to implement dependency interfaces as
dynamic functions that are set each time right before calling. This approach
gives the following benefits:

- a mock remains simple
- test implementor decides how the mock behavior should look like depended on the test purposes
- it's easy to provide a reliable set of checks right in the mock method (e.g. argument checks)
- the mock methods implementation remains simple

The only downside of this approach is that tests couldn't be run in parallel but for parallel running
other types of mocks will be even harder to implement.
We highly recommend to use this pattern unless you have a specific case.

Example:

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// usecase/usecase.go
type Repo interface {
    Search(*Params) []*Data 
}

type UseCase struct {
    r repo
}

func NewUseCase(r repo) *UseCase {
  return &UseCase{r: r}
}

func(u *UseCase) Search(value string, limit int) []*Data {
    //...
}

// usecase/usecase_test.go

type repoMock struct {
    d []*Data 
}

func(r *repoMock) Search(p *Params) []*Data  {
    // Need to implement a search algorithm
    // that may be not straightforward.
    // Consider search with nested
    // objects and 10 search params.
    // Sometimes writing a mock will be
    // even more complex then real implementation.
    // It is easy to make a mistake in the mock algorithm
    values := []*Data{}
    for _, d := range r.d {
      if d.Value == p.Value {
        values = append(values, d)
        if len(values) == p.Limit {
          return values
        }
      }
    }
    return values

    // or return all the data but even don't
    // have an opportunity to check arguments
    // return r.d
}

func TestSearch(t *testing.T) {
    v := "vehicle"
    l := 10
    // Need to prepare appropriate data that
    // will be found by our algorithm.
    // Need to remember how algorithm works
    // that may be rather difficult for large algorithms
    expectedData := []*Data{{Value: v}}

    r := &repoMock{d: expected}  
    uc := usecase.NewUseCase(r)
    assert.Equal(t, expectedData, uc.Search(v, l))
}
```

</td><td>

```go
// usecase/usecase.go
type Repo interface {
    Search(*Params) []*Data 
}

type UseCase struct {
    r repo
}

func NewUseCase(r repo) *UseCase {
  return &UseCase{r: r}
}

func(u *UseCase) Search(value string, limit int) []*Data {
    //...
}

// usecase/usecase_test.go

type repoMock struct {
    searchFn Search(*Params) []*Data 
}

func(r *repoMock) Search(p *Params) []*Data  {
    return searchFn(p)
}

func TestSearch(t *testing.T) {
    v := "vehicle"
    l := 10
    expectedData := []*Data{new(Data)}

    r := &repoMock{
      searchFn: func(p *Params) []*Data {
        // able to check arguments
        assert.Eqaul(t, v, p.Value)
        assert.Eqaul(t, l, p.Limit)

        // able to return appropriate for our usecase result
        // no need to implement algorithm at all
        return expectedData
    }
  
    uc := usecase.NewUseCase(r)
    assert.Equal(t, expectedData, uc.Search(v, l))
}
```

</td></tr>
</tbody></table>

This pattern really shines for complex mocks and gets us rid of the choice between
writing algorythm for mock and having not reliable tests.


## Code review

PR owner:

- fill all the PR checklist sections
- wait for green actions before adding reviewers
- it is forbidden to merge PRs with red actions
- add two or more reviewers
- wait for two or more accepts before merging
- choose the most familiar with PR's context reviewers
- send PR link to the team's slack channel, tag reviewers
- reply with `done` or `fixed` to conversations
- don't resolve conversations. Only a conversation's
  author may close it (except `[nit]` conversations)
- notify that fixing is finished to PR's thread in slack
- be polite. Remember that reviewers spend their time trying
  help you rather then annoy you

PR reviewer:

- pay attention to the proposed code base changes in the following priority:
  - new functionality:
      - should work correctly and implement all business features described in the task
      - should not break the other functionality
      - should not cause potential performance issues
      - should be maintainable
  - style. Should not violate the accepted code conventions

  When you propose some change, it should resolve one of these issues
  rather than be you personal preference.
  Issues that are out of the code conventions' scope should be discussed and
  added to the code conventions or rejected.

- mark minor conversations that you don't expect to be fixed with `[nit]` comment.
  The PR's author decides to fix or not and may close conversation.
- issues that are out of the code conventions scope should be discussed and
  added to the code conventions or rejected
- commit all you conversations at once (use `start review` and `add review comment`
  instead of `add single comment`)
- be polite. You should help the author rather then annoy them
- finish review in 24h


## Automated tools

The huge amount of checks are implemented by automated tools. Using them is the best way avoid mistakes
and simplify a code review process.

### Linter

We use the [golangci-lint](https://golangci-lint.run/) tool.

We have the following custom linters:
- `cerrl` is a golangci-lint plugin.It requires installation that is described [here](https://github.com/edenlabllc/go-lint-cerrl)


### Formatting

We recommend to use the standard `goimports` tool and format your code on save.
