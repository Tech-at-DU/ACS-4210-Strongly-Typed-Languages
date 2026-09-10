<!-- markdownlint-disable MD010 -->
# APIs, Middleware, and You

⭐️ **GOAL:** Leave able to explain why middleware exists, trace an Echo request through the onion, and ship a custom `e.Use` middleware that labels “inside the building?” without trusting spoofable headers.

| **Elapsed** | **Time** | **Activity** |
| ----------- | -------- | ------------------------- |
| 0:00 | 0:05 | Why / Objectives |
| 0:05 | 0:40 | Overview / TT |
| 0:45 | 0:20 | Activity 1 |
| 1:05 | 0:10 | BREAK |
| 1:15 | 0:30 | Activity 2 |
| 1:45 | 0:10 | Lab Time |
| 1:55 | 0:05 | Wrap Up |
| **TOTAL** | **2:00** | |

---

## Why You Should Know This (2 min)

Middleware is the **hallway** of your API. Every request walks through it before a handler runs — and often again on the way out.

If you can write middleware well, you can:

- add **logging, tracing, auth, rate limits, and request IDs** without copy-pasting into every handler
- keep handlers thin: **parse → decide → respond**
- spot rookie traps: stuffing business logic into the hallway, trusting spoofable IP headers, or registering middleware in the wrong order

**GOAL:** leave able to explain *why* middleware exists, walk a request through Echo’s lifecycle, and ship a custom middleware that decides “inside the building?” without lying to yourself about headers.

---

## Outcomes (3 min)

1. **Define** middleware and name three jobs that belong there (cross-cutting) vs three that do **not**.
2. **Trace** an Echo request: `Pre` → router → `Use` chain → handler → unwind, including why registration order matters.
3. **Implement** a custom Echo middleware (`func(next echo.HandlerFunc) echo.HandlerFunc`) and register it with `e.Use`.
4. **Build** an “inside the building” detector that uses `c.RealIP()` / private ranges **and** names the header-spoof traps.

**How you’ll know:** `/whoami` returns JSON with `ip`, `inside`, and `via`, and a forged `X-Forwarded-For` does **not** silently flip the answer unless you explicitly opted into that extractor.

---

## Overview / TT (40 min)

**Next action:** Run this talk track — GOAL first, then the onion, then code.  
**Done when:** The room can sketch the onion and say one sentence for “when NOT middleware.”  
**≤2m next after TT:** Open Activity 1 and create the module.

### 1. GOAL — what “good” looks like (~5m)

Say:

> “Handlers do the *work* of a route. Middleware does the *policies* that apply to many routes. If you catch yourself putting ‘is this user allowed to buy this SKU?’ in middleware, stop — that’s handler (or service) territory. If you catch yourself pasting the same logger into twelve handlers, that’s middleware.”

Whiteboard / screen three buckets:

| Belongs in middleware | Belongs in handler / service | Maybe either — decide on purpose |
| --- | --- | --- |
| Recover from panic | Business rules / domain decisions | Feature flags (thin check vs full eval) |
| Request ID + access log | DB reads/writes for this resource | Light authn gate vs full authz |
| CORS / Secure headers | Response body shape for this route | Caching (usually careful + explicit) |
| Rate limit / body size cap | Orchestrating third-party APIs | — |
| Tracing span start/end | — | — |

**Beginner tip for on-the-job success:** middleware should be **cheap, predictable, and side-effect light**. Heavy work belongs deeper.

**Pulse check 1 (≤60s):** Which layer owns authz for “can this user buy this SKU?” — middleware or handler/service? Drop one word in chat or raise a hand. Expected: **handler/service** (middleware can do a thin authn gate; SKU rules stay out of the hallway).

### 2. The onion — Echo request lifecycle (~9m)

Echo middleware signature (current docs, Echo **v5**):

```go
// MiddlewareFunc
func(next echo.HandlerFunc) echo.HandlerFunc
```

Handlers / middleware receive `*echo.Context` in v5.

Two registration lanes:

| API | When it runs | Use it for |
| --- | --- | --- |
| `e.Pre(...)` | **Before** the router matches a route (still runs on 404) | Path rewrites, trailing-slash normalize, anything that must see *every* request |
| `e.Use(...)` | **After** the router found a matching route, before the handler | Logging, recover, auth, your custom “inside” check on real routes |

**Onion with registration order** — first registered = outermost:

```text
e.Pre(A)
e.Use(B)   // registered first  → outer
e.Use(C)   // registered second → inner
// handler H

Request in:   A → (router) → B → C → H
Response out: H → C → B → A
```

