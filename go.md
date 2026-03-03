## Go (Golang) — Detailed Notes with Examples, Exercises, and Interview Questions

These notes are aimed at **strong Go fundamentals + backend interviews + real-world production code**. Most examples are minimal and runnable. Copy snippets into `main.go` and run `go run .` (or `go run main.go`).

---

## Quick Setup / Tooling

- **Install Go**: follow official docs for your OS.
- **Check version**:

```bash
go version
```

- **Initialize a module**:

```bash
mkdir hello-go && cd hello-go
go mod init example.com/hello
```

- **Run**:

```bash
go run .
```

- **Format / lint basics**:
  - `gofmt` (standard formatter): `gofmt -w .`
  - `go vet`: `go vet ./...`
  - Race detector: `go test -race ./...`

### Go command cheat sheet (very useful)

```bash
go env                      # print Go environment
go list ./...               # list packages
go test ./...               # run tests
go test -run TestX ./...    # run specific tests
go test -race ./...         # race detector
go test -cover ./...        # coverage summary
go test -coverprofile=cover.out ./... && go tool cover -html=cover.out
go build ./...              # compile
go build -o app ./cmd/app   # build binary
go clean -cache -testcache  # clear caches
go fmt ./...                # formats packages (uses gofmt)
go vet ./...                # static checks
```

### Environment knobs (know they exist)

- **`GOMAXPROCS`**: max OS threads executing Go code simultaneously (defaults to CPU count).
- **`GOGC`**: GC target percentage (higher → less frequent GC, more memory; lower → more frequent GC).
- **`GODEBUG`**: debug flags for runtime behavior (advanced).

---

## 1) Core Language Fundamentals (Must Be Strong)

### 1.1 Syntax & Types

#### Basic types

- **Integers**: `int` (word size), `int8/16/32/64`, `uint*`, `uintptr`
- **Bytes and runes**:
  - `byte` = `uint8` (often raw bytes)
  - `rune` = `int32` (Unicode code point)
- **Floats**: `float32`, `float64`
- **Complex**: `complex64`, `complex128`
- **Boolean**: `bool`
- **String**: immutable bytes (UTF-8 by convention)

```go
package main

import "fmt"

func main() {
	var i int = 42
	var u uint = 42
	var f float64 = 3.14
	var ok bool = true
	var b byte = 0x41   // 65
	var r rune = '世'   // rune literal
	var s string = "Go"
	var c complex128 = complex(2, 3)

	fmt.Printf("i=%d u=%d f=%.2f ok=%v b=%d r=%d (%q) s=%q c=%v\n",
		i, u, f, ok, b, r, r, s, c)
}
```

#### Zero values

Every variable has a default value:

- numbers → `0`
- bool → `false`
- string → `""`
- pointers/slices/maps/chans/funcs/interfaces → `nil`

```go
var (
	i int
	f float64
	ok bool
	s string
	p *int
	sl []int
	m map[string]int
	ch chan int
	fn func()
)

// i=0 f=0 ok=false s="" p=<nil> sl=nil m=nil ch=nil fn=nil
```

#### Declarations: `var`, `:=`, and shadowing (common bug source)

```go
var a int            // zero value
var b = 10           // type inferred
c := 20              // short declaration (function scope only)

// multiple assignment
x, y := 1, "hi"
_, _ = x, y
```

Notes:
- `:=` works only inside functions.
- In the same scope, `:=` reuses existing variables only if **at least one** name on the left is new.

**Shadowing** (creates a new variable with same name in an inner scope):

```go
err := do()
if err != nil {
	return err
}

// accidentally shadows outer err
if _, err := io.ReadAll(r); err != nil {
	return err
}
// outer err is unchanged here
```

Guideline: keep scopes small; be careful mixing `:=` and `=`.

#### Type inference

```go
x := 10        // int
y := 3.14      // float64
z := "hi"      // string
```

Rules/notes:
- `:=` works inside functions only.
- You can redeclare with `:=` in the same scope only if **at least one new variable** is introduced.

#### Type conversion (explicit only)

Go does **not** do implicit numeric casts.

```go
var a int32 = 10
var b int64 = int64(a)
var f float64 = float64(b)
_ = f
```

#### Constants and `iota`

- `const` values are compile-time constants.
- Untyped constants can be used flexibly until context gives them a type.

```go
package main

import "fmt"

const Pi = 3.1415926535

const (
	_ = iota
	KB = 1 << (10 * iota)
	MB
	GB
)

func main() {
	fmt.Println("Pi:", Pi)
	fmt.Println("KB:", KB, "MB:", MB, "GB:", GB)
}
```

#### Strings, bytes, and runes (common gotcha)

- `len(s)` returns **bytes**, not characters.
- Iterate over runes via `for range` on string.

```go
s := "héllo世"
fmt.Println(len(s)) // bytes

for i, r := range s {
	fmt.Printf("%d: %q (%d)\n", i, r, r)
}
```

#### Nil vs empty slice (important in APIs and JSON)

- `var s []int` is **nil** (no allocation)
- `s := []int{}` is **empty but non-nil**

Both have `len==0` and can be ranged over and appended to, but differ in:
- `s == nil` checks
- JSON marshaling in some cases (nil slice often becomes `null`, empty slice becomes `[]`)

```go
var a []int
b := []int{}
fmt.Println(a == nil, len(a)) // true 0
fmt.Println(b == nil, len(b)) // false 0
```

