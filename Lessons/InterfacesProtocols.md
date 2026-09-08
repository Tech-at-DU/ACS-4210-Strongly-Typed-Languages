# 📜 Day N: Interfaces & Protocols (Behavior Over Inheritance)

<!-- omit in toc -->
## ⏱ Agenda

1. [[**05m**] 🌤 Check-In / Entrance Ticket](#05m--check-in--entrance-ticket)
2. [[**05m**] 🏆 Objectives](#05m--objectives)
3. [[**15m**] 🎮 Warm-Up: Protocol Telephone](#15m--warm-up-protocol-telephone)
4. [[**45m**] 💬 TT: Interfaces & Protocols in Go](#45m--tt-interfaces--protocols-in-go)
5. [[**10m**] 🌴 BREAK](#10m--break)
6. [[**10m**] 🔍 Activity: My Favorite No (Broken satisfaction)](#10m--activity-my-favorite-no-broken-satisfaction)
7. [[**45m**] 🧪 Lab Time: Solo Zoom Work](#45m--lab-time-solo-zoom-work)
8. [[**05m**] 🌃 Exit Ticket](#05m--exit-ticket)
9. [📚 Resources & Credits](#-resources--credits)

**Class shape (Fall 2026):** 2 students on Zoom in the same room. Keep discussion purposeful (short, structured), not open-ended debate. Active learning ≥20m = Warm-Up (15) + My Favorite No (10). TT stays **45m**.

Techniques from [How to Write a Self-Paced Lesson](https://docs.google.com/document/d/16O2xGeNSayfQ1cC03fwMKzBA25c5yrnMrrQ5QfKyY50) + [Lesson Template](https://docs.google.com/document/d/1Tr3wg6LeU_ZVkeKhyf4p1uPfpUgkgnuj4rUFJCfRrKM): check-in, warm-up game, teacher talk, independent + guided lab, My Favorite No, exit ticket, measurement.

### Instructor prep (before class)

- [ ] Confirm both students can open a terminal and have a **supported** Go (`go version` → **1.27.x or 1.26.x** as of Sep 2026).
- [ ] Open [Effective Go — Interfaces](https://go.dev/doc/effective_go#interfaces), [Code Review Comments — Interfaces](https://go.dev/wiki/CodeReviewComments#interfaces), and [pkg.go.dev/io](https://pkg.go.dev/io) in browser tabs.
- [ ] Create a throwaway demo folder ready for live typing (`/tmp/greeterdemo`).
- [ ] DM list ready for Protocol Telephone cards (below).
- [ ] Favorite No paste + answer key ready (below).
- [ ] Optional: skim [MethodSets wiki](https://go.dev/wiki/MethodSets) once so the pointer-trap demo is crisp.
- [ ] Optional: open [Types.md](Types.md) survey line + [07-Fullstack.md](07-Fullstack.md) Shape warmup so you can cross-link without derailing.

**Instructor note — explicit non-goals (state once, out loud or in chat):** This lesson is **not** a Java inheritance redesign. We will **not** deep-dive type sets / `~T`, reflection, `unsafe`, custom `ReadWriteCloser` when `io` already composes, mockgen, or interface-call / escape-analysis performance. Day 7 geometry `Shape` is **not** the hire-bar interfaces story — keep [Types.md](Types.md) as survey and treat 07-Fullstack Shape as an optional **5m prior-knowledge reframe** (“cool syntax; wrong default design story”). Hire-bar depth lives **here**.

## [**05m**] 🌤 Check-In / Entrance Ticket

**Solo, then paste in Zoom chat.**

1. How are you feeling today (one word)?
2. On methods / receivers / structs: Behind, on track, or ahead?
3. What is your plan for today’s lab (one sentence)?

**Instructor:** Say both names when reflecting pacing (“Alex is solid on pointer receivers; Jordan wants a clean mental model for interfaces”). Growth-mindset language. If someone is “behind,” name one concrete first lab step they will hit first (`go mod init` + empty `PipeTo` stub).

## [**05m**] 🏆 Objectives

**Learning outcomes** — by the end of this session, students will be able to:

1. Explain that a type implements an interface **implicitly** (no `implements` keyword).
2. Predict compile errors from **pointer vs value receiver** method-set rules when assigning to an interface.
3. Use `io.Reader` / `io.Writer` (and `io.Copy`) as the canonical small-interface story.
4. Distinguish `any` (alias for `interface{}`) from meaningful interfaces; use type assertions / switches safely.
5. Apply “accept interfaces, return structs” + consumer-side interfaces for a testable fake — and state when **not** to introduce an interface.

**Pre-measurement (chat, 1–5):** “I can design a small consumer-side interface and satisfy it implicitly with a concrete type.”

**Measurement (end of class):** Exit ticket + pasted `PipeTo` signature + evidence that the function accepts `io.Writer` (not only `*os.File`).

**Why this matters:** Hire-bar Go is protocol thinking, not inheritance theater. Your SSG already writes bytes somewhere — `io.Writer` is how that stays testable without inventing a god-interface. Interviews will ask “when would you *not* introduce an interface?” — answer that out loud today.

## [**15m**] 🎮 Warm-Up: Protocol Telephone

**Game** (relationship-building + concept hook). Both students stay in the main Zoom room. No breakouts.

### Secret cards (DM privately — one per round)

**Card A:** Go has **no** `implements` keyword. A type satisfies an interface when its **method set contains** every method the interface requires — that is it.

**Card B:** The method set of `T` is not the same as `*T`. Interface assignment uses the **strict** method set of the value you assign. Go will **not** auto-`&` a value just to make an interface assignment compile.

**Card C:** Prefer **small** interfaces. `io.Reader` is only `Read([]byte) (int, error)` — one method. That is a feature, not incomplete design.

**Card D:** Put interfaces with the **consumer**, not the producer. Constructors should usually **return concrete structs** (or pointers), not fat interfaces invented on the producer side.

### Timing script

| Minute | Move |
| --- | --- |
| 0:00–0:01 | Explain rules. No jargon dumps — write **steps** only. |
| 0:01–0:07 | Round 1: Student A gets Card A or B. Writes steps in chat. Student B reconstructs the rule. Reveal. |
| 0:07–0:13 | Round 2: Swap. New card (C or D). Reveal. |
| 0:13–0:15 | Debrief in chat (one line each). |

### Rules (play)

1. **Student A** has ~60 seconds to write **only requirements / steps** that would let someone else rediscover the truth — into Zoom chat.
2. **Student B** cannot see the secret. From A’s steps alone, B writes what they think the rule is.
3. Reveal the secret. Compare.
4. Swap roles.

### Debrief prompts (structured discussion)

Each student answers in chat (one line each):

- What got lost in translation?
- Where would that loss show up in a real repo (compile fail on interface assign, untestable `*os.File`-only API, fat DAO interface nobody needed)?

**Success check:** both students can restate at least one of: **implicit satisfaction** / **pointer trap** / **small Reader** / **consumer-side**.

**Instructor punchline:** Interfaces are protocols — “can you do this?” — not family trees.

## [**45m**] 💬 TT: Interfaces & Protocols in Go

Teacher talk + live demos. Students follow along quietly; unmute only if stuck.

### Minute guide (keep the clock honest)

| TT minutes | Beat |
| --- | --- |
| 0–7 | Protocols not inheritance; implicit satisfaction |
| 7–15 | Live demo: tiny `Greeter` + `Human` / `Robot` |
| 15–24 | `io.Reader` / `Writer` + `io.Copy` |
| 24–32 | Method sets: pointer vs value + interface assignment trap |
| 32–38 | `any` vs `interface{}`; type assert / switch; nil interface flash |
| 38–45 | Accept interfaces / return structs; fakes; when NOT to interface; optional ~3m generics flash |

Lightning only (do **not** derail): Day 7 geometry `Shape` is prior-knowledge syntax — cool; **wrong default design story** for hire-bar Go. Generics constraints are interfaces under the hood — flash at the end, not a second lesson.

**TT overrun cut order:** drop optional generics flash first; keep Clock fake + when-NOT (cut `realClock` nil-default branch narration if needed, keep the fake). Never cut Telephone, Favorite No, or the method-set demo.

### Protocols, not inheritance (0–7)

Java / C# habits whisper: declare an interface, write `implements`, build a hierarchy, interface *everything*.

Go does something quieter.

An **interface** names a **protocol** — a set of method signatures. A type **satisfies** that protocol when it has those methods. There is **no** `implements` keyword. The compiler checks method-set **containment** at the assignment / call site.

That means:

- Types never have to mention the interface they satisfy.
- One type can satisfy many interfaces without listing them.
- You can invent a tiny interface **next to the code that needs it**, long after the concrete type was written.

**Chat checkpoint (30s):** Type `yes` or `no` — does a Go type need an `implements` line to satisfy an interface?

### Live demo 1 — Greeter never mentioned by the types (7–15)

```bash
mkdir -p /tmp/greeterdemo && cd /tmp/greeterdemo
go mod init example.com/greeterdemo
```

Write `main.go`:

```golang
package main

import "fmt"

// Greeter is a tiny protocol: can you greet?
type Greeter interface {
	Greet() string
}

type Human struct {
	Name string
}

func (h Human) Greet() string {
	return "hello, " + h.Name
}

type Robot struct {
	ID int
}

func (r Robot) Greet() string {
	return fmt.Sprintf("beep-%d", r.ID)
}

func announce(g Greeter) {
	fmt.Println(g.Greet())
}

func main() {
	announce(Human{Name: "Alex"})
	announce(Robot{ID: 7})
}
```

Run:

```bash
go run .
```

**Say this out loud:** `Human` and `Robot` never mention `Greeter`. Satisfaction is **implicit**. If you delete `Greet` from `Robot`, the error shows up at `announce(Robot{…})` — the **use** site — not on the struct definition.

**Also say:** This is why “interface everything on the producer” is the wrong default. The interesting protocol lives with the **caller** (`announce`), not with a fake “IGreeter” package invented up front.

### Live demo 2 — `io.Reader` / `Writer` + `io.Copy` (15–24)

Open [pkg.go.dev/io](https://pkg.go.dev/io) and read the real signatures:

```golang
type Reader interface {
	Read(p []byte) (n int, err error)
}

type Writer interface {
	Write(p []byte) (n int, err error)
}
```

That is the whole protocol. Files, buffers, network connections, string readers — same two shapes.

```golang
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

**Narrate:** `strings.NewReader` returns a concrete `*strings.Reader`. `os.Stdout` is an `*os.File`. Neither type “implements Writer/Reader” in source — they just have `Read` / `Write`. `io.Copy` only asks for the protocol.

**SSG / hire-bar bridge:** If your function takes `*os.File`, tests need a real file. If it takes `io.Writer`, tests can use `bytes.Buffer` or a three-line fake. Small interfaces buy testability without a mock framework.

**Composition flash (~30s):** Stdlib composes small interfaces — `io.ReadWriter` embeds `Reader` and `Writer`. Prefer that over inventing `MyReadWriteCloser`.

**Chat checkpoint (30s):** How many methods does `io.Reader` require?

### Live demo 3 — method sets and the pointer trap (24–32)

This is the compile error students hit in interviews and in lab.

```golang
package main

type Incrementer interface {
	Inc()
}

type Counter struct {
	N int
}

func (c *Counter) Inc() {
	c.N++
}

func main() {
	var c Counter

	// Fine: method call sugar. For an addressable c, Go may call (&c).Inc().
	c.Inc()

	// FAIL to compile: method set of Counter does NOT include Inc
	// (Inc has a pointer receiver → method set of *Counter).
	// var i Incrementer = c

	// OK: *Counter's method set includes Inc.
	var i Incrementer = &c
	i.Inc()
	_ = i
}
```

**Teach the two rules students mix up:**

1. **Method call sugar:** if `c` is **addressable**, `c.Inc()` can become `(&c).Inc()` even when `Inc` has a pointer receiver.
2. **Interface assignment is stricter:** the value you assign must **already** have the method in its method set. Go will **not** invent `&c` to satisfy `Incrementer`.

| You have | Method set includes… | Assign to interface needing `Inc()`? |
| --- | --- | --- |
| `Counter` value (`T`) | value-receiver methods on `T` | **No** if `Inc` is on `*T` |
| `*Counter` (`*T`) | value-receiver **and** pointer-receiver methods | **Yes** |

**Instructor line:** `*T` method set = pointer-receiver **and** value-receiver methods; `T` never gains pointer-receiver methods for free.

**Also flash:** once a value is **inside** an interface, that interface value is **not** addressable the way a local variable is — you do not get free “take address and call pointer method” sugar through the interface slot. Stick to putting the right concrete type (`*Counter`) into the interface.

**Predict aloud:** “Will `var i Incrementer = c` compile?” Wait for chat. Then uncomment / show the error.

### `any`, assertions, switches, nil flash (32–38)

Since **Go 1.18**, `any` is a predeclared alias for `interface{}`. Same empty interface. Prefer `any` in new code for readability.

```golang
package main

import "fmt"

func describe(v any) string {
	// Comma-ok assertion — never panic-assert in homework / prod paths.
	if s, ok := v.(string); ok {
		return "string:" + s
	}

	switch x := v.(type) {
	case int:
		return fmt.Sprintf("int:%d", x)
	case nil:
		return "nil-interface" // only true nil interface (no dynamic type)
	default:
		return fmt.Sprintf("other:%T", x)
	}
}

func main() {
	fmt.Println(describe("hi"))
	fmt.Println(describe(3))

	var p *int = nil
	var i any = p // i is not the nil interface; it holds (*int, nil)
	fmt.Println(i == nil) // false — classic footgun
	fmt.Println(describe(i))
}
```

**Say this out loud:**

- `any` / `interface{}` means “I gave up on a meaningful protocol.” Use it for unknown JSON bags, debug helpers, rare escape hatches — **not** as your domain API.
- Prefer **comma-ok** assertions (`v.(T)` two-value form). Single-value assert panics on mismatch.
- **Nil interface flash:** a nil **pointer** stored in an interface is **not** equal to a nil interface. That bites error returns and “is it nil?” checks. Name it; do not spend the whole TT here.
- **Typed nil vs `case nil`:** typed nil in `any` still has a dynamic type — `describe(i)` prints `other:*int`, **not** the `case nil` arm. Only a true nil interface (no dynamic type) matches `case nil`. `i == nil` is false for the same reason.

### Accept interfaces, return structs; fakes; when NOT (38–45)

Hire-bar paraphrase of [Code Review Comments — Interfaces](https://go.dev/wiki/CodeReviewComments#interfaces): put interfaces with the consumer; return concrete types from constructors.

> **Accept interfaces, return structs.** (community slogan — not a verbatim CRC quote)

More precisely for this class:

- **Parameters:** ask for the **smallest** protocol you need (`io.Writer`, not `*os.File`; `clock`, not `time` package god-object).
- **Returns / constructors:** return **concrete** types (`*Mem`, `*Server`) so callers can use the full API and you can add methods later without breaking every interface implementor.
- **Define the interface next to the consumer** (often the package under test), not “for mocking” on the producer.

Consumer-side clock fake:

```golang
package bill

import "time"

// Clock lives with the consumer. Production code passes a real clock;
// tests pass a fake. Producer packages do not invent IClock.
type Clock interface {
	Now() time.Time
}

type realClock struct{}

func (realClock) Now() time.Time { return time.Now() }

type Service struct {
	clock Clock
}

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

```golang
package bill

import (
	"testing"
	"time"
)

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

### When *not* to introduce an interface (~90s)

Do **not** “optimize” design by:

- Writing `IStore` / `IUserRepository` before you have **two** real call sites that need the protocol
- Returning interfaces from constructors “for testability” when a concrete `*T` + consumer-side interface would do
- Growing a fat interface (Get/Set/Delete/List/Flush/Connect…) so every fake must implement methods you never call
- Replacing a perfectly fine concrete helper with `any` + type switches
- Porting Java DAO / Spring-style interface-everything into Go homework

**Exception (~20s — do not overfit):** returning an interface **is** OK when the concrete type is chosen at runtime (`hash.Hash`, `net.Conn`) or when the type *is* the protocol (`error`). Homework default remains: return `*T`.

**Interview prompts (say aloud if time):** “Do I have two real implementations, or am I mocking fear?” / “Need only `Write` → take `io.Writer`; need `Write`+`Close` → `io.WriteCloser` — not `*os.File` just in case.”

**Optional ~3m generics flash (only if time):** constraints are interfaces. `comparable` is special (constraint-only; you cannot use it as a normal interface variable type the way students expect). Punchline: **generics** share algorithms across types; **behavioral interfaces** share method protocols. Do not rewrite `io.Reader` as a type parameter — the blog [When To Use Generics](https://go.dev/blog/when-generics) says the same thing.

### Failure modes worth a grin

| What you see | What you probably did | Fix |
| --- | --- | --- |
| `X does not implement Y (Inc method has pointer receiver)` | Assigned `T` where `*T` is required | Assign `&x` / store `*T`; or give value receivers if mutation is not needed |
| Works as `x.Inc()` but fails as `var y I = x` | Mixed up call sugar vs interface method set | Teach addressable sugar ≠ interface assign |
| Tests need real files / network | Function took `*os.File` / concrete client | Accept `io.Writer` / small consumer interface |
| Every fake is huge | Fat producer-side interface | Shrink; define tiny interface at consumer |
| Panic on assert | `v.(T)` single-value form | Use comma-ok or type switch |
| `i == nil` is false with a nil pointer inside | Nil concrete in non-nil interface | Check typed nils carefully; prefer concrete error returns |

### On the job (quick)

- Start with concrete types. Extract an interface when a second implementation (or a test fake) **earns** it.
- Keep interfaces small — one or two methods is normal (`io.Reader`, `fmt.Stringer`, `json.Marshaler`).
- Put interfaces with the **consumer**. Return structs from constructors.
- Compose stdlib protocols (`io.Reader` + `io.Writer`) instead of inventing `ReadWriteCloser` clones.
- Prefer `any` only when the value is truly unknown; prefer named protocols for domain behavior.

**Close the TT:** Protocols over inheritance. Small, consumer-side, implicit. Next: catch a broken satisfaction story, then build `logpipe`.

## [**10m**] 🌴 BREAK

## [**10m**] 🔍 Activity: My Favorite No (Broken satisfaction)

**My Favorite No** — whole-class analysis of one anonymous wrong answer. No pile-on critique.

### Paste this artifact (Zoom chat or shared doc)

```golang
// anonymous "favorite no" — do not roast the author
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

### Solo (4m)

In notes, list:

1. What did they do **correctly**?
2. What is the **mistake** (there may be more than one)?
3. What **exact changes** fix it?

### Structured discussion (6m, 2 students)

- Student A: what’s correct (60–90s)
- Student B: mistakes + fix (60–90s)
- Instructor: tie back to Telephone + TT (2–3m)

### Instructor answer key (do not paste whole)

**Before design lecture: does `return Mem{}` compile? No.**

**Correct-ish:** They reached for a storage abstraction and used methods on a struct. Intent to test is fine.

**Mistakes:**

1. **Fat interface before use** — six methods including `Connect` with no second implementation and no consumer that needs all six. Classic producer-side / Java DAO smell.
2. **Method-set failure** — `NewStore() Store { return Mem{} }` returns a **value**. `Set` / `Delete` have **pointer** receivers, so `Mem` does **not** satisfy `Store`. Compile error (or they “fixed” it by making everything value receivers and sharing a broken map copy story).
3. **Return-interface smell** — constructor returns `Store` instead of `*Mem`. Callers lose concrete API; every test fake must implement the whole fat surface.
4. **`Connect` on an in-memory map** — inherited ORM / DAO ritual, not a Go protocol earned by the consumer.

**Fix path:**

- Delete the god-interface on the producer.
- Use consistent pointer receivers on `Mem`; construct with `NewMem() *Mem`.
- At the **consumer**, define a tiny interface with only the methods that consumer calls (often just `Get` / `Set`).
- Put a fake in `_test.go` that implements **that** tiny interface — no mockgen, no `Connect`.

Sketch:

```golang
func NewMem() *Mem {
	return &Mem{data: map[string]string{}}
}

// In the consuming package:
type Getter interface {
	Get(key string) (string, error)
}

// optional compile-time guard (interviewers love this):
var _ Getter = (*Mem)(nil)
```

## [**45m**] 🧪 Lab Time: Solo Zoom Work

**Independent work** first; **guided** check-ins at the end. Stay in the main Zoom room. Mute is fine. Chat if stuck. Optional 2-person “ask classmate once in chat before the instructor.”

### Success criteria (everyone)

Build a tiny `logpipe` module that writes log lines through `io.Writer`.

You are done with **core** when:

1. `PipeTo(w io.Writer, msg string) error` exists and compiles
2. You call it with **both** `os.Stdout` and a `bytes.Buffer` (or equivalent)
3. `go test` passes with either `bytes.Buffer` or a `fakeWriter`
4. You did **not** invent a god-interface
5. You pasted the `PipeTo` signature into Zoom chat **and** one sentence: why `io.Writer` instead of `*os.File`

### Suggested sequence

1. `mkdir -p ~/logpipe && cd ~/logpipe` (or `/tmp/logpipe` if you prefer throwaway)
2. `go mod init example.com/logpipe`
3. Implement:

```golang
package logpipe

import "io"

func PipeTo(w io.Writer, msg string) error {
	_, err := io.WriteString(w, msg)
	return err
}
```

4. Smoke from a small `main` or example: `PipeTo(os.Stdout, "hello\n")` and `PipeTo(&buf, "hello\n")`.
5. Write `pipe_test.go` using `bytes.Buffer` **or**:

```golang
type fakeWriter struct {
	b []byte
}

func (f *fakeWriter) Write(p []byte) (int, error) {
	f.b = append(f.b, p...)
	return len(p), nil
}
```

6. `go test`
7. Paste signature + “why Writer not File” into Zoom chat

### Common blockers

| Blocker | Try this |
| --- | --- |
| `Write` undefined on `io.Writer` | Call `w.Write([]byte(msg))` or `io.WriteString(w, msg)` — do not assume extra methods |
| Test cannot see writes | Use `bytes.Buffer` / pointer fake; read `buf.String()` after `PipeTo` |
| Wanted to take `*os.File` | That is the whole point — widen to `io.Writer` |
| `Mem`-style assign error in stretch | Re-check pointer vs value method sets before blaming the interface |
| Overbuilt `LoggerStoreRepository` | Delete it. One function + `io.Writer` is enough |
| `go test` finds no tests | File must end in `_test.go` and funcs must be `Test…` |

### Bonus

`UpperWriter` decorator — a type that wraps an `io.Writer` (decorator; named field, not embedding) and uppercases bytes before forwarding:

```golang
type UpperWriter struct {
	W io.Writer
}

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

### Stretch (if you finish early)

1. `Pipe(r io.Reader, w io.Writer) (int64, error)` implemented with `io.Copy`.
2. **Mandatory if core is done before minute 30** — method-set predict/confirm: write a tiny `Counter` / `Incrementer` snippet (same as TT); paste your prediction in chat **before** `go run`; confirm with the compiler whether `var i Incrementer = c` compiles.
3. Short `Describe(v any) string` using a type switch (`string`, `int`, default).

### Guided check-ins (last ~10m)

- Student A screen share: `PipeTo` + passing test + chat paste
- Student B screen share: same
- Remaining minutes: bonus / stretch or keep coding

## [**05m**] 🌃 Exit Ticket

**Solo in Zoom chat / quick form:**

1. Did you finish core? Paste your `PipeTo` signature (or `blocked: …` + one blocker).
2. Why might an interface assignment fail when `myValue.Method()` seemed fine? (Name the rule: addressable sugar vs method set of the assigned type.)
3. Prefer **return struct** or **return interface** from a constructor — why (hire-bar)?
4. Who helped you today / who did you help? (classmate or instructor)

### After class

- Finish any unfinished `logpipe` core / bonus before next class.
- Skim [Effective Go — Interfaces](https://go.dev/doc/effective_go#interfaces) and [Code Review Comments — Interfaces](https://go.dev/wiki/CodeReviewComments#interfaces) if “consumer-side” still feels fuzzy.
- Optional reframe: if you already did geometry `Shape` in [07-Fullstack.md](07-Fullstack.md), rewrite the story as “syntax you know; design default you should not copy.”

## 📚 Resources & Credits

- [**How to Write a Self-Paced Lesson**](https://docs.google.com/document/d/16O2xGeNSayfQ1cC03fwMKzBA25c5yrnMrrQ5QfKyY50) — check-in, warm-up menu, exit ticket patterns.
- [**Lesson Template**](https://docs.google.com/document/d/1Tr3wg6LeU_ZVkeKhyf4p1uPfpUgkgnuj4rUFJCfRrKM) — LO / pre-measurement / independent + guided work / measurement.
- [**Effective Go — Interfaces**](https://go.dev/doc/effective_go#interfaces) — protocols, small interfaces, `Stringer`-style naming.
- [**Code Review Comments — Interfaces**](https://go.dev/wiki/CodeReviewComments#interfaces) — consumer-side interfaces; accept interfaces, return concrete types; don’t interface before use.
- [**Go Wiki — MethodSets**](https://go.dev/wiki/MethodSets) — `T` vs `*T` method sets and interface assignment.
- [**pkg.go.dev/io**](https://pkg.go.dev/io) — `Reader`, `Writer`, `Copy`, composition helpers.
- [**builtin — any**](https://pkg.go.dev/builtin#any) — `any` as alias for `interface{}` (Go 1.18+).
- [**When To Use Generics**](https://go.dev/blog/when-generics) — optional; behavioral interfaces vs type parameters.
- [**DocsDeploy.md**](DocsDeploy.md) — existing `io.Writer` samples in the deploy / docs track (cross-link).
- [**Types.md**](Types.md) — survey mention of interfaces; hire-bar depth is this lesson.
- [**07-Fullstack.md**](07-Fullstack.md) — geometry `Shape` warmup as prior-knowledge reframe only.