Code shape of “before and after”:

```go
func TraceName(name string) echo.MiddlewareFunc {
	return func(next echo.HandlerFunc) echo.HandlerFunc {
		return func(c *echo.Context) error {
			// BEFORE next
			c.Logger().Info("enter", "mw", name)
			err := next(c)
			// AFTER next (always runs unless you panic without Recover)
			c.Logger().Info("leave", "mw", name)
			return err
		}
	}
}
```

**Pulse check 2 (≤60s):** `Pre` vs `Use` — which lane still runs on a 404? Expected: **`Pre`**.

**Order that won’t embarrass you in review** (outer → inner):

1. **`Recover`** — outermost so panics in anything below become 500s, not process death  
2. **Request ID** — so every log line can correlate  
3. **Request logger / access log** — sees status + latency after the chain returns  
4. **Security / CORS / body limit / rate limit** — reject early, cheaply  
5. **Authn** (who are you?) — before handlers that need identity  
6. **Your feature middleware** (e.g. inside-the-building tag) — after you trust identity/IP strategy  
7. **Handler** — business logic  

**Auth vs logging vs tracing, said out loud:**

- **Tracing / request ID** early so *everything* shares an ID — including failed auth.  
- **Logging** wraps the chain so it can record final status + latency (post-`next`).  
- **Auth** after recover/ID/log plumbing, **before** expensive handler work — and usually **before** “feature” middleware that assumes a user.  
- Do **not** put auth inside the logger. Do **not** put “load the whole user graph” in middleware unless you love latency.

### 3. Built-ins you’ll actually touch (~5m)

From `github.com/labstack/echo/v5/middleware` (names per current Echo docs):

```go
e := echo.New()
e.Use(middleware.Recover())
e.Use(middleware.RequestID())
e.Use(middleware.RequestLogger()) // v5: prefer this over the removed Logger()
```

⚠️ **v4 → v5 gotchas (say once):**

- **Toolchain gate:** Echo **v5.3.1** needs **Go ≥ 1.25**. Below that → stay on **v4 today** (`github.com/labstack/echo/v4` + `echo.Context` + `middleware.Logger()`). Ideas identical; signatures differ.
- Prefer pin: `go get github.com/labstack/echo/v5@v5.3.1` (not `@latest` on session day).
- Import path: `github.com/labstack/echo/v5` (and `.../v5/middleware`).  
- Context is `*echo.Context`, not the old interface-style `echo.Context`.  
- `middleware.Logger()` / `Timeout()` were removed from core in v5 — use `RequestLogger` (and a timeout strategy you choose deliberately).  
- Custom middleware cookbook still shows `e.Use(ServerHeader)` with `c.Response().Header().Set(...)`.
- `e.Start(":1323")` is still valid for MVP; `StartConfig` is graceful-shutdown later — no MVP change.
- `middleware.RequestLogger()` zero-config exists; richer setups use `RequestLoggerWithConfig`.
- `c.Logger()` is a usable `*slog.Logger` (Info kv style OK).

**Sample vs safer ship order:** Echo’s own hello-world often registers `RequestLogger` *before* `Recover` (logger outer so it can observe the recovered error). Today’s TT still defaults to **Recover outermost** as the safer ship default when you panic-proof *everything* including logger middleware — call the tradeoff in one sentence if someone notices the docs differ.

**Pulse check 3 (≤60s):** Recover outer vs logger outer — which do you ship by default here, and why? Expected: **Recover outermost** so a panic inside logger middleware (or anything below) still becomes a 500, not process death. Logger-outer is a valid alternative when you want the access log to record the recovered error — name the tradeoff.

If the service under review is still on **v4**, the *ideas* are identical; swap types/imports and use `middleware.Logger()` where `RequestLogger` isn’t available.

### 4. Live skeleton — custom middleware in under a minute (~7m)

Speak while typing (or paste once, then walk):

```go
package main

import (
	"net/http"

	"github.com/labstack/echo/v5"
	"github.com/labstack/echo/v5/middleware"
)

func ServerHeader(next echo.HandlerFunc) echo.HandlerFunc {
	return func(c *echo.Context) error {
		c.Response().Header().Set(echo.HeaderServer, "ACS-4210-Echo")
		return next(c)
	}
}

func main() {
	e := echo.New()
	e.Use(middleware.Recover())
	e.Use(middleware.RequestID())
	e.Use(ServerHeader)

	e.GET("/", func(c *echo.Context) error {
		return c.String(http.StatusOK, "hallway cleared")
	})

	if err := e.Start(":1323"); err != nil {
		e.Logger.Error("failed to start server", "error", err)
	}
}
```