#### `new` vs `make`

- `new(T)` allocates zeroed `T` and returns `*T`.
- `make` initializes **slices, maps, channels** (and returns the value, not pointer).

```go
p := new(int)      // *int, value 0
sl := make([]int, 3)
m := make(map[string]int)
ch := make(chan int, 1)
_, _, _, _ = p, sl, m, ch
```

#### Composite literals (struct/map/slice)

```go
type Point struct{ X, Y int }
p := Point{X: 1, Y: 2}

sl := []string{"a", "b"}
m := map[string]int{"a": 1}
_, _, _ = p, sl, m
```

#### Comparability (what can be used with `==`?)

- Comparable: numbers, bool, string, pointers, channels, interfaces (with comparable dynamic values), arrays (if element comparable), structs (if all fields comparable)
- Not comparable: slices, maps, functions

```go
type A struct{ X int }
fmt.Println(A{1} == A{1}) // true

// []int{1} == []int{1} // compile error: slice not comparable
```

---

### 1.2 Control Flow

#### `if` (with short statement)

```go
if v := 2 + 2; v == 4 {
	fmt.Println("math works")
}
```

#### `switch` (expression switch)

- No need for `break` (implicit).
- `fallthrough` exists but is rarely needed.

```go
switch n := 7; {
case n%2 == 0:
	fmt.Println("even")
case n%3 == 0:
	fmt.Println("divisible by 3")
default:
	fmt.Println("other")
}
```

#### Type switch

```go
func describe(x any) {
	switch v := x.(type) {
	case nil:
		fmt.Println("nil")
	case int:
		fmt.Println("int", v)
	case string:
		fmt.Println("string", v)
	default:
		fmt.Printf("type %T value %v\n", v, v)
	}
}
```

#### `for` (only loop)

```go
for i := 0; i < 3; i++ {
	fmt.Println(i)
}

for x < 10 {
	x++
}

for {
	if done {
		break
	}
}
```

#### `range` (arrays, slices, maps, strings, channels)

```go
for i, v := range []int{10, 20} {
	fmt.Println(i, v)
}

m := map[string]int{"a": 1}
for k, v := range m {
	fmt.Println(k, v)
}
```

**Range variable address pitfall**:

```go
type User struct{ Name string }
users := []User{{"a"}, {"b"}}
ptrs := []*User{}
for _, u := range users {
	// &u points to the same loop variable each iteration
	ptrs = append(ptrs, &u)
}
```

Fix:

```go
for i := range users {
	ptrs = append(ptrs, &users[i])
}
```

#### `defer` (LIFO; extremely important)

- Deferred calls execute when function returns (normal return or panic).
- Useful for closing resources, unlocking mutexes, timing.

```go
func f() {
	defer fmt.Println("third")
	defer fmt.Println("second")
	fmt.Println("first")
}
// output: first, second, third
```

**Defer evaluates arguments immediately** (but runs the call later):

```go
x := 1
defer fmt.Println("x =", x) // captures 1 now
x = 2
```

**Defer + named returns** can modify the returned value:

```go
func incr() (x int) {
	defer func() { x++ }()
	return 41
}
// returns 42
```

#### `panic` vs `recover`

- Use `panic` for programmer errors/invariants, not standard control flow.
- `recover` works only in deferred funcs on same goroutine.

```go
func safeRun(fn func()) (panicked bool) {
	defer func() {
		if r := recover(); r != nil {
			panicked = true
		}
	}()
	fn()
	return false
}
```

**Interview prompt**: when would you use `panic`?
- For impossible states, corruption, violations of assumptions, `init` failures in libraries (sparingly).
- Not for I/O errors, user input, network failures—return `error`.

---

### 1.3 Functions

#### Multiple return values

```go
func div(a, b int) (int, error) {
	if b == 0 {
		return 0, fmt.Errorf("divide by zero")
	}
	return a / b, nil
}
```

#### Named return values

```go
func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}
```

Use named returns primarily when they improve clarity (or for small helpers).

#### Variadic functions

```go
func sum(nums ...int) int {
	total := 0
	for _, n := range nums {
		total += n
	}
	return total
}

vals := []int{1, 2, 3}
fmt.Println(sum(vals...))
```

#### Closures (capturing variables)

```go
func makeCounter() func() int {
	n := 0
	return func() int {
		n++
		return n
	}
}
```

**Common gotcha (closure + loop variable)**:

```go
for i := 0; i < 3; i++ {
	go func() { fmt.Println(i) }() // may print 3,3,3 (timing)
}
```

Fix by capturing a new variable:

```go
for i := 0; i < 3; i++ {
	i := i
	go func() { fmt.Println(i) }()
}
```

---

## 2) Data Structures in Go

### 2.1 Arrays vs Slices (Very Important)

#### Arrays

- Fixed length, value type.
- `[3]int` and `[4]int` are different types.

```go
var a [3]int
b := [3]int{1, 2, 3}
_ = a
_ = b
```

#### Slices

Slice header contains:
- pointer to underlying array
- length
- capacity

```go
s := []int{1, 2, 3, 4}
sub := s[:2]      // len=2 cap=4 shares backing array
fmt.Println(sub)  // [1 2]
```

#### `make([]T, len, cap)` semantics

```go
s := make([]int, 3)    // len=3 cap=3, elements are zeroed
t := make([]int, 0, 3) // len=0 cap=3, empty slice with capacity
fmt.Println(s, len(s), cap(s))
fmt.Println(t, len(t), cap(t))
```

