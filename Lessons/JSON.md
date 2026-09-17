<!-- Run as a slideshow: reveal-md Lessons/JSON.md -w -->
# Working with JSON — Day 6

⭐️ **GOAL:** Leave able to marshal Go values to JSON (and read them back), customize struct field names with tags, and write scrape results to a file as JSON.

<!-- omit in toc -->
## ⏱ Agenda

- [[**15m**] ☀️ Warm Up](#15m-️-warm-up)
- [[**35m**] 📚 TT: Serialization and encoding/json](#35m--tt-serialization-and-encodingjson)
- [[**10m**] 🌴 Break](#10m--break)
- [[**25m**] 💻 Activity 1: Marshal Basics Slices and Maps](#25m--activity-1-marshal-basics-slices-and-maps)
- [[**30m**] 💻 Activity 2: Struct Tags and Write a JSON File](#30m--activity-2-struct-tags-and-write-a-json-file)
- [[**5m**] Wrap Up](#5m-wrap-up)

<!-- > -->

<!-- omit in toc -->
## 🏆 Objectives

*By the end of this session, you'll be able to&hellip;*

1. Explain serialization in plain language (store or transmit, then reconstruct)
1. Marshal bools, numbers, strings, slices, and maps with `encoding/json`
1. Encode a struct with `json` tags and write the bytes to a file with `os`

<!-- > -->

## [**15m**] ☀️ Warm Up

In the web scraper project, one requirement is to encode the struct that stores scraped data into JSON, then save it to a file.

You already know how to write files with the `os` package. Today is the encode half — and how tags control the JSON keys.

**Warm-up prompt:** In one sentence, what problem does JSON solve between a Go struct in memory and a `.json` file on disk?

> **📈 TIP:** JSON is text. If you can print the marshaled string and it looks right, you are halfway to a durable scrape export.

<!-- > -->

## [**35m**] 📚 TT: Serialization and encoding/json

### 1. What Serialization Is (~6m)

From [Wikipedia — Serialization](https://en.wikipedia.org/wiki/Serialization): translating data structures or object state into a format that can be **stored** or **transmitted**, then **reconstructed later** (maybe on another machine).

We serialize our scrape struct into JSON so the results outlive the process.

> **💬 ASK QUESTION:** Is writing a struct with `fmt.Fprintf` “serialization”? Why or why not?

<details>
<summary>Answer</summary>

**Weak / incomplete.** `fmt` prints a Go-facing dump, not a portable agreed format. JSON (or another codec) gives a format other tools and languages can read back.

</details>

### 2. A Short Analogy (~4m)

You have an idea at home. Your teammate is elsewhere. You write an email and send it.

You **serialized** the idea into a message that can be transmitted and stored. When they read it, they **deserialize** it into their own understanding.

Same job for a scrape struct → JSON bytes → file or HTTP body → someone else’s `Unmarshal`.

### 3. Package (~2m)

```go
import "encoding/json"
```

Primary calls today: `json.Marshal` / `json.Unmarshal` (and later `Encoder` / `Decoder` for streams).

> **‼️ WATCH OUT:** Ignoring the error from `Marshal` (`val, _ := json.Marshal(...)`) hides failures. Check `err` before you trust the bytes.

### 4. Basic Data Types (~7m)

```go
package main

import (
  "encoding/json"
  "fmt"
)

func main() {
  aBoolValue, err := json.Marshal(true)
  if err != nil {
    panic(err)
  }
  fmt.Println(string(aBoolValue))

  anIntValue, err := json.Marshal(1)
  if err != nil {
    panic(err)
  }
  fmt.Println(string(anIntValue))

  aFloatValue, err := json.Marshal(2.34)
  if err != nil {
    panic(err)
  }
  fmt.Println(string(aFloatValue))

  aStringValue, err := json.Marshal("ACS-4210")
  if err != nil {
    panic(err)
  }
  fmt.Println(string(aStringValue))
}
```

Predict each `Println` before you run.

> **💬 QUICK CHECK:** What does `json.Marshal("hello")` print — with or without quotes in the output string?

<details>
<summary>Answer</summary>

JSON strings are quoted. You should see `"hello"` including the quote characters inside the printed text.

</details>

### 5. Slices and Maps (~7m)

```go
fruitSlice := []string{"apple", "peach", "pear"}
fruitJSON, err := json.Marshal(fruitSlice)
if err != nil {
  panic(err)
}
fmt.Println(string(fruitJSON))

totalFruitsMap := map[string]int{"apple": 5, "lettuce": 7}
totalFruitsJSON, err := json.Marshal(totalFruitsMap)
if err != nil {
  panic(err)
}
fmt.Println(string(totalFruitsJSON))
```

Predict outputs. Note: map key order in printed JSON is not something to depend on for tests.

### 6. Structs and Tags (~9m)

Use tags on struct fields to customize encoded key names:

```go
type FruitList struct {
  Page   int      `json:"page"`
  Fruits []string `json:"fruits"`
}

fruitList := &FruitList{
  Page:   1,
  Fruits: []string{"apple", "peach", "pear"},
}
fruitJSON, err := json.Marshal(fruitList)
if err != nil {
  panic(err)
}
fmt.Println(string(fruitJSON))
```

> **💬 YOUR TURN:** What changes if you remove the `` `json:"page"` `` / `` `json:"fruits"` `` tags?

<details>
<summary>Answer</summary>

Keys fall back to the exported field names (`Page`, `Fruits`) — not the lowercase JSON style most APIs expect.

</details>

> **📈 DO THIS:** Prefer lowercase JSON keys via tags for anything a browser, Python script, or scrape consumer will read.

<!-- > -->

## [**10m**] 🌴 Break

<!-- > -->

## [**25m**] 💻 Activity 1: Marshal Basics Slices and Maps

> **✅ DONE WHEN:** A `go run .` program prints marshaled bool, int, float, string, one slice, and one map — each on its own line — and every `Marshal` call checks `err`.

1. Create a scratch module (`go mod init json-day6` or a throwaway folder with `main.go`).
1. Paste the basic-types example. Replace `_` with real `err` checks.
1. Add the slice + map examples. Run. Compare to your predictions.
1. Change one map value and re-run. Confirm the JSON text changed.

> **📈 SHORTCUT:** `string(bytes)` is the fast way to eyeball marshaled output in the terminal.
> **FINISHED EARLY?** `json.Unmarshal` the slice JSON back into a `[]string` and print `len`.

<!-- > -->

## [**30m**] 💻 Activity 2: Struct Tags and Write a JSON File

> **✅ SHIP WHEN:** You have a tagged struct, `json.Marshal` succeeds, and `os.WriteFile` (or create/write) saves pretty or compact JSON to disk; you can open the file and see your keys.

1. Define a scrape-shaped struct (name it for your project — e.g. page title, URL, items slice).
1. Add `json:"..."` tags for every exported field you want in the file.
1. Marshal to `[]byte`. Write to `out.json` with `0644` (or your team’s usual perms).
1. Open the file. Confirm keys match tags, not Go field names.

> **‼️ CAUTION:** `Marshal` does not indent. For a readable file use `json.MarshalIndent(v, "", "  ")` when you want humans to skim it.
> **FINISHED EARLY?** Round-trip: read the file, `Unmarshal` into a fresh struct, print one field.

<!-- > -->

## [**5m**] Wrap Up

1. Serialization = store or send, then rebuild later.
1. `encoding/json` marshals Go values; tags control keys.
1. Check `err`. Write bytes to disk for the scraper export path.

<!-- > -->

## Additional Resources

1. **[Go by Example — JSON](https://gobyexample.com/json)** — marshal / unmarshal patterns.
1. **[encoding/json package docs](https://pkg.go.dev/encoding/json)** — `Marshal`, tags, `Encoder`.
1. **[Wikipedia — Serialization](https://en.wikipedia.org/wiki/Serialization)** — definition used in TT.

<details>
<summary>For Curriculum Authors</summary>

## For Curriculum Authors

### In Class

| | |
| --- | --- |
| **Next action** | Open this file → Warm Up → TT §1. |
| **Done when** | Room has marshaled a tagged struct to a file at least once. |

- Continuity: web scraper export requirement; SSG lab time from the old plan can sit in Activity 2 FINISHED EARLY or next session.
- Old lesson had a 20m group worksheet review and a 50m open lab — replaced with two titled Activities so the encode path has a clear done-state.

### Facilitator Notes

- Keep TT examples runnable; prefer `err` checks over `_`.
- Solo-capable Activities; no required breakouts.

### Expert Follow-Ups

- Optional later: streaming `json.Encoder` for large scrapes; `omitempty` tags.

</details>
