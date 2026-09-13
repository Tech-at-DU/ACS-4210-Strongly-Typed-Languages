<!-- Run as a slideshow: reveal-md Lessons/WebScraping.md -w -->
# Scraping the Web — Day 6

⭐️ **GOAL:** Leave able to tell scrape from crawl, extract fields with Colly `OnHTML` + goquery selectors, and marshal a struct slice to `output.json` with `encoding/json`.

<!-- omit in toc -->
## ⏱ Agenda

- [[**5m**] Attendance &amp; Announcements](#5m-attendance--announcements)
- [[**15m**] ☀️ Warm Up](#15m-️-warm-up)
- [[**35m**] 📚 TT: Overview](#35m--tt-overview)
- [[**10m**] 🌴 Break](#10m--break)
- [[**20m**] 💻 Activity 1](#20m--activity-1)
- [[**30m**] 💻 Activity 2](#30m--activity-2)
- [[**5m**] Wrap Up](#5m-wrap-up)

<!-- > -->

<!-- omit in toc -->
## 🏆 Objectives

*By the end of this block, you'll be able to&hellip;*

1. **Define** scrape vs crawl and name when HTML extraction is the wrong tool (use an API / JSON feed first).
2. **Test** a CSS/goquery selector in DevTools before putting it in `OnHTML` — and say why a copied `nth-child` path is brittle.
3. **Implement** a Colly collector (`NewCollector`, `AllowedDomains`, `OnRequest`, `OnHTML`, `OnError`, `Visit`) that prints real fields from a page.
4. **Build** a `struct` with `json` tags, `json.MarshalIndent` a slice, and write `output.json` (`os.WriteFile` or `json.NewEncoder`).

**How you’ll know:** `go run .` prints quote text + author from [quotes.toscrape.com](https://quotes.toscrape.com/), and Activity 2 leaves a pretty-printed `output.json` you can `Unmarshal` back into the same type.

<!-- > -->

## [**5m**] Attendance &amp; Announcements

<!-- > -->

## [**15m**] ☀️ Warm Up

A scraper is a **job-shaped HTTP client**: fetch HTML, pick fields with selectors, emit structured data (today: JSON). A crawler is the **follow-every-link** cousin. Most production work is a scraper with a *tight* crawl (same host, max depth, delay).

If you can ship this well, you can:

- pull fields a site never put in an API
- archive a page before it disappears
- feed a struct you already know how to persist
- spot rookie traps: unbounded `a[href]` visits, trusting a DevTools “Copy selector” path, ignoring `robots.txt`, or treating JS-only DOM as if Colly rendered it

**On-the-job prompt (think → chat → share):** You’re reviewing a PR that `Visit`s every `a[href]` with no `AllowedDomains`, no `MaxDepth`, and no delay. In one sentence each: what is the blast radius, and what three guards do you require before merge?

Whiteboard / screen three buckets if chat stays quiet:

| Belongs in the collector | Belongs in your struct / JSON | Do not ship |
| --- | --- | --- |
| `AllowedDomains` + `MaxDepth` | Exported fields + `json` tags | Visit every `a[href]` on the public web |
| `OnHTML` for *known* cards | `MarshalIndent` / `Encode` to a file | “Copy selector” `nth-child` paths you never tested |
| `OnError` + `OnRequest` logs | Round-trip `Unmarshal` to prove the shape | Trusting Colly’s default `IgnoreRobotsTxt = true` on a real host |
| `Limit` (`DomainGlob` + `Delay`) | — | Scraping JS-only fields Colly never saw |

**Rookie tip for on-the-job success:** DevTools → Inspect → Copy → Copy selector is a *hint*, not a contract. Paste the selector into the Elements panel find box (`Ctrl`/`Cmd`+`F`). If it does not highlight the node you meant, Colly will not extract it either. Prefer stable `#id`, `.class`, and `[attr]` selectors.

Open [quotes.toscrape.com](https://quotes.toscrape.com/) and, in two minutes, write down selectors for:

1. one quote card
2. the quote text
3. the author
4. (stretch) the “Next” pager link

<!-- > -->

## [**35m**] 📚 TT: Overview

**Next action:** Run this talk track — GOAL first, then selectors, then Colly, then JSON.  
**Done when:** You can say callback order out loud and write a `struct` that marshals to the keys you want.  
**≤2m next after TT:** Open Activity 1 and create the module.

### 1. GOAL — HTML in, JSON out (~5m)

Say:

> “If the site already publishes JSON, take the JSON. Scraping is for HTML you have permission and a reason to parse. Colly fetches the **bytes the server sent**. It does not run JavaScript. `encoding/json` is how you leave with something the rest of the stack can read.”

| Job | What it does | Typical output |
| --- | --- | --- |
| **Crawl** | Walk links, enqueue more URLs | A frontier of pages |
| **Scrape** | Extract *named* fields from a known layout | Rows / structs |
| **Serialize** | Encode those structs | `output.json` (today) |

**Extract → transform → load** still applies: HTML is the extract, the struct is the transform, the file (or API) is the load.

> **ASK AUDIENCE:** You need the price and SKU from *one* product template. Do you write a crawler or a scraper?

<details>
<summary>Answer</summary>

**Scraper.** You already know the card. A crawler is only the extra hop if you must discover URLs first — and even then you cap host, depth, and rate.

</details>

### 2. Selectors you can defend in review (~7m)

Colly’s `OnHTML` first argument is a **goquery** selector (CSS as implemented by [cascadia](https://github.com/andybalholm/cascadia) / [goquery](https://github.com/PuerkitoBio/goquery)). Same mental model as DevTools.

Verified against today’s [quotes.toscrape.com](https://quotes.toscrape.com/) markup:

| You want | Selector | Why it is stable |
| --- | --- | --- |
| Quote card | `div.quote` | Class on the repeating block |
| Text | `span.text` (or `div.quote` + `ChildText("span.text")`) | Class on the text node |
| Author | `small.author` | Class on the author node |
| Tags | `a.tag` | Class on each tag link |
| Next page | `li.next a` | Pager item + the `href` |

Common selector shapes (goquery / CSS):

| Kind | Syntax | Matches |
| --- | --- | --- |
| Element | `a` | Any `a` |
| ID | `#login` | `id="login"` |
| Class | `.quote` | `class="quote"` (among others) |
| Attribute | `a[href]` | `a` that has `href` |
| Descendant | `div.quote small.author` | Author inside a card |
| Pseudo-class | `td:nth-of-type(1)` | First `td` in its parent |

Live-demo (or paste a screenshot): Inspect a `.quote` → find `span.text` → search `div.quote span.text` in the panel. Contrast with a copied `body > div > ... > span:nth-child(1)` path. The long path breaks when the header adds a div.

**Colly cannot see a shadow DOM or a client-rendered tree.** If View Source does not contain the string, `OnHTML` will not either. Headless browsers are a different tool — out of scope for this clock.

**Rookie tip for on-the-job success:** one `OnHTML` per *card*, then `ChildText` / `ChildAttr` / `ChildTexts` for fields inside it. Do not register twelve top-level callbacks that each walk the whole document.

### 3. Colly — collector, callbacks, guards (~9m)

Install / import per the current Colly README (the site’s getting-started page still shows the pre-module path — **use `/v2`**):

```go
import "github.com/gocolly/colly/v2"

c := colly.NewCollector()
```

```bash
go get github.com/gocolly/colly/v2@v2.3.0
```

⚠️ **Toolchain gate (say once):** Colly **v2.3.0** needs **Go ≥ 1.24** (`go version`). Below that, either upgrade Go or pin an older `v2` tag that your toolchain accepts — ideas are identical; do not invent APIs. Prefer the pin above (not `@latest` on the day you run this).

`Collector` owns HTTP and runs the callbacks you attach. Official callback order ([Getting started](https://go-colly.org/docs/introduction/start/)):

| # | Callback | When |
| --- | --- | --- |
| 1 | `OnRequest` | Before the request goes out |
| 2 | `OnError` | Request failed |
| 3 | `OnResponseHeaders` | Headers arrived |
| 4 | `OnResponse` | Body arrived |
| 5 | `OnHTML` | Body is HTML — **per matching element** |
| 6 | `OnXML` | After `OnHTML`, if HTML or XML |
| 7 | `OnScraped` | After `OnXML` for that response |

`Visit` starts the job. `HTMLElement` helpers you will actually call (current `htmlelement.go`):

| API | Returns |
| --- | --- |
| `e.Text` | Text of the matched node |
| `e.Attr("href")` | Attribute, or `""` |
| `e.ChildText("span.text")` | Trimmed concatenated text of descendants |
| `e.ChildTexts("a.tag")` | `[]string` — one trimmed text per match |
| `e.ChildAttr("a", "href")` | First descendant’s attribute |
| `e.Request.Visit(href)` | Enqueue URL; **resolves relative hrefs** via `AbsoluteURL` |
| `e.DOM` | `*goquery.Selection` if you need `Find` / `Each` |

Guards that belong in every review:

| API | What it does |
| --- | --- |
| `colly.AllowedDomains("quotes.toscrape.com")` | Domain whitelist (empty = any host) |
| `colly.MaxDepth(n)` | `0` = unlimited; reject when `depth > n` |
| `colly.UserAgent("…")` | Override default `"colly - https://github.com/gocolly/colly"` |
| `c.Limit(&colly.LimitRule{DomainGlob: "*quotes.toscrape.*", Delay: 200 * time.Millisecond, Parallelism: 1})` | Requires `DomainGlob` **or** `DomainRegexp` |
| `c.IgnoreRobotsTxt = false` | **Not the default.** `Init` sets `IgnoreRobotsTxt = true` |

`colly.Async()` (or `colly.Async(true)`) makes `Visit` return before work finishes — then you must `c.Wait()`. Stay **sync** today so a slice append in `OnHTML` does not need a mutex.

> **ASK AUDIENCE:** For one HTML page, which fires first — `OnHTML` or `OnScraped`?

<details>
<summary>Answer</summary>

**`OnHTML`.** Official order is `OnResponse` → `OnHTML` → `OnXML` → `OnScraped`. `OnScraped` is “this response is done,” not “here is a node.”

</details>

> **ASK AUDIENCE:** Does `colly.NewCollector()` honor `robots.txt` by default?

<details>
<summary>Answer</summary>

**No.** `Collector.Init` sets `IgnoreRobotsTxt = true`. The option `colly.IgnoreRobotsTxt()` turns that flag *on* (already the default). To honor the file, set `c.IgnoreRobotsTxt = false` after `NewCollector`. There is no `RespectRobotsTxt()` helper.

</details>

### 4. `encoding/json` — the ship format (~8m)

Package: `"encoding/json"` ([pkg.go.dev](https://pkg.go.dev/encoding/json)). Go 1.27 also shipped `encoding/json/v2`; this block uses **v1** (`encoding/json`) — still supported, same APIs as the scraper project.

```go
func Marshal(v any) ([]byte, error)
func MarshalIndent(v any, prefix, indent string) ([]byte, error)
func Unmarshal(data []byte, v any) error
func NewEncoder(w io.Writer) *Encoder
func NewDecoder(r io.Reader) *Decoder
```

Only **exported** fields participate. Tags rename keys and can drop fields:

```go
type Quote struct {
    Text   string   `json:"text"`
    Author string   `json:"author"`
    Tags   []string `json:"tags,omitempty"`
    Debug  string   `json:"-"`
}
```

| Tag | Effect |
| --- | --- |
| `` `json:"text"` `` | Key is `"text"`, not `"Text"` |
| `` `json:"tags,omitempty"` `` | Omit if empty (v1: empty slice/string/zero) |
| `` `json:"-"` `` | Never encode or decode this field |

Round-trip (must pass a **pointer** to `Unmarshal`):

```go
q := Quote{Text: "hello", Author: "Ada"}
b, err := json.MarshalIndent(q, "", "  ")
if err != nil {
    log.Fatal(err)
}
fmt.Println(string(b))

var back Quote
if err := json.Unmarshal(b, &back); err != nil {
    log.Fatal(err)
}
fmt.Println(back.Author)
```

File write — two official paths; pick one:

```go
// A) bytes + WriteFile
b, err := json.MarshalIndent(quotes, "", "  ")
if err != nil {
    log.Fatal(err)
}
if err := os.WriteFile("output.json", b, 0o644); err != nil {
    log.Fatal(err)
}

// B) stream to a file
f, err := os.Create("output.json")
if err != nil {
    log.Fatal(err)
}
defer f.Close()
enc := json.NewEncoder(f)
enc.SetIndent("", "  ")
if err := enc.Encode(quotes); err != nil {
    log.Fatal(err)
}
```

> **ASK AUDIENCE:** `json.Unmarshal(b, back)` — missing `&`. What happens?

<details>
<summary>Answer</summary>

**Error.** `Unmarshal` must write through a non-nil pointer (`&back`). Passing the struct value cannot update the caller’s variable.

</details>

### 5. Live skeleton — one page, real fields (~5m)

Speak while typing (or paste once, then walk). Predict: how many `div.quote` cards on page 1? (Ten.) Then run.

```go
package main

import (
    "fmt"
    "log"

    "github.com/gocolly/colly/v2"
)

func main() {
    c := colly.NewCollector(
        colly.AllowedDomains("quotes.toscrape.com"),
        colly.UserAgent("acs4210-quotes-lab/1.0"),
    )

    c.OnRequest(func(r *colly.Request) {
        fmt.Println("visiting", r.URL)
    })
    c.OnError(func(_ *colly.Response, err error) {
        log.Println("request failed:", err)
    })
    c.OnHTML("div.quote", func(e *colly.HTMLElement) {
        fmt.Printf("%s — %s\n", e.ChildText("span.text"), e.ChildText("small.author"))
    })

    if err := c.Visit("https://quotes.toscrape.com/"); err != nil {
        log.Fatal(err)
    }
}
```

**Predict-then-run:** First log line is `visiting https://quotes.toscrape.com/` (`OnRequest` before `OnHTML`). Then ten `text — author` lines.

### 6. Bridge into practice (~1m)

Say:

> “Activity 1: get that collector compiling and printing ten quotes. Activity 2: collect into a `[]Quote`, marshal, write `output.json`, prove it with `Unmarshal`. Stretch inside Activity 2 if you finish early: follow `li.next a` behind `MaxDepth`. Break next.”

<!-- > -->

## [**10m**] 🌴 Break

Stand up. Leave the tab on quotes.toscrape.com open — you will reuse those selectors.  
**≤2m next when back:** open Activity 1 if you haven’t started; don’t invent a new repo after Activity 1.

<!-- > -->

## [**20m**] 💻 Activity 1

**Goal:** A running Colly collector that extracts real fields.  
**Artifact:** repo (or folder) with `main.go` that visits quotes.toscrape.com.  
**MVP:** `OnHTML("div.quote")` prints text + author; `AllowedDomains` is set; `OnError` is registered.  
**Visible checkpoint:** `go run .` prints `visiting …` then ten `text — author` lines.

| | |
| --- | --- |
| **Next action** | `mkdir` / `go mod init` → paste TT skeleton → `go run .` |
| **Done when** | Ten quotes print and a bad selector prints *nothing* (you tried one). |
| **≤2m next** | Paste one printed line into notes; keep the module for Activity 2. |

### Steps

1. Create a module (solo; remote-friendly):

   ```bash
   mkdir -p ~/acs4210-day6-scrape && cd ~/acs4210-day6-scrape
   go mod init github.com/YOU/acs4210-day6-scrape
   ```

2. Add Colly v2:

   ```bash
   go get github.com/gocolly/colly/v2@v2.3.0
   ```

3. Write `main.go` from the TT skeleton (`AllowedDomains`, `OnRequest`, `OnError`, `OnHTML` + `ChildText`, `Visit`).
4. Run: `go run .`
5. **Break the selector on purpose** (e.g. `div.quotez`) → rerun → confirm silence → put `div.quote` back. That is the DevTools lesson in code.

### Checkpoint paste

```text
Built: Colly MVP (AllowedDomains + OnHTML div.quote + ChildText)
Verified: go run . → 10 lines; author of line 1=
Broken selector I tried:
Next: Activity 2 struct + output.json
```

### Stretch (only if MVP is green)

- Print `e.ChildTexts("a.tag")` next to each quote.  
- Add `c.OnScraped` and confirm it logs *once per page*, after the quote lines.

If you finish early, help a peer who’s stuck.

<!-- > -->

## [**30m**] 💻 Activity 2

**Goal:** Same collector, ship JSON.  
**Artifact:** `output.json` in the module root.  
**MVP:** `[]Quote` with `json` tags; `MarshalIndent`; write the file; print the byte length.  
**Visible checkpoint:** `head -c 200 output.json` shows `"text"` / `"author"` keys (not `"Text"` / `"Author"`).

| | |
| --- | --- |
| **Next action** | Add `Quote` + append in `OnHTML` → marshal after `Visit` returns. |
| **Done when** | `output.json` exists; you `Unmarshal` it back into `[]Quote` and print `len`. |
| **≤2m next** | Open the file; write one sentence on which tag changed the keys. |

### Why this lab exists (60 seconds, then build)

The course scraper project asks for the same pipeline: **struct → `OnHTML` → JSON → `output.json`**. Today you run that pipeline on a site that *wants* to be scraped so the project is muscle memory, not archaeology.

### Skeleton (adapt; don’t invent APIs)

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "os"

    "github.com/gocolly/colly/v2"
)

type Quote struct {
    Text   string   `json:"text"`
    Author string   `json:"author"`
    Tags   []string `json:"tags,omitempty"`
}

func main() {
    var quotes []Quote

    c := colly.NewCollector(
        colly.AllowedDomains("quotes.toscrape.com"),
        colly.UserAgent("acs4210-quotes-lab/1.0"),
    )

    c.OnError(func(_ *colly.Response, err error) {
        log.Println("request failed:", err)
    })
    c.OnHTML("div.quote", func(e *colly.HTMLElement) {
        quotes = append(quotes, Quote{
            Text:   e.ChildText("span.text"),
            Author: e.ChildText("small.author"),
            Tags:   e.ChildTexts("a.tag"),
        })
    })

    if err := c.Visit("https://quotes.toscrape.com/"); err != nil {
        log.Fatal(err)
    }

    data, err := json.MarshalIndent(quotes, "", "  ")
    if err != nil {
        log.Fatal(err)
    }
    if err := os.WriteFile("output.json", data, 0o644); err != nil {
        log.Fatal(err)
    }

    var back []Quote
    if err := json.Unmarshal(data, &back); err != nil {
        log.Fatal(err)
    }
    fmt.Printf("wrote output.json (%d bytes, %d quotes)\n", len(data), len(back))
}
```

### Steps

1. Keep Activity 1’s module. Add the `Quote` type and the append / marshal / write / unmarshal block.  
2. Run: `go run .`  
3. **Verify keys:**

   ```bash
   head -c 200 output.json
   # expect "text" and "author", ten objects on page 1
   ```

4. **Verify round-trip:** the printed count is `10`.  
5. **Optional contrast:** temporarily drop the `json` tags, rerun, watch keys become `"Text"` / `"Author"`, then **put the tags back**.

### Checkpoint paste

```text
Built: []Quote + MarshalIndent + output.json + Unmarshal
Verified: bytes= ; count= ; keys=
Blocked by:
Next: Lab polish / stretch
Tag I can explain:
```

### Hard no for this activity

- Do not `Visit` every `a[href]` on the page (login, author, tag, next — you will wander).  
- Do not switch on `Async` and append to `quotes` without a lock.  
- Do not scrape a random production site from this block. Stay on quotes.toscrape.com.

### Stretch (pick one if MVP is green)

**Done when:** `output.json` still validates and you improved one sharp edge. Don’t start all three.

1. **Pager** — second `OnHTML("li.next a", …)` that `e.Request.Visit(e.Attr("href"))`, plus `colly.MaxDepth(2)` (page 1 is depth 1; next is depth 2). Expect ~20 quotes.  
2. **Encoder** — write with `json.NewEncoder(f)` + `SetIndent("", "  ")` + `Encode` instead of `WriteFile`.  
3. **Citizen collector** — `c.IgnoreRobotsTxt = false` and `c.Limit(&colly.LimitRule{DomainGlob: "*quotes.toscrape.*", Parallelism: 1, Delay: 200 * time.Millisecond})`. Keep `AllowedDomains`.

<!-- > -->

## [**5m**] Wrap Up

**GOAL check:** You can say, in one breath:

1. Scrape named fields; crawl only with host / depth / rate caps.  
2. Test selectors in DevTools; Colly uses goquery, not a browser JS engine.  
3. Callback order: request → response → `OnHTML` → `OnScraped`.  
4. `NewCollector` **ignores** `robots.txt` until you set `IgnoreRobotsTxt = false`.  
5. Exported fields + `json` tags + `MarshalIndent` / `Encode` → `output.json`; `Unmarshal` needs a pointer.

**≤2m next after this block:** Commit the lab. The course scraper project is the same pipeline on *your* site — worksheet selectors, struct, JSON, `output.json`.

### After-block stretch (optional)

- Read Colly’s [Getting started](https://go-colly.org/docs/introduction/start/) callback list end-to-end.  
- Skim [JSON and Go](https://go.dev/blog/json) (`Marshal` / `Unmarshal` / `Encoder`).  
- Point the same collector at **your** project URL and replace `div.quote` with selectors you tested in DevTools.

<!-- > -->

## Additional Resources

1. **[Colly — Getting started](https://go-colly.org/docs/introduction/start/)** — `NewCollector`, official callback order, `OnHTML` / `OnScraped`. Import path on this page may omit `/v2`; use `github.com/gocolly/colly/v2`.  
2. **[Colly — Configuration](https://go-colly.org/docs/introduction/configuration/)** — `NewCollector(options…)`, `UserAgent`, env vars (`COLLY_ALLOWED_DOMAINS`, `COLLY_MAX_DEPTH`, `COLLY_IGNORE_ROBOTSTXT`).  
3. **[Colly v2 API](https://pkg.go.dev/github.com/gocolly/colly/v2)** — `Collector`, `HTMLElement`, `LimitRule`, `Visit`.  
4. **[Colly examples](https://github.com/gocolly/colly/tree/master/_examples)** — basic, rate limit (`Limit` + `Async` + `Wait`).  
5. **[goquery](https://pkg.go.dev/github.com/PuerkitoBio/goquery)** — `Selection.Find`, `Text`, `Attr`, `Each` (what `e.DOM` is).  
6. **[encoding/json](https://pkg.go.dev/encoding/json)** — `Marshal`, `MarshalIndent`, `Unmarshal`, `Encoder` / `Decoder`.  
7. **[JSON and Go](https://go.dev/blog/json)** — official intro; exported fields, tags, streaming encoder.  
8. **[os.WriteFile](https://pkg.go.dev/os#WriteFile)** — write `output.json` in one call.

<details>
<summary>For curriculum authors</summary>

## For curriculum authors

### In Class

| | |
| --- | --- |
| **Next action** | Open this file → skim the Agenda jump list → start Attendance, then Warm Up. |
| **Done when** | A running Colly app prints ten quotes and Activity 2 has produced `output.json` you can open on screen. |
| **≤2m next** | Paste the Feeling check-in into notes: Feeling / Behind\|On track\|Ahead / Today’s MVP. |

```text
Feeling (1 word):
Behind | On track | Ahead:
Today's MVP (1 sentence):
```

- Warm-up is a Zoom variety beat: keep it short, memorable, and easy to explain (PR-review blast radius is fine).
- Breakouts of 3–4 if used. Visit rooms; do not dump extra facilitator direction into the body above.
- After Activity 1, debrief one failure mode in the main room (wrong selector, missing `/v2` import, `AllowedDomains` typo, network).

### Facilitator notes

- Prefer speakable TT. Behind at ~0:35? Skip the Encoder path — jump to the live skeleton → Activity 1.
- Solo lab by design. Optional after Activity 2: 60s compare of `output.json` key names.
- Keep all ASK AUDIENCE pulses; they replace digressions.
- **Import gate (say once):** current module is `github.com/gocolly/colly/v2`. go-colly.org getting-started snippets may still show `github.com/gocolly/colly`.
- **Go gate (say once):** Colly **v2.3.0** → **Go ≥ 1.24**; otherwise pin an older `v2` tag.
- **Robots default (say once):** `Init` sets `IgnoreRobotsTxt = true`. Do not invent a `RespectRobotsTxt()` API.
- quotes.toscrape.com down? Fall back to saving the homepage HTML and teaching selectors / JSON only; do not pivot to a random production host.
- Live-code the first five minutes of the skeleton only. Then get out of the way.
- Activity 2 stretch (pager / Encoder / Limit) is the early-finisher extension — not a third activity.
- `Lessons/JSON.md` and `Lessons/Lesson06.md` are pointers. Teach from this file only.
- Do not play the old headless-browser video in this block.

### Expert follow-ups

- Optional after-block: Colly examples folder; JSON and Go blog through the encoder section; point the same pipeline at the course scraper project URL.

</details>