#### `append` and reallocation

- If `len+new <= cap`, append happens in-place.
- If not, a new underlying array is allocated and old data is copied.

```go
s := make([]int, 0, 2)
s = append(s, 1, 2)
t := append(s, 3) // likely realloc

s[0] = 100
fmt.Println("s:", s)
fmt.Println("t:", t)
```

#### Slice growth behavior (high-level)

When capacity is exceeded, `append` allocates a new underlying array and copies elements. Growth rate is **not a language guarantee** and can change across Go versions and sizes. The safe assumptions:
- `append` *may* reallocate
- reallocation breaks sharing with other slices referencing the old backing array

#### Full slice expression (control capacity to prevent overwrites)

Use `a := base[:len:cap]` to explicitly set capacity of a derived slice.

```go
base := []int{1, 2, 3, 4}
a := base[:2:2]     // len=2 cap=2
b := append(a, 99)  // must allocate new backing array now
fmt.Println("base:", base) // unchanged
fmt.Println("a:", a, "b:", b)
```

#### Slice sharing bugs

```go
base := []int{1, 2, 3, 4}
a := base[:2]           // cap includes rest
b := append(a, 99, 100) // may overwrite base[2], base[3]
fmt.Println(base, a, b)
```

**Safe copy**:

```go
aCopy := append([]int(nil), base[:2]...)
_ = aCopy
```

#### Copying slices

```go
dst := make([]int, len(src))
copy(dst, src)
```

---

### 2.2 Maps

#### Basics

```go
m := map[string]int{"a": 1}
v := m["a"]
v2, ok := m["missing"] // v2=0 ok=false
delete(m, "a")
```

#### Nil map behavior (important)

- Reads are OK, writes panic.

```go
var m map[string]int
fmt.Println(m["x"]) // 0
// m["x"] = 1       // panic
m = make(map[string]int)
m["x"] = 1
```

#### Concurrency issues

- Built-in maps are **not safe** for concurrent writes (or read+write).
- Use `sync.RWMutex` or `sync.Map`.

```go
type SafeMap struct {
	mu sync.RWMutex
	m  map[string]int
}

func (s *SafeMap) Get(k string) (int, bool) {
	s.mu.RLock()
	defer s.mu.RUnlock()
	v, ok := s.m[k]
	return v, ok
}
```

#### Map keys (what is allowed?)

Map keys must be **comparable** (can be used with `==`): numbers, bool, string, pointers, channels, interfaces (with comparable dynamic values), and structs/arrays whose fields/elements are comparable.

Not allowed: slices, maps, functions.

```go
type Key struct{ A int; B string }
m := map[Key]int{{1, "x"}: 10}
_ = m
```

#### Map as a set

```go
set := map[string]struct{}{}
set["a"] = struct{}{}
_, ok := set["a"] // membership test
```

#### `sync.Map` (when it fits)

Use `sync.Map` when you have many goroutines mostly reading a stable key set with occasional writes. Otherwise, `map + RWMutex` is usually clearer and often faster.

#### Map iteration notes

- Iteration order is randomized (don’t rely on it).
- Never take address of loop variables expecting distinct addresses.

---

### 2.3 Structs

#### Value vs pointer receivers

```go
type User struct {
	Name string
}

func (u User) Greet() string { // copies User (small here)
	return "hi " + u.Name
}

func (u *User) Rename(name string) {
	u.Name = name
}
```

Guideline:
- Use pointer receiver if method mutates receiver or receiver is large.
- Keep receiver type consistent across methods.

#### Method sets (interview favorite)

- `T` has methods with receiver `T`
- `*T` has methods with receiver `T` and `*T`

This matters when assigning to interfaces.

#### Struct embedding (composition)

```go
type Base struct{ ID int }
func (b *Base) SetID(id int) { b.ID = id }

type User struct {
	Base
	Name string
}

u := User{Name: "A"}
u.SetID(10)      // promoted
fmt.Println(u.ID)
```

---

## 3) Pointers & Memory Model

### 3.1 Pointers

```go
x := 10
p := &x
*p = 20
fmt.Println(x) // 20
```

Key points:
- Go has pointers but **no pointer arithmetic**.
- Prefer values unless you need mutation / avoid copies / represent optional.

### 3.2 Stack vs Heap, Escape Analysis

Go compiler decides whether variables live on stack or heap.

Common escape reasons:
- returning address of local
- storing pointer in heap-allocated structure
- capturing variable by closure that outlives stack frame

You can inspect with:

```bash
go test -gcflags='-m' ./...
```

#### Example: why something escapes

```go
func makePtr() *int {
	x := 10
	return &x // x must live beyond makePtr(), so it escapes to heap
}
```

### 3.3 Garbage Collection (high-level)

- Concurrent mark-and-sweep GC.
- Too many allocations → more GC pressure → latency/throughput issues.

Practical tips:
- preallocate slices/maps when possible
- reuse buffers (`bytes.Buffer`, `sync.Pool`) if proven hot
- avoid converting `[]byte`↔`string` repeatedly in hot paths

### 3.4 Interview: large struct by pointer vs value

Why pointers can be better:
- copying a huge struct is expensive (CPU/cache)
- pointer pass cost is small

Trade-offs:
- pointers can increase heap escapes / aliasing
- value semantics can be safer (immutability-like)

