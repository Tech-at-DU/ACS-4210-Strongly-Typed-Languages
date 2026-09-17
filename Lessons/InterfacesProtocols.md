<!-- Run as a slideshow: reveal-md Lessons/InterfacesProtocols.md -w -->
# Interfaces & Protocols — Behavior Over Inheritance

⭐️ **GOAL:** Leave able to satisfy a small interface implicitly, predict pointer vs value method-set traps on interface assignment, use `io.Reader` / `io.Writer` (and `io.Copy`) as the default I/O protocol, and choose when **not** to invent an interface.

<!-- omit in toc -->
## ⏱ Agenda

- [[**5m**] Attendance &amp; Announcements](#5m-attendance--announcements)
- [[**15m**] ☀️ Warm Up](#15m-️-warm-up)
- [[**35m**] 📚 TT: Overview](#35m--tt-overview)
- [[**10m**] 🌴 Break](#10m--break)
- [[**20m**] 💻 Activity 1: Broken Store Satisfaction](#20m--activity-1-broken-store-satisfaction)
- [[**30m**] 💻 Activity 2: Build logpipe on io.Writer](#30m--activity-2-build-logpipe-on-iowriter)
- [[**5m**] Wrap Up](#5m-wrap-up)

<!-- > -->

<!-- omit in toc -->
## 🏆 Objectives

*By the end of this session, you'll be able to&hellip;*

1. **Explain** that a type satisfies an interface **implicitly** (no `implements` keyword).
2. **Predict** compile errors from **pointer vs value** method-set rules when assigning to an interface.
3. **Use** `io.Reader` / `io.Writer` (and `io.Copy`) as the canonical small-interface story.
4. **Distinguish** `any` (alias for `interface{}`) from a meaningful protocol; use comma-ok assertions / type switches safely.
5. **Apply** “accept interfaces, return structs” with a consumer-side fake — and name when **not** to introduce an interface.

<!-- > -->

## [**5m**] Attendance &amp; Announcements

Today is **protocols over inheritance**. Hire-bar Go asks “can you do this?” — not “what family tree are you on?”

If you already saw geometry `Shape` elsewhere in the course, treat that as **syntax you know** — not the default design story for production APIs. Depth for interfaces lives **here**.

Have a terminal ready with a **supported** Go (`go version` → **1.26.x or 1.27.x** as of Sep 2026).

<!-- > -->

## [**15m**] ☀️ Warm Up

Interfaces are **protocols**: a set of method signatures. A type **satisfies** the protocol when its method set contains those methods. There is **no** `implements` line.

**Warm-up prompt (think → chat → share):** In one sentence each, answer:

1. What gets lost if you describe a rule only as vague steps (no jargon allowed)?
2. Where would that loss show up in a real repo — compile fail on interface assign, untestable `*os.File`-only API, or a fat DAO nobody needed?

Whiteboard four protocol cards if chat stays quiet (read only; do not debate):

| Card | Rule |
| --- | --- |
| A | No `implements`. Satisfaction = method-set **containment**. |
| B | Method set of `T` ≠ method set of `*T`. Interface assign will **not** auto-`&` for you. |
| C | Prefer **small** interfaces. `io.Reader` is one method — on purpose. |
| D | Put interfaces with the **consumer**. Constructors usually **return concrete structs**. |

> **PROTIP:** Say it once out loud: interfaces ask “can you do this?” — not “what are you a subclass of?”

<!-- > -->

## [**35m**] 📚 TT: Overview

**Next action:** Protocols → live `Greeter` → `io` → method-set trap → `any` / asserts → accept-interfaces / return-structs → when **not** to interface.  
**Done when:** You can restate implicit satisfaction, the pointer trap, why `io.Writer` is safer than `*os.File` in APIs, and one reason to skip inventing an interface.

### 1. Protocols, not inheritance (~5m)

Java / C# habits whisper: declare an interface, write `implements`, build a hierarchy, interface *everything*.

Go is quieter. An **interface** names a **protocol**. A type **satisfies** it when it has those methods. The compiler checks containment at the **use** site.

That means:

- Types never have to mention the interface they satisfy.
- One type can satisfy many interfaces without listing them.
- You can invent a tiny interface **next to the code that needs it**, long after the concrete type was written.

> **ASK AUDIENCE:** Does a Go type need an `implements` line to satisfy an interface?

<details>
<summary>Answer</summary>

**No.** If the method set contains every required method, it satisfies — silently.

</details>

### 2. Live demo — Greeter never named by the types (~7m)

```bash
mkdir -p /tmp/greeterdemo && cd /tmp/greeterdemo
go mod init example.com/greeterdemo
```

```go
package main

import "fmt"

// Greeter is a tiny protocol: can you greet?
type Greeter interface {
	Greet() string
}

type Human struct{ Name string }

func (h Human) Greet() string { return "hello, " + h.Name }

type Robot struct{ ID int }

func (r Robot) Greet() string { return fmt.Sprintf("beep-%d", r.ID) }

func announce(g Greeter) { fmt.Println(g.Greet()) }

func main() {
	announce(Human{Name: "Alex"})
	announce(Robot{ID: 7})
}
```

```bash
go run .
```

**Say this out loud:** `Human` and `Robot` never mention `Greeter`. If you delete `Greet` from `Robot`, the error shows up at `announce(Robot{…})` — the **call site** — not on the struct.

**Also say:** The interesting protocol lives with the **caller** (`announce`), not with a fake `IGreeter` package invented up front.

### 3. `io.Reader` / `Writer` + `io.Copy` (~8m)

Open [pkg.go.dev/io](https://pkg.go.dev/io):

```go
type Reader interface {
	Read(p []byte) (n int, err error)
}

type Writer interface {
	Write(p []byte) (n int, err error)
}
```

That is the whole protocol. Files, buffers, network connections, string readers — same shapes.

```go
package main

import (
	"fmt"
	"io"
	"os"
	"strings"
)

func main() {
	r := strings.NewReader("hello from a Reader\n")
	n, err := io.Copy(os.Stdout, r)
	fmt.Fprintf(os.Stderr, "copied %d bytes, err=%v\n", n, err)
}
```

**Narrate:** `*strings.Reader` and `*os.File` never “implement Writer/Reader” in source — they just have `Read` / `Write`. `io.Copy` only asks for the protocol.

**SSG / hire-bar bridge:** If your function takes `*os.File`, tests need a real file. If it takes `io.Writer`, tests can use `bytes.Buffer` or a three-line fake.

**Composition flash:** Stdlib composes small interfaces — `io.ReadWriter` embeds `Reader` and `Writer`. Prefer that over inventing `MyReadWriteCloser`.

> **ASK AUDIENCE:** How many methods does `io.Reader` require?

<details>
<summary>Answer</summary>

**One:** `Read([]byte) (int, error)`.

</details>

### 4. Method sets and the pointer trap (~8m)

This is the compile error that shows up in interviews and in lab.

```go
package main

type Incrementer interface{ Inc() }

type Counter struct{ N int }

func (c *Counter) Inc() { c.N++ }

func main() {
	var c Counter
	c.Inc() // OK: addressable call sugar may become (&c).Inc()

	// FAIL: method set of Counter does NOT include Inc
	// var i Incrementer = c

	var i Incrementer = &c // OK: *Counter includes Inc
	i.Inc()
	_ = i
}
```

**Two rules people mix up:**

1. **Method call sugar:** if `c` is **addressable**, `c.Inc()` can become `(&c).Inc()` even with a pointer receiver.
2. **Interface assignment is stricter:** the value you assign must **already** have the method in its method set. Go will **not** invent `&c` to satisfy `Incrementer`.

| You have | Method set includes… | Assign to interface needing `Inc()`? |
| --- | --- | --- |
| `Counter` (`T`) | value-receiver methods on `T` | **No** if `Inc` is on `*T` |
| `*Counter` (`*T`) | value-receiver **and** pointer-receiver methods | **Yes** |

> **ASK AUDIENCE:** Will `var i Incrementer = c` compile when `Inc` has a pointer receiver?

<details>
<summary>Answer</summary>

**No.** Assign `&c` (or store a `*Counter`). Call-site sugar does not apply to interface assignment.

</details>

> **BE AWARE:** Once a value is **inside** an interface, you do not get free “take address and call pointer method” sugar through the interface slot. Put the right concrete type (`*Counter`) in from the start.

### 5. `any`, assertions, switches, nil flash (~5m)

Since **Go 1.18**, `any` is a predeclared alias for `interface{}`. Prefer `any` in new code.

```go
package main

import "fmt"

func describe(v any) string {
	if s, ok := v.(string); ok {
		return "string:" + s
	}
	switch x := v.(type) {
	case int:
		return fmt.Sprintf("int:%d", x)
	case nil:
		return "nil-interface"
	default:
		return fmt.Sprintf("other:%T", x)
	}
}

func main() {
	fmt.Println(describe("hi"))
	fmt.Println(describe(3))

	var p *int = nil
	var i any = p // not a nil interface — holds (*int, nil)
	fmt.Println(i == nil) // false — classic footgun
	fmt.Println(describe(i))
}
```

- `any` means “I gave up on a meaningful protocol.” Fine for unknown JSON bags — **not** your domain API.
- Prefer **comma-ok** assertions. Single-value assert panics on mismatch.
- **Nil interface flash:** a nil **pointer** stored in an interface is **not** equal to a nil interface.

### 6. Accept interfaces, return structs; fakes; when NOT (~7m)

Hire-bar paraphrase of [Code Review Comments — Interfaces](https://go.dev/wiki/CodeReviewComments#interfaces): put interfaces with the consumer; return concrete types from constructors.

- **Parameters:** ask for the **smallest** protocol you need (`io.Writer`, not `*os.File`).
- **Returns / constructors:** return **concrete** types (`*Mem`, `*Server`).
- **Define the interface next to the consumer**, not “for mocking” on the producer.

```go
package bill

import "time"

type Clock interface{ Now() time.Time }

type realClock struct{}

func (realClock) Now() time.Time { return time.Now() }

type Service struct{ clock Clock }

func NewService(c Clock) *Service {
	if c == nil {
		c = realClock{}
	}
	return &Service{clock: c}
}

func (s *Service) Stamp() string {
	return s.clock.Now().UTC().Format(time.RFC3339)
}
```

Test fake in `_test.go` (no mockgen):

```go
type fixedClock struct{ t time.Time }

func (f fixedClock) Now() time.Time { return f.t }

func TestStamp(t *testing.T) {
	want := time.Date(2026, 9, 8, 16, 0, 0, 0, time.UTC)
	s := NewService(fixedClock{t: want})
	if got := s.Stamp(); got != "2026-09-08T16:00:00Z" {
		t.Fatalf("got %q", got)
	}
}
```

**When *not* to introduce an interface**

Do **not**:

- Write `IStore` before you have **two** real call sites that need the protocol
- Return interfaces from constructors “for testability” when `*T` + consumer-side interface would do
- Grow a fat interface so every fake implements methods you never call
- Replace a fine concrete helper with `any` + type switches
- Port Java DAO / interface-everything into Go homework

**Exception:** returning an interface **is** OK when the concrete type is chosen at runtime (`hash.Hash`, `net.Conn`) or when the type *is* the protocol (`error`). Homework default: return `*T`.

> **ASK AUDIENCE:** You only need `Write`. Do you take `io.Writer`, `io.WriteCloser`, or `*os.File` “just in case”?

<details>
<summary>Answer</summary>

Take **`io.Writer`**. Widen only when you actually need `Close` (then `io.WriteCloser`). `*os.File` couples tests to the filesystem.

</details>

### Failure modes worth a grin

| What you see | What you probably did | Fix |
| --- | --- | --- |
| `X does not implement Y (Inc method has pointer receiver)` | Assigned `T` where `*T` is required | Assign `&x` / store `*T` |
| Works as `x.Inc()` but fails as `var y I = x` | Mixed up call sugar vs method set | Addressable sugar ≠ interface assign |
| Tests need real files / network | Took `*os.File` / concrete client | Accept `io.Writer` / small consumer interface |
| Every fake is huge | Fat producer-side interface | Shrink; define tiny interface at consumer |
| Panic on assert | `v.(T)` single-value form | Comma-ok or type switch |
| `i == nil` is false with a nil pointer inside | Nil concrete in non-nil interface | Check typed nils carefully |

**Close the TT:** Protocols over inheritance. Small, consumer-side, implicit. Next: catch a broken satisfaction story, then build `logpipe`.

<!-- > -->

## [**10m**] 🌴 Break

<!-- > -->

## [**20m**] 💻 Activity 1: Broken Store Satisfaction

> **DONE WHEN:** You have listed (1) what the author did right, (2) at least two concrete mistakes, and (3) a fix path that deletes the fat producer interface and uses a tiny consumer-side protocol — then compared notes with a teammate or the room for 2 minutes.

Paste this artifact into your editor (do not roast the author):

```go
package store

// Fat interface invented on the producer "for flexibility / mocking".
type Store interface {
	Get(key string) (string, error)
	Set(key, val string) error
	Delete(key string) error
	List() ([]string, error)
	Flush() error
	Connect() error // Java DAO muscle memory
}

type Mem struct {
	data map[string]string
}

func (m Mem) Get(key string) (string, error) {
	return m.data[key], nil
}

func (m *Mem) Set(key, val string) error {
	if m.data == nil {
		m.data = map[string]string{}
	}
	m.data[key] = val
	return nil
}

func (m *Mem) Delete(key string) error {
	delete(m.data, key)
	return nil
}

func (m Mem) List() ([]string, error) { return nil, nil }
func (m Mem) Flush() error            { return nil }
func (m Mem) Connect() error          { return nil }

// Return-interface smell + method-set landmine.
func NewStore() Store {
	return Mem{} // oops: Set/Delete need *Mem
}
```

1. Solo (about 8m): in notes, answer the three DONE WHEN prompts.
2. Pair or room share (about 8m): one person covers what is correct; the other covers mistakes + fix.
3. Optional compile-time guard sketch for your notes:

```go
func NewMem() *Mem {
	return &Mem{data: map[string]string{}}
}

// In the consuming package:
type Getter interface {
	Get(key string) (string, error)
}

var _ Getter = (*Mem)(nil)
```

> **PROTIP:** Before design lecture: does `return Mem{}` compile against `Store`? **No** — pointer-receiver methods are not in `Mem`'s method set.

> **BE AWARE:** “Fixed” by making every method a value receiver often creates a broken map-copy story. Prefer consistent `*Mem` + `NewMem() *Mem`.

> **FINISHED EARLY?** Rewrite `NewStore` into `NewMem() *Mem` and list the **minimum** interface a billing package would need if it only calls `Get` / `Set`.

<!-- > -->

## [**30m**] 💻 Activity 2: Build logpipe on io.Writer

> **DONE WHEN:** (1) `PipeTo(w io.Writer, msg string) error` exists and compiles, (2) you call it with **both** `os.Stdout` and a `bytes.Buffer` (or equivalent), (3) `go test` passes with `bytes.Buffer` or a `fakeWriter`, (4) you did **not** invent a god-interface, (5) you can say in one sentence why `io.Writer` instead of `*os.File`.

1. `mkdir -p ~/logpipe && cd ~/logpipe` (or `/tmp/logpipe`).
2. `go mod init example.com/logpipe`
3. Implement:

```go
package logpipe

import "io"

func PipeTo(w io.Writer, msg string) error {
	_, err := io.WriteString(w, msg)
	return err
}
```

4. Smoke from a small `main` or example: `PipeTo(os.Stdout, "hello\n")` and `PipeTo(&buf, "hello\n")`.
5. Write `pipe_test.go` using `bytes.Buffer` **or**:

```go
type fakeWriter struct{ b []byte }

func (f *fakeWriter) Write(p []byte) (int, error) {
	f.b = append(f.b, p...)
	return len(p), nil
}
```

6. `go test`
7. In chat or notes: paste the `PipeTo` signature + one sentence “why Writer not File.”

### Common blockers

| Blocker | Try this |
| --- | --- |
| `Write` undefined on `io.Writer` | `w.Write([]byte(msg))` or `io.WriteString` |
| Test cannot see writes | `bytes.Buffer` / pointer fake; read `buf.String()` after |
| Wanted to take `*os.File` | Widen to `io.Writer` — that is the point |
| Overbuilt `LoggerStoreRepository` | Delete it. One function + `io.Writer` is enough |
| `go test` finds no tests | File must end in `_test.go`; funcs `Test…` |

### Bonus

`UpperWriter` decorator — wraps an `io.Writer` (named field, not embedding) and uppercases bytes before forwarding:

```go
type UpperWriter struct{ W io.Writer }

func (u UpperWriter) Write(p []byte) (int, error) {
	up := make([]byte, len(p))
	for i, c := range p {
		if c >= 'a' && c <= 'z' {
			up[i] = c - 'a' + 'A'
		} else {
			up[i] = c
		}
	}
	return u.W.Write(up)
}
```

### Stretch (if core is done early)

1. `Pipe(r io.Reader, w io.Writer) (int64, error)` via `io.Copy`.
2. Method-set predict/confirm: tiny `Counter` / `Incrementer`; paste prediction **before** `go run`.
3. Short `Describe(v any) string` with a type switch.

> **PROTIP:** Gradescope or take-home follow-ups are fine after session — today’s gate is `PipeTo` + test + the one-sentence why.

> **BE AWARE:** Do not “fix for flexibility” with a six-method logger interface. Earn the second implementation first.

> **FINISHED EARLY?** Trade `UpperWriter` outputs with a teammate: predict, then run.

<!-- > -->

## [**5m**] Wrap Up

Say out loud:

1. Satisfaction is **implicit** — method-set containment, no `implements`.
2. Interface assign is **stricter** than call-site pointer sugar.
3. **Small** stdlib protocols (`io.Writer`) buy testability.
4. **Accept interfaces, return structs** — consumer-side, when a second implementation earns it.

Optional notes card:

```text
Shipped today:
Stuck on:
Next session first 15m:
```

After session: finish any unfinished `logpipe` core. Skim Effective Go + Code Review Comments on interfaces if “consumer-side” still feels fuzzy. Optional reframe: geometry `Shape` = syntax you know; design default you should not copy.

<!-- > -->

## Additional Resources

1. **[Effective Go — Interfaces](https://go.dev/doc/effective_go#interfaces)** — protocols, small interfaces, `Stringer`-style naming.
2. **[Code Review Comments — Interfaces](https://go.dev/wiki/CodeReviewComments#interfaces)** — consumer-side interfaces; accept interfaces, return concrete types.
3. **[Go Wiki — MethodSets](https://go.dev/wiki/MethodSets)** — `T` vs `*T` method sets and interface assignment.
4. **[pkg.go.dev/io](https://pkg.go.dev/io)** — `Reader`, `Writer`, `Copy`, composition helpers.
5. **[builtin — any](https://pkg.go.dev/builtin#any)** — `any` as alias for `interface{}` (Go 1.18+).
6. **[When To Use Generics](https://go.dev/blog/when-generics)** — optional; behavioral interfaces vs type parameters.
7. **[Types.md](Types.md)** — survey mention of interfaces; hire-bar depth is this lesson.
8. **[07-Fullstack.md](07-Fullstack.md)** — geometry `Shape` as prior-knowledge reframe only.

<details>
<summary>For curriculum authors</summary>

### In Class

| | |
| --- | --- |
| **Next action** | Open this file → Agenda → Announcements (Shape reframe one-liner) → Warm Up protocol cards. |
| **Done when** | Room has restated implicit satisfaction + pointer trap, and at least one person has a compiling `PipeTo` + test path. |

- Warm Up is think → chat → share; keep the four cards short.
- Live-code Greeter + method-set trap only; then get out of the way for Activity 1.
- Behind at ~0:40? Cut generics flash and `UpperWriter`; protect Favorite-No analysis + `PipeTo`.

### Facilitator Notes

- Prefer speakable TT. Keep Day-7 Shape as a **5m prior-knowledge reframe**, not a second lesson.
- Protocol Telephone secret cards (DM if you still run the game form):  
  **A** no `implements` / method-set containment · **B** `T` vs `*T` / no auto-`&` on assign · **C** small `io.Reader` · **D** consumer-side / return structs.
- Favorite No answer key (do not paste whole into chat): fat interface before use; `return Mem{}` method-set failure; return-interface smell; `Connect` on a map. Fix = `NewMem() *Mem` + tiny consumer `Getter`/`GetterSetter`.
- Prep tabs: [Effective Go — Interfaces](https://go.dev/doc/effective_go#interfaces), [CRC — Interfaces](https://go.dev/wiki/CodeReviewComments#interfaces), [pkg.go.dev/io](https://pkg.go.dev/io), optional [MethodSets](https://go.dev/wiki/MethodSets).
- Explicit non-goals (say once): not a Java inheritance redesign; no deep type sets / `~T`, reflection, `unsafe`, mockgen, or escape-analysis performance dive.

### Expert Follow-Ups

- Optional after-session: interface values + escape analysis curiosity (do not derail).
- If a Gradescope interfaces drill exists for your section, confirm the queue is open before Wrap.
- Cross-link SSG / docs deploy tracks that already write bytes — `io.Writer` is how that stays testable.

</details>