**Predict-then-run:** What header do you expect on `curl -i localhost:1323/`?  
Then run it. Point at `Server` in the response.

Optional stretch while talking: the official [Custom Middleware cookbook](https://echo.labstack.com/cookbook/middleware/) `Stats.Process` pattern — count requests **after** `next(c)` so status codes are real.

### 5. IP reality check — fuel for Activity 2 (~10m)

Echo gives you `c.RealIP()`. **That string is only as trustworthy as `e.IPExtractor`.**

From Echo’s IP guide:

| Situation | Set |
| --- | --- |
| No proxy (dev laptop, direct) | `e.IPExtractor = echo.ExtractIPDirect()` |
| Proxies append `X-Forwarded-For` | `e.IPExtractor = echo.ExtractIPFromXFFHeader(/* TrustOption... */)` |
| Proxies set `X-Real-IP` | `e.IPExtractor = echo.ExtractIPFromRealIPHeader(/* TrustOption... */)` |
| Default (unset) | Legacy mix of XFF / X-Real-IP / remote addr — **not a secure default** |

**Traps to name out loud (write these on screen):**

1. **Forged headers** — a client can send `X-Forwarded-For: 10.0.0.1` and look “internal” if you trust headers blindly.  
2. **Leftmost XFF is client-controlled** — Echo’s XFF extractor walks from the **right** among trusted hops; don’t invent your own `strings.Split` and take `[0]`.  
3. **Loopback lies during local demo** — `curl` to `:1323` often shows `127.0.0.1` / `::1`, which *is* private — great for a green path, bad if you forget production sits behind a LB.  
4. **“Inside the building” ≠ “logged in”** — IP/header checks are a **weak signal**. Prefer them for soft UX (extra metrics, friendlier errors), not as your only authn.  
5. **Middleware that does DNS or DB on every request** — don’t. Keep the check O(headers + parse).

**Private ranges (stdlib):** `net.ParseIP(ip).IsPrivate()` (and loopback via `IsLoopback()`). That’s enough for today’s lab definition of “inside.”

**Pulse check 4 (≤60s):** Trust `X-Forwarded-For` from a random client? **Yes / No.** Expected: **No.** Yes only when a **trusted** proxy in *your* infra appends hops and the edge strips client-supplied XFF.

### 6. Bridge into practice (~2m)

Say:

> “Activity 1: get the hallway compiling — Recover, RequestID, one custom header middleware, `/whoami` printing RealIP. Activity 2: decide inside/outside and name the traps. Lab: harden one sharp edge. Break in twenty.”

---

## Activity 1 (20 min) — Hallway MVP

**Goal:** A running Echo server with real middleware registration.  
**Artifact:** repo (or folder) with `main.go` that boots on `:1323`.  
**MVP:** custom middleware sets one response header; `GET /whoami` returns JSON `{ "ip": "..." }`.  
**Visible checkpoint:** `curl -i localhost:1323/whoami` shows your header + JSON.

| | |
| --- | --- |
| **Next action** | `mkdir` / `go mod init` → paste skeleton → `go run .` |
| **Done when** | `/whoami` returns 200 JSON with `ip`, and response includes your custom header. |
| **≤2m next** | `curl -i localhost:1323/whoami` and paste the `ip` value into notes. |

### Steps

1. Create a module (solo; remote-friendly):

   ```bash
   mkdir -p ~/acs4210-day5-middleware && cd ~/acs4210-day5-middleware
   go mod init github.com/YOU/acs4210-day5-middleware
   ```

2. Add Echo v5:

   ```bash
   go get github.com/labstack/echo/v5@v5.3.1
   ```

3. Write `main.go` with:
   - `e.Use(middleware.Recover())`
   - `e.Use(middleware.RequestID())`
   - one custom middleware that sets a response header (copy `ServerHeader` from TT)
   - `e.IPExtractor = echo.ExtractIPDirect()` for local honesty
   - `GET /whoami` → `c.JSON(200, map[string]any{"ip": c.RealIP()})`
4. Run: `go run .`
5. Verify:

   ```bash
   curl -i http://127.0.0.1:1323/whoami
   ```

### Checkpoint paste

```text
Built: hallway MVP (Recover + RequestID + custom header + /whoami)
Verified: curl -i → header present, ip=
Blocked by:
Next: Activity 2 inside-the-building decision
```

### Stretch (only if MVP is green)

- Add `middleware.RequestLogger()` and watch one access line per curl.  
- Register `TraceName("A")` then `TraceName("B")` and predict log order before running.

---

## BREAK (10 min)

Stand up. Leave the server running if you want — or kill it and restart after break.  
**≤2m next when back:** open Activity 2; don’t invent a new repo.

---

## Activity 2 (30 min) — Inside the Building

**Goal:** Custom middleware (or helper used by middleware) that labels a request as inside/outside.  
**Artifact:** `/whoami` JSON includes `inside` (bool) + `via` (how you decided) + `ip`.  
**MVP:** treat **loopback + private** (`IsLoopback` / `IsPrivate`) as inside when using `ExtractIPDirect`.  
**Visible checkpoint:** two curls — normal local (inside true) vs forged header still inside/outside per **your documented rules**.

| | |
| --- | --- |
| **Next action** | Add `InsideTheBuilding` middleware → set `c.Set("inside", bool)` → read it in `/whoami`. |
| **Done when** | JSON shows `inside` + `via`; you can explain one spoof attempt you tried. |
| **≤2m next** | Run the two curls in the verify section; write one sentence on which trap you hit. |

### Why this lab exists (90 seconds, then build)

People want “inside the building” for: office-only admin UIs, friendlier errors on corp network, metrics tagged `campus=true`, soft allow-lists in front of stronger auth.  
**Technique options (pick one primary):**

1. **Network identity** — `c.RealIP()` + private/loopback (today’s default).  
2. **Explicit header from *your* edge** — e.g. `X-Campus: 1` set only by a trusted reverse proxy that **strips** inbound copies (production pattern; don’t trust the browser).  
3. **mTLS / VPN identity** — out of scope for today’s clock; name it as the grown-up upgrade path.

### Skeleton (adapt; don’t invent APIs)

```go
package main

import (
	"net"
	"net/http"

	"github.com/labstack/echo/v5"
	"github.com/labstack/echo/v5/middleware"
)

func InsideTheBuilding(next echo.HandlerFunc) echo.HandlerFunc {
	return func(c *echo.Context) error {
		ipStr := c.RealIP()
		ip := net.ParseIP(ipStr)
		inside := false
		via := "unparseable"
		if ip != nil {
			inside = ip.IsLoopback() || ip.IsPrivate()
			via = "realip+private/loopback"
		}
		c.Set("inside", inside)
		c.Set("via", via)
		c.Response().Header().Set("X-Inside-The-Building", map[bool]string{true: "1", false: "0"}[inside])
		return next(c)
	}
}

func main() {
	e := echo.New()
	// Local/dev: trust the network peer only.
	e.IPExtractor = echo.ExtractIPDirect()

	e.Use(middleware.Recover())
	e.Use(middleware.RequestID())
	e.Use(InsideTheBuilding)

	e.GET("/whoami", func(c *echo.Context) error {
		inside, _ := c.Get("inside").(bool)
		via, _ := c.Get("via").(string)
		return c.JSON(http.StatusOK, map[string]any{
			"ip":     c.RealIP(),
			"inside": inside,
			"via":    via,
		})
	})

	if err := e.Start(":1323"); err != nil {
		e.Logger.Error("failed to start server", "error", err)
	}
}
```

### Steps

1. Keep Activity 1’s module. Set `e.IPExtractor = echo.ExtractIPDirect()`.  
2. Add `InsideTheBuilding` as above (or equivalent). Register it **after** Recover/RequestID.  
3. Extend `/whoami` to return `inside` and `via`.  
4. **Verify green path:**

   ```bash
   curl -s http://127.0.0.1:1323/whoami
   # expect inside=true (loopback), via mentions realip
   ```

5. **Verify spoof attempt (header should NOT flip the answer under ExtractIPDirect):**

   ```bash
   curl -s -H 'X-Forwarded-For: 8.8.8.8' http://127.0.0.1:1323/whoami
   curl -s -H 'X-Real-IP: 8.8.8.8' http://127.0.0.1:1323/whoami
   ```

6. **Optional contrast (only if you have time — and comment it):** temporarily set  
   `e.IPExtractor = echo.ExtractIPFromXFFHeader()`  
   re-run the forged XFF curl, observe the change, then **put `ExtractIPDirect` back** and write one sentence: *when would trusting XFF be correct?* (Answer: when a **trusted** proxy in *your* infra appends hops and the edge strips client-supplied XFF.)

### Checkpoint paste

```text
Built: InsideTheBuilding middleware + /whoami {ip,inside,via}
Verified: local inside= ; forged XFF inside= ; extractor=
Blocked by:
Next: Lab polish / stretch
Trap I can explain:
```

### Hard no for this activity

- Do not treat spoofable headers as auth.  
- Do not put DB lookups in this middleware.  
- Do not block the handler with a 403 unless you **intentionally** chose “enforce” mode and documented it — today’s default is **label + header + JSON**, not a lock.

---

## Lab Time (10 min)

**Next action:** Pick one stretch; ship the checkpoint.  
**Done when:** `/whoami` still works and you improved one sharp edge.  
**≤2m next:** Choose from the list — don’t start all three.

1. **Enforce mode** — if `inside == false`, return `echo.NewHTTPError(http.StatusForbidden, "outside the building")` on a nested group `e.Group("/internal", InsideTheBuilding)` instead of globally.  
2. **Trusted-header path** — read `X-Campus` **only after** documenting “proxy must strip inbound”; never enable it with the default extractor.  
3. **Order demo** — add two tiny middlewares that log enter/leave; screenshot or paste the order proving outer→inner→handler→inner→outer.

---

## Wrap Up (5 min)

**GOAL check:** You can say, in one breath:

1. Middleware = cross-cutting hallway; handlers = route work.  
2. `Pre` before router; `Use` after match; first registered is outermost.  
3. Recover → ID/log → security → auth → feature tags → handler.  
4. `RealIP` is only as honest as `IPExtractor`.  
5. “Inside the building” is a **signal**, not a substitute for real auth.

**≤2m next after session:** Commit the lab. Tomorrow, every time you see an `e.Use`, ask “why this layer?”

### After-session stretch (optional)

- Read Echo’s [Custom Middleware cookbook](https://echo.labstack.com/cookbook/middleware/) end-to-end (`Stats` + `ServerHeader`).  
- Read [IP Address](https://echo.labstack.com/guide/ip-address/) — especially the XFF “from the right” diagram.  
- Sketch where middleware would live in your API service (logging + request ID first).

---

## Additional Resources

1. **[Echo — Custom Middleware cookbook](https://echo.labstack.com/cookbook/middleware/)** — official `Stats` + `ServerHeader` patterns (`echo/v5`).  
2. **[Echo — IP Address guide](https://echo.labstack.com/guide/ip-address/)** — `RealIP`, `IPExtractor`, XFF vs X-Real-IP, trust options.  
3. **[Echo — Context](https://echo.labstack.com/guide/context/)** — `Get`/`Set`, request/response helpers.  
4. **[Echo — Request Logger middleware](https://echo.labstack.com/middleware/logger/)** — `RequestLogger` configuration (v5 logging path).  
5. **[Echo v5 API changes](https://github.com/labstack/echo/blob/v5/API_CHANGES_V5.md)** — `*echo.Context`, removed `Logger`/`Timeout` middleware exports.  
6. **[Go — `net.IP.IsPrivate`](https://pkg.go.dev/net#IP.IsPrivate)** — private network detection for the lab.  
7. **[go.dev — Writing Web Applications](https://go.dev/doc/articles/wiki/)** — baseline HTTP mental model if anyone needs to zoom out from Echo.

---

<details>
<summary>For curriculum authors</summary>

## For curriculum authors

<details>
<summary>For curriculum authors</summary>


### Run-of-show (whole session)

| | |
| --- | --- |
| **Next action** | Open this file → skim the agenda table → start Why / Objectives at 1:00. |
| **Done when** | You have a running Echo app with custom middleware + an “inside the building” decision you can demo with `curl`. |
| **≤2m next** | Paste the Feeling check-in into notes: Feeling / Behind\|On track\|Ahead / Today’s MVP. |

```text
Feeling (1 word):
Behind | On track | Ahead:
Today's MVP (1 sentence):
```

### Facilitator notes

- Prefer speakable TT. Behind at ~0:35? Skip the Stats aside — jump to IP traps → Activity 1.  
- Solo lab by design. Optional after Activity 2: 60s compare of `via` strings.  
- Keep all four pulse checks; they replace digressions.  
- Module on **echo/v4**? Keep walking the onion; swap to `echo.Context` + `middleware.Logger()`. No mid-session major bump unless they’re unblocked.  
- **Go gate (say once):** Echo **v5** → **Go ≥ 1.25**; otherwise stay on **v4**.

</details>

</details>