Rule of thumb:
- small structs: pass by value
- large structs or mutations: pointer
- measure performance in hot paths

---

## 4) Interfaces (Very Important)

### 4.1 Implicit implementation

```go
type Reader interface {
	Read(p []byte) (int, error)
}
```

Any type with `Read([]byte) (int, error)` implements `Reader` automatically.

### 4.2 Empty interface: `interface{}` / `any`

```go
var x any = 10
```

Use cases:
- generic containers (pre-generics era)
- decode JSON into arbitrary structure (but prefer typed structs)
- APIs that truly accept any type

Prefer:
- typed interfaces (small method sets)
- generics (when appropriate)

### 4.3 Type assertions

```go
v, ok := x.(int)
if !ok {
	// not int
}
```

### 4.4 Type switches

```go
switch v := x.(type) {
case int:
	_ = v
case string:
	_ = v
default:
}
```

### 4.5 Interface nil pitfalls (classic interview question)

An interface is a pair: **(dynamic type, dynamic value)**.

```go
type MyErr struct{}
func (*MyErr) Error() string { return "boom" }

func f() error {
	var e *MyErr = nil
	return e // interface holds (type=*MyErr, value=nil) => interface != nil
}

func main() {
	err := f()
	fmt.Println(err == nil) // false
}
```

Fix:
- return `nil` explicitly when you mean no error

```go
func g() error {
	var e *MyErr = nil
	if e == nil {
		return nil
	}
	return e
}
```

### 4.6 Designing good interfaces (real-world)

Principles:
- keep interfaces **small** (1–3 methods)
- define interfaces **where they are consumed**, not where produced
- accept interfaces, return concrete types (common guideline)

Example:

```go
type Store interface {
	GetUser(ctx context.Context, id string) (User, error)
}
```

### 4.7 Method sets with interfaces (concrete example)

```go
type Greeter interface{ Greet() string }

type Person struct{ Name string }
func (p Person) Greet() string { return "hi " + p.Name }

type Robot struct{ ID string }
func (r *Robot) Greet() string { return "beep " + r.ID }

func say(g Greeter) { fmt.Println(g.Greet()) }

func main() {
	say(Person{"A"})   // ok: Person has value receiver method

	// say(Robot{"R"}) // not ok: Robot (value) doesn't have (*Robot).Greet
	say(&Robot{"R"})   // ok
}
```

Interview angle: explain why the `Robot` value doesn’t implement the interface.

---

## 5) Concurrency (Critical)

### 5.1 Goroutines

- lightweight, scheduled by Go runtime (M:N scheduler).
- don’t assume order; always synchronize.

```go
go func() {
	// work
}()
```

### 5.1.1 Scheduler model (high level, interview-friendly)

- **G**: goroutine
- **M**: OS thread
- **P**: processor (scheduling resource)

Go runtime schedules many goroutines (G) onto a smaller set of OS threads (M) using P resources (M:N scheduling).

### 5.1.2 Goroutine leaks (very important in production)

Leaks happen when goroutines block forever on:
- channel receives/sends with no counterpart
- `select` without cancellation case
- waiting on a `WaitGroup` that never completes

Rule: every goroutine should have a **clear exit condition** (`context`, done channel, closed input channel).

### 5.2 Channels

#### Unbuffered vs buffered

- Unbuffered: send blocks until receiver is ready.
- Buffered: send blocks only when buffer is full.

```go
ch := make(chan int)      // unbuffered
buf := make(chan int, 10) // buffered
_, _ = ch, buf
```

#### Closing channels

- Only close from sender side.
- Closing signals “no more values”.
- Receiving from closed channel yields zero value and `ok=false`.

```go
close(ch)
v, ok := <-ch
_ = v
_ = ok
```

Important channel rules:
- Sending on a **closed** channel panics.
- Closing a channel twice panics.
- Receiving from a **nil** channel blocks forever.
- Sending to a **nil** channel blocks forever.

```go
var ch chan int // nil
// ch <- 1  // blocks forever
// <-ch    // blocks forever
```

#### `range` over channel

```go
for v := range ch {
	fmt.Println(v)
}
```

### 5.3 `select`

#### Multiplexing

```go
select {
case v := <-ch1:
	_ = v
case v := <-ch2:
	_ = v
}
```

#### Timeouts and cancellation

```go
select {
case v := <-ch:
	_ = v
case <-time.After(500 * time.Millisecond):
	fmt.Println("timeout")
}
```

#### Non-blocking receive/send

```go
select {
case v := <-ch:
	_ = v
default:
	// nothing ready
}
```

#### Disabling select cases with nil channels (advanced)

You can set a channel variable to `nil` to disable its `select` case dynamically.

```go
var in <-chan int = ch1
if disable {
	in = nil // case on in will never fire
}

select {
case v := <-in:
	_ = v
case <-time.After(time.Second):
}
```

### 5.4 `sync` primitives

#### `sync.Mutex` / `sync.RWMutex`

```go
type Counter struct {
	mu sync.Mutex
	n  int
}

func (c *Counter) Inc() {
	c.mu.Lock()
	defer c.mu.Unlock()
	c.n++
}

func (c *Counter) Value() int {
	c.mu.Lock()
	defer c.mu.Unlock()
	return c.n
}
```

#### `sync.WaitGroup`

```go
var wg sync.WaitGroup

for i := 0; i < 3; i++ {
	wg.Add(1)
	go func() {
		defer wg.Done()
		// work
	}()
}

wg.Wait()
```

#### `sync.Once`

```go
var once sync.Once
once.Do(func() {
	// init once
})
```

#### `sync.Cond` (advanced)

Use when you need complex waiting conditions that channels don’t model cleanly.

#### `sync/atomic` (when you need lock-free counters)

For simple counters or flags, atomics can avoid lock overhead. Use them carefully; atomic operations don’t automatically make multi-step invariants safe.

```go
var n atomic.Int64
n.Add(1)
fmt.Println(n.Load())
```

### 5.5 Common concurrency patterns (with examples)

#### Worker pool

```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, jobs <-chan int, results chan<- int, wg *sync.WaitGroup) {
	defer wg.Done()
	for j := range jobs {
		results <- j * j
	}
}

func main() {
	jobs := make(chan int, 100)
	results := make(chan int, 100)

	var wg sync.WaitGroup
	for w := 0; w < 4; w++ {
		wg.Add(1)
		go worker(w, jobs, results, &wg)
	}

	for j := 1; j <= 10; j++ {
		jobs <- j
	}
	close(jobs)

	wg.Wait()
	close(results)

	for r := range results {
		fmt.Println(r)
	}
}
```

#### Fan-out / fan-in

- Fan-out: distribute work to multiple goroutines.
- Fan-in: merge multiple channels into one.

```go
func fanIn(chs ...<-chan int) <-chan int {
	out := make(chan int)
	var wg sync.WaitGroup
	wg.Add(len(chs))

	for _, ch := range chs {
		go func(c <-chan int) {
			defer wg.Done()
			for v := range c {
				out <- v
			}
		}(ch)
	}

	go func() {
		wg.Wait()
		close(out)
	}()

	return out
}
```

#### Producer-consumer with backpressure

Backpressure is naturally modeled with bounded buffered channels.

```go
queue := make(chan Task, 100) // bounded capacity
```

#### Context-aware worker (avoid leaks)

```go
func worker(ctx context.Context, jobs <-chan Job) error {
	for {
		select {
		case <-ctx.Done():
			return ctx.Err()
		case j, ok := <-jobs:
			if !ok {
				return nil
			}
			_ = j // do work
		}
	}
}
```

#### Rate limiting (ticker-based)

```go
limiter := time.NewTicker(100 * time.Millisecond)
defer limiter.Stop()

for req := range requests {
	<-limiter.C
	handle(req)
}
```

#### Graceful shutdown (signal + context)

Use `context` to signal goroutines to stop, and ensure all exit.

#### Deadlock example (interview favorite)

```go
func main() {
	ch := make(chan int)
	ch <- 1          // deadlocks: no receiver
	fmt.Println(<-ch)
}
```

Fix by using a goroutine or buffered channel:

```go
ch := make(chan int, 1)
ch <- 1
fmt.Println(<-ch)
```

### 5.6 Race conditions and `-race`

- Race: unsynchronized shared memory access with at least one write.
- Detect with:

```bash
go test -race ./...
```

### 5.7 Mutex vs channel (interview)

- Use **mutex** when protecting shared state is simplest.
- Use **channels** when you can design ownership/communication cleanly (pipelines, worker pools).

Common mistake: “channels replace mutexes.” In reality, both are tools—choose based on design clarity and performance needs.

---

## 6) `context` Package (Very Important for Backend)

### 6.1 Core rules

- Pass `ctx` as the **first** param: `func Foo(ctx context.Context, ...)`
- Do not store contexts in structs long-term.
- Use `context.Background()` for root contexts (main, tests).
- Prefer request-scoped contexts from `*http.Request`.

### 6.2 Cancel / timeout / deadline

```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

ctx2, cancel2 := context.WithTimeout(context.Background(), 500*time.Millisecond)
defer cancel2()
```

### 6.2.1 Context values (`WithValue`) (use carefully)

`context.WithValue` is for **request-scoped metadata** (request IDs, auth claims) crossing API boundaries.

Avoid using it as a “bag of optional params” in business logic; it hides dependencies and makes code harder to test.

### 6.3 Pattern: select on `ctx.Done()`

```go
func doWork(ctx context.Context) error {
	select {
	case <-time.After(2 * time.Second):
		return nil
	case <-ctx.Done():
		return ctx.Err()
	}
}
```

### 6.4 Interview questions

- What does `ctx.Done()` represent? (a channel closed on cancellation)
- Should you call `cancel()`? (yes, to release resources)
- Where should you set timeouts? (at boundaries: handlers, client calls)

---

## 7) HTTP & Networking

### 7.1 `net/http` fundamentals

- `http.Handler` interface:

```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```

- `http.HandlerFunc` adapts a function to handler.

```go
func hello(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "hello")
}
```

### 7.2 Multiplexer and routes

```go
mux := http.NewServeMux()
mux.HandleFunc("/hello", hello)

srv := &http.Server{
	Addr:    ":8080",
	Handler: mux,
}
log.Fatal(srv.ListenAndServe())
```

### 7.3 Middleware pattern

```go
func logging(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		next.ServeHTTP(w, r)
		log.Printf("%s %s %s", r.Method, r.URL.Path, time.Since(start))
	})
}
```

Compose:

```go
handler := logging(mux)
```

### 7.4 Request lifecycle & concurrency

- Each request is handled concurrently (goroutines).
- Avoid shared mutable state in handlers without protection.
- Use server timeouts to prevent slowloris and resource exhaustion.

Recommended server timeouts:

```go
srv := &http.Server{
	Addr:         ":8080",
	Handler:      handler,
	ReadTimeout:  5 * time.Second,
	WriteTimeout: 10 * time.Second,
	IdleTimeout:  120 * time.Second,
}
```

### 7.5 JSON (`encoding/json`)

#### Decode safely

```go
type CreateUserRequest struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

dec := json.NewDecoder(r.Body)
dec.DisallowUnknownFields()
var req CreateUserRequest
if err := dec.Decode(&req); err != nil {
	http.Error(w, "invalid json", http.StatusBadRequest)
	return
}
```

#### Encode response

```go
w.Header().Set("Content-Type", "application/json")
json.NewEncoder(w).Encode(map[string]any{"ok": true})
```

### 7.6 HTTP client (timeouts + context)

```go
req, _ := http.NewRequestWithContext(ctx, http.MethodGet, "https://example.com", nil)
client := &http.Client{Timeout: 5 * time.Second}
resp, err := client.Do(req)
if err != nil { /* handle */ }
defer resp.Body.Close()
```

### 7.6.1 Client reuse (important production advice)

- Reuse an `http.Client` (or a small set) across requests to benefit from connection pooling and keep-alives.
- Avoid creating a new `http.Client`/`Transport` per request unless you know why.

### 7.6.2 Transport tuning (advanced)

`http.Transport` controls pooling, TLS, timeouts, HTTP/2 behavior. Tune only with metrics and understanding; defaults are good for many services.

### 7.7 Common backend concerns

- **Validation**: validate inputs early, return clear errors.
- **Logging**: structured logs; include request ID.
- **Metrics**: request latency, error rate, saturation.
- **Limits**: request body size (`http.MaxBytesReader`), timeouts.
- **Streaming**: use `io.Copy`, avoid loading huge bodies into memory.

### 7.8 Request body limits (defense)

```go
r.Body = http.MaxBytesReader(w, r.Body, 1<<20) // 1 MB
```

### 7.9 Consistent error responses

For APIs, use a consistent JSON error schema so clients can programmatically react:

```json
{ "error": { "code": "invalid_request", "message": "name is required" } }
```

---

## 8) Error Handling Philosophy

### 8.1 Idiomatic pattern

```go
v, err := doThing()
if err != nil {
	return fmt.Errorf("doThing: %w", err)
}
```

### 8.2 Wrapping errors (`%w`) and inspection

```go
if errors.Is(err, os.ErrNotExist) { /* ... */ }

var pe *os.PathError
if errors.As(err, &pe) { /* ... */ }
```

### 8.3 Sentinel errors vs custom types

- Sentinel errors for simple “category” checks.
- Custom types for structured information.

```go
var ErrNotFound = errors.New("not found")

type HTTPError struct {
	Code int
	Msg  string
}
func (e *HTTPError) Error() string { return fmt.Sprintf("%d %s", e.Code, e.Msg) }
```

### 8.5 Multiple errors: `errors.Join` (Go 1.20+)

When multiple independent operations fail (e.g., closing multiple resources), you can join errors:

```go
err := errors.Join(err1, err2)
if err != nil {
	return err
}
```

### 8.6 `defer` + `Close()` errors (practical)

In real code you often want to preserve a close error without hiding earlier errors:
- If the main operation failed, return that error (maybe log close error).
- If main succeeded but close failed, return close error.

### 8.7 Why explicit errors over exceptions? (interview)

- Clear control flow and visibility at call sites.
- Easier to reason about and test.
- No hidden stack-unwinding; errors are values.
- Forces deliberate handling and propagation with context.

---

## 9) Testing

### 9.1 Unit tests (`testing`)

```go
func Add(a, b int) int { return a + b }

func TestAdd(t *testing.T) {
	if got := Add(2, 3); got != 5 {
		t.Fatalf("got %d want %d", got, 5)
	}
}
```

### 9.2 Table-driven tests

```go
func TestAdd(t *testing.T) {
	tests := []struct{
		name string
		a, b int
		want int
	}{
		{"basic", 1, 2, 3},
		{"zero", 0, 0, 0},
		{"neg", -1, 2, 1},
	}
	for _, tc := range tests {
		t.Run(tc.name, func(t *testing.T) {
			if got := Add(tc.a, tc.b); got != tc.want {
				t.Fatalf("got %d want %d", got, tc.want)
			}
		})
	}
}
```

### 9.3 Benchmarks (`-bench`, `-benchmem`)

```go
func BenchmarkAdd(b *testing.B) {
	for i := 0; i < b.N; i++ {
		_ = Add(1, 2)
	}
}
```

Run:

```bash
go test -bench=. -benchmem ./...
```

### 9.4 HTTP tests (`httptest`)

```go
func TestHello(t *testing.T) {
	req := httptest.NewRequest(http.MethodGet, "/hello", nil)
	rec := httptest.NewRecorder()

	hello(rec, req)

	if rec.Code != http.StatusOK {
		t.Fatalf("code=%d", rec.Code)
	}
}
```

### 9.5 Mocking with interfaces

Define interface at consumer side and inject fake implementation in tests.

### 9.6 Subtests and parallel tests

```go
t.Run("case", func(t *testing.T) {
	t.Parallel()
	// ...
})
```

Only use `t.Parallel()` when tests are truly independent (no shared global state).

### 9.7 Fuzz testing (great for parsers/validators)

Go supports fuzzing via `testing` (Go 1.18+). Use it to discover crashes and edge cases.

### 9.8 Integration testing tips

- Use `httptest.Server` for HTTP integration tests.
- Use `t.TempDir()` for filesystem isolation.
- Prefer deterministic tests: control time, randomness, and external dependencies.

---

## 10) Go Modules & Project Structure

### 10.1 Go modules

- `go mod init`, `go mod tidy`, `go get`
- Keep dependencies minimal; prefer standard library.

### 10.2 Common structure

```text
cmd/
  app/
    main.go
internal/
  httpapi/
  service/
  repo/
pkg/              (optional, for reusable libraries)
```

Guidelines:
- `internal/` packages are private to module.
- keep `main` thin; business logic in packages.

### 10.3 `go.sum`, semantic versions, and `replace`

- `go.sum` stores checksums of module versions (integrity/reproducibility).
- Major versions `v2+` often appear in import path: `example.com/pkg/v2`.
- `replace` can point a dependency to a fork/local path (use carefully; keep it out of released builds).

### 10.4 Workspaces (`go work`) (multi-module development)

Workspaces help develop multiple modules together without scattering `replace` directives.

---

## 11) Performance & Profiling (Advanced)

### 11.1 pprof

Enable pprof endpoints:

```go
import _ "net/http/pprof"

go func() {
	log.Println(http.ListenAndServe("localhost:6060", nil))
}()
```

Profile CPU:

```bash
go tool pprof http://localhost:6060/debug/pprof/profile
```

### 11.2 Allocation reduction patterns

- preallocate: `make([]T, 0, n)`
- avoid repeated `fmt.Sprintf` in hot loops
- reuse buffers where it matters
- use `strings.Builder` for building strings

```go
var b strings.Builder
for i := 0; i < 10; i++ {
	b.WriteString("x")
}
s := b.String()
_ = s
```

### 11.3 `runtime/trace` (deeper performance debugging)

Tracing can reveal goroutine scheduling, blocking, and GC events. Use it when pprof isn’t enough.

---

## 12) System Design with Go (Backend Interviews)

### 12.1 High concurrency server

Key areas:
- timeouts everywhere (server/client)
- bounded queues to prevent overload
- connection pooling (DB, outbound HTTP)
- metrics, tracing, structured logging
- graceful shutdown

### 12.2 Handling very high QPS (e.g., “1M requests”)

Typical answer outline:
- scale horizontally behind LB
- optimize per-request CPU/allocations
- cache hot reads
- batch expensive operations where possible
- use async processing for non-critical tasks
- apply rate limits/backpressure
- observe with SLOs and dashboards

### 12.3 Rate limiting

- local per-instance token bucket (simple)
- distributed limiter (Redis) for global quotas

### 12.4 Caching

- cache-aside
- write-through / write-back (careful)
- TTLs and invalidation strategy (hard part)

### 12.5 Graceful shutdown (full example)

```go
package main

import (
	"context"
	"log"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(http.StatusOK)
		w.Write([]byte("ok"))
	})

	srv := &http.Server{
		Addr:         ":8080",
		Handler:      mux,
		ReadTimeout:  5 * time.Second,
		WriteTimeout: 10 * time.Second,
		IdleTimeout:  120 * time.Second,
	}

	go func() {
		log.Println("listening on", srv.Addr)
		if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
			log.Fatal(err)
		}
	}()

	stop := make(chan os.Signal, 1)
	signal.Notify(stop, os.Interrupt, syscall.SIGTERM)
	<-stop

	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	if err := srv.Shutdown(ctx); err != nil {
		log.Fatal(err)
	}
	log.Println("shutdown complete")
}
```

---

## Additional High-Value Go Topics (Recommended)

### A) Packages, visibility, and init

- Identifiers starting with uppercase are exported: `MyType`, `DoThing`.
- `init()` runs automatically when package is imported (avoid heavy work in `init`).

### B) `defer` cost considerations

`defer` is extremely useful, but in very hot loops it can be expensive. In tight loops, consider manual cleanup. Most application code should still prefer clarity.

### C) `io` patterns (important in Go)

- `io.Reader`, `io.Writer`, `io.ReadCloser`
- streaming is preferred for large data

```go
data, err := io.ReadAll(r.Body) // avoid for huge bodies
```

Prefer bounded reads when needed.

### D) Time handling

- Always store times in UTC in databases.
- Be careful with timezones and formatting.
- Use `time.Now().UTC()` for consistent timestamps.

### E) Generics (Go 1.18+)

Use generics when they improve correctness and reduce duplication, without hurting readability.

```go
func Map[T any, R any](in []T, f func(T) R) []R {
	out := make([]R, len(in))
	for i, v := range in {
		out[i] = f(v)
	}
	return out
}
```

#### Constraints example

```go
func Contains[T comparable](s []T, x T) bool {
	for _, v := range s {
		if v == x {
			return true
		}
	}
	return false
}
```

Use generics when it improves correctness and reduces duplication—avoid over-generalizing simple code.

### F) Embedding files with `go:embed` (assets/templates)

```go
//go:embed templates/*
var templatesFS embed.FS
```

### G) `go generate` (code generation hooks)

`go generate` runs commands specified in `//go:generate ...` directives. Useful for mocks, `stringer`, protobufs, etc.

### H) `database/sql` quick primer (backend staple)

Key facts:
- `sql.DB` is a **pool handle**, safe for concurrent use. Create once and reuse.
- Always use **context-aware** methods (`QueryContext`, `ExecContext`, etc.).
- Close rows (`rows.Close()`) and check `rows.Err()`.
- Configure pool limits (`SetMaxOpenConns`, `SetMaxIdleConns`, `SetConnMaxLifetime`) based on DB capacity.

```go
db, err := sql.Open("postgres", dsn)
if err != nil { /* handle */ }

db.SetMaxOpenConns(25)
db.SetMaxIdleConns(25)
db.SetConnMaxLifetime(5 * time.Minute)

ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
defer cancel()

row := db.QueryRowContext(ctx, "select id, name from users where id=$1", id)
var u User
if err := row.Scan(&u.ID, &u.Name); err != nil {
	if errors.Is(err, sql.ErrNoRows) {
		// not found
	}
	// handle
}
```

### I) `io` and buffering patterns (practical)

- Wrap readers with `bufio.Reader` for many small reads.
- Use `io.LimitReader` to avoid unbounded reads.
- Use `io.Copy` / `io.CopyBuffer` for streaming efficiently.

```go
r := io.LimitReader(src, 1<<20) // 1MB max
_, err := io.Copy(dst, r)
_ = err
```

### J) Common pitfalls

- taking address of range variable
- interface nil confusion
- map concurrent writes
- unbounded goroutine leaks (never-ending goroutines)
- using `time.After` repeatedly in loops (can allocate); consider `time.NewTimer`

---

## Exercises (with prompts)

### 1) Core

- **Exercise 1**: write `ParseInt(s string) (int, error)` using `strconv.Atoi`; wrap errors with context.
- **Exercise 2**: implement `ReverseRunes(s string) string` (handle unicode correctly).
- **Exercise 3**: given a slice, return a new slice without modifying original (must avoid sharing bug).

### 2) Data structures

- **Exercise 4**: implement a frequency counter for words in a text file (`map[string]int`).
- **Exercise 5**: implement `DedupInts([]int) []int` preserving order.

### 3) Concurrency

- **Exercise 6 (worker pool)**: given `[]int`, compute squares concurrently using N workers and preserve output order.
- **Exercise 7 (fan-in)**: merge results from 3 goroutines that each produce numbers; stop when context cancels.
- **Exercise 8 (rate limiter)**: build a limiter that allows 5 requests/sec with burst 10; simulate 50 requests.

### 4) HTTP

- **Exercise 9**: write a JSON API:
  - `POST /users` with `{name, age}`
  - validate inputs
  - return created user with ID
  - add request logging middleware
- **Exercise 10**: implement graceful shutdown + background worker that stops on context cancel.

### 5) Testing

- **Exercise 11**: write table-driven tests for `DedupInts`.
- **Exercise 12**: benchmark two implementations of string concatenation (`+` vs `strings.Builder`).

### 6) Bonus (advanced)

- **Exercise 13**: implement an in-memory cache with TTL and a cleanup goroutine; ensure it shuts down cleanly with context.
- **Exercise 14**: implement a bounded queue with `sync.Cond` and compare it with a channel-based queue.
- **Exercise 15**: write a generic `Contains` and benchmark it against map-based membership for large datasets.

---

## Interview Questions (Practice)

### Language fundamentals

- What are zero values and why do they matter?
- Why doesn’t Go allow implicit numeric conversions?
- Explain `defer` execution order and how it interacts with named returns.

### Slices / maps

- Explain slice header: pointer/len/cap.
- When does `append` reallocate? What bugs can happen with slice sharing?
- Why can writing to a nil map panic but reading does not?
- Are maps safe for concurrent access? What options do you have?

### Pointers / memory

- What is escape analysis? How can you inspect it?
- When is passing a struct by pointer better than by value?

### Interfaces

- How is interface implementation done in Go?
- Explain type assertion vs type switch.
- Why can `err != nil` even if the underlying pointer inside interface is nil?

### Concurrency

- Difference between buffered and unbuffered channels (blocking semantics).
- What causes deadlocks? Give a minimal example.
- When to use mutex vs channels?
- How to prevent goroutine leaks?
- What does `-race` detect and what doesn’t it guarantee?

### Context / HTTP

- How do you propagate cancellation through layers?
- How do you set timeouts correctly in HTTP servers and clients?
- How does `net/http` handle concurrency?

### Error handling

- Explain `%w` and how `errors.Is/As` works.
- Sentinel errors vs custom error types: when to use which?

### Testing / performance

- What are table-driven tests and why do Go devs love them?
- How do you benchmark and profile a hot path?
- How do you reduce allocations?

---

## “Mini mock interview” drills (answer out loud)

1) You have a handler that calls an external API. Where do you apply timeouts and how do you propagate cancellation?
2) You see `fatal error: concurrent map writes`. Explain root cause and show two fixes.
3) Given `base := []int{1,2,3,4}`, why might `append(base[:2], 9, 9)` change `base`? Show safe alternatives.
4) Explain why `var err *MyErr = nil; return err` can produce `err != nil`.
5) When would you use `sync.RWMutex` vs `sync.Mutex`? When might RWMutex be slower?
6) How do you prevent goroutine leaks in fan-in/fan-out pipelines?
7) Show how to test an HTTP handler without binding to a real port.
8) How would you profile a Go service using pprof? What would you look at first?

