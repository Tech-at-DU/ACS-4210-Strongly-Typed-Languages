# 📜 Day 5: Fast Functionality Via 3rd Party Libraries

<!-- omit in toc -->
## ⏱ Agenda

1. [[**05m**] 🌤 Check-In / Entrance Ticket](#05m--check-in--entrance-ticket)
2. [[**05m**] 🏆 Objectives](#05m--objectives)
3. [[**15m**] 🎮 Warm-Up: Dependency Telephone](#15m--warm-up-dependency-telephone)
4. [[**45m**] 💬 TT: Integrating Third-Party Libraries in Go](#45m--tt-integrating-third-party-libraries-in-go)
5. [[**10m**] 🌴 BREAK](#10m--break)
6. [[**10m**] 🔍 Activity: My Favorite No (Broken `go.mod`)](#10m--activity-my-favorite-no-broken-gomod)
7. [[**45m**] 🧪 Lab Time: Solo Zoom Work](#45m--lab-time-solo-zoom-work)
8. [[**05m**] 🌃 Exit Ticket](#05m--exit-ticket)
9. [📚 Resources & Credits](#-resources--credits)

**Class shape (Fall 2026):** 2 students on Zoom in the same room. Keep discussion purposeful (short, structured), not open-ended debate. Active learning ≥20m = Warm-Up (15) + My Favorite No (10). TT stays **45m**.

Techniques from [How to Write a Self-Paced Lesson](https://docs.google.com/document/d/16O2xGeNSayfQ1cC03fwMKzBA25c5yrnMrrQ5QfKyY50) + [Lesson Template](https://docs.google.com/document/d/1Tr3wg6LeU_ZVkeKhyf4p1uPfpUgkgnuj4rUFJCfRrKM): check-in, warm-up game, teacher talk, independent + guided lab, My Favorite No, exit ticket, measurement.

### Instructor prep (before class)

- [ ] Confirm both students can open a terminal and have a **supported** Go (`go version` → **1.27.x or 1.26.x** as of Sep 2026).
- [ ] Open [pkg.go.dev](https://pkg.go.dev) and [makesite v1.2](https://github.com/Tech-at-DU/makesite#v12) in a browser tab.
- [ ] Create a throwaway demo folder ready for live typing (`/tmp/quotecheck`).
- [ ] DM list ready for Telephone cards (below).
- [ ] Favorite No paste + answer key ready (below).
- [ ] Optional: `go install golang.org/x/vuln/cmd/govulncheck@latest` once so the demo is fast.

## [**05m**] 🌤 Check-In / Entrance Ticket

**Solo, then paste in Zoom chat.**

1. How are you feeling today (one word)?
2. On modules / dependencies: Behind, on track, or ahead?
3. What is your plan for today’s lab (one sentence)?

**Instructor:** Say both names when reflecting pacing (“Alex is on track for SSG modules; Jordan is catching up from v1.1”). Growth-mindset language. If someone is “behind,” name one concrete first lab step they will hit first (`go mod init`).

## [**05m**] 🏆 Objectives

**Learning outcomes** — by the end of this session, students will be able to:

1. Initialize a Go module with `go mod init`.
2. Add a third-party dependency with `go get`, then keep `go.mod` / `go.sum` honest with `go mod tidy`.
3. Prefer the standard library when it already solves the problem; when taking a dependency, evaluate it on [pkg.go.dev](https://pkg.go.dev) (import path including `/v2+`, version, license, docs, vuln signals).
4. Explain why both `go.mod` and `go.sum` belong in git.

**Pre-measurement (chat, 1–5):** “I can add a Go module dependency and commit reproducible module files.”

**Measurement (end of class):** Exit ticket + pasted `module` line + evidence of a non-empty `go.sum` after lab.

**Why this matters:** Your SSG stops being “works on my laptop” theater the moment a classmate clones it. Modules are how Go makes that boring — on purpose.

## [**15m**] 🎮 Warm-Up: Dependency Telephone

**Game** (relationship-building + concept hook). Both students stay in the main Zoom room. No breakouts.

### Secret cards (DM privately — one per round)

**Card A:** `go mod download` does **not** add a new `require` to `go.mod`. It only fills the module cache for modules you already require.

**Card B:** You must commit **`go.sum`**, not only `go.mod`. `go.mod` records which versions; `go.sum` records checksums so Go can prove the downloaded bits match.

**Card C:** Prefer the import path from **pkg.go.dev** (Copy path). A guessed GitHub URL is a common 404 / wrong-module trap.

**Card D:** `go mod tidy` adds missing requires, drops unused ones, and refreshes `go.sum` so files match your imports.

### Timing script

| Minute | Move |
| --- | --- |
| 0:00–0:01 | Explain rules. No jargon dumps — write **steps** only. |
| 0:01–0:07 | Round 1: Student A gets Card A or B. Writes steps in chat. Student B reconstructs the rule. Reveal. |
| 0:07–0:13 | Round 2: Swap. New card. Reveal. |
| 0:13–0:15 | Debrief in chat (one line each). |

### Rules (play)

1. **Student A** has ~60 seconds to write **only requirements / steps** that would let someone else rediscover the truth — into Zoom chat.
2. **Student B** cannot see the secret. From A’s steps alone, B writes what they think the rule is.
3. Reveal the secret. Compare.
4. Swap roles.

### Debrief prompts (structured discussion)

Each student answers in chat (one line each):

- What got lost in translation?
- Where would that loss show up in a real repo (classmate cannot build, wrong import path, mystery versions)?

**Instructor punchline:** Modules exist so we stop playing telephone with versions.

## [**45m**] 💬 TT: Integrating Third-Party Libraries in Go

Teacher talk + live demos. Students follow along quietly; unmute only if stuck.

### Minute guide (keep the clock honest)

| TT minutes | Beat |
| --- | --- |
| 0–8 | Problem + packages vs modules |
| 8–18 | Live demo: `go mod init` + `go get` + `go run` |
| 18–25 | Read `go.mod` / `go.sum` + `go list -m all` |
| 25–32 | Toolchain / `GOTOOLCHAIN` (Sep 2026) |
| 32–38 | Evaluate on pkg.go.dev |
| 38–45 | `tidy` / `verify` / govulncheck flash + failure modes + when *not* to optimize |

Lightning only (do **not** derail): `go.work` is local multi-module glue — usually **do not commit**. `retract` / `exclude` exist; prefer upgrading past a bad version.

### The problem modules solve (0–8)

You ship a tiny CLI. It imports someone else’s markdown parser. On your machine: works. On a classmate’s machine: different version, different bugs, different build.

Modules pin **what** you depend on and **which version**, so builds stop being a vibes-based scavenger hunt.

### Packages vs modules

| Term | Meaning | Example |
| --- | --- | --- |
| Package | Folder of Go files with one `package` name | `encoding/json` |
| Module | Versioned tree of packages with a `go.mod` | `github.com/you/makesite` |

**Analogy:** A package is a chapter. A module is the book with an ISBN. GitHub is the bookstore.

**How Go picks versions:** Minimal Version Selection (MVS) — for each module path, take the highest version anyone required. It does **not** mean “always grab latest from the internet.”

**Chat checkpoint (30s):** Type `package` or `module` — which one has versions pinned in a `go.mod`?

### Live demo 1 — birth of a module (8–12)

```bash
mkdir -p /tmp/quotecheck && cd /tmp/quotecheck
go mod init example.com/quotecheck
cat go.mod
```

Point at:

- `module` — the **name** / import path prefix
- `go` — **minimum** language / toolchain line

The GitHub repo does **not** need to exist yet. The path is a name Go uses for imports.

### Live demo 2 — take a dependency the modern way (12–18)

Write `main.go`:

```golang
package main

import (
    "fmt"

    "rsc.io/quote"
)

func main() {
    fmt.Println(quote.Hello())
}
```

Then:

```bash
go get rsc.io/quote@latest
go run .
cat go.mod
head -n 20 go.sum
```

Narrate the terminal: finding → downloading → extracting. Show the new `require` line.

**Say this out loud:** `go mod download` is **not** how you add a library. It fills the cache for modules you already require. Even `go mod download path@version` only fills the module cache — it still does **not** add a `require` the way `go get` (or import + `tidy`) does. To add something: import it and build, or `go get path@version`, then `go mod tidy`.

**Also say:** `go install pkg@version` installs a **program** for you. It is not the same job as adding a library to this module.

### Read `go.mod` and `go.sum` (18–25)

- `go.mod` — declared module path, minimum `go` (and optional `toolchain`), plus `require`s (and rare `replace` / `exclude` / `retract`).
- `go.sum` — checksums so Go can prove the downloaded bits match. Same bits next time.

Commit **both**. Deleting `go.sum` does **not** “clean” the repo — it just recreates classmate-build friction and integrity / reproducibility pain.

```bash
go list -m all
go mod graph | head
```

Direct vs transitive deps — the graph is bigger than one import. Point at anything marked `// indirect` if it appears after upgrades.

### Toolchain reality check (25–32)

As of **Sep 2026**, students should be on a **supported** Go (**1.27** or **1.26**). Point releases matter for security fixes.

In `go.mod`:

- `go 1.XX.Y` = **minimum** required
- `toolchain go1.XX.Y` (Go 1.21+) = **suggested** toolchain when this module is main
- Default `GOTOOLCHAIN=auto` → `go` may download a toolchain via `GOPROXY` and verify via `GOSUMDB`

```bash
go version
go env GOPROXY GOSUMDB GOTOOLCHAIN
grep -E '^(go|toolchain) ' go.mod || true
```

**Classroom safety:** do **not** “fix” private-module pain with `GOSUMDB=off`. Use `GOPRIVATE` / `GONOSUMDB` patterns instead.

### Live demo 3 — evaluate on pkg.go.dev (32–38)

Open [pkg.go.dev](https://pkg.go.dev). Search a real need (markdown, CLI flags, HTTP router — pick one and stick with it).

**First ask:** Does the stdlib already solve this? (`encoding/json`, `flag`, `net/http`, `html/template`, `path/filepath`, `os` …)

On the module page, click through out loud:

1. **Import path** (Copy path) — this is what goes in `import`; check for a major-version suffix (`/v2`, `/v3`, …) and use that path if pkgsite shows it
2. **Versions** — prefer tagged semver over random pseudo-versions for shared pins
3. **License** — can you use it in class / your project?
4. **Docs / examples** — can you call it without guessing? Skim one concrete type or function you’d actually call
5. **Vuln signals** — pkg.go.dev surfaces known issues; `govulncheck` asks if they reach *your* code

**Do not** send students to dead indexes like GoCenter / `search.gocenter.io`.

### Tidy, verify, security flash (38–45)

```bash
go mod tidy
go mod verify
```

After you add or remove imports, `tidy` makes `go.mod` / `go.sum` match the code. Run it before you commit. Expect `-mod=readonly` failures if you hand-edit `go.mod` then `go build` without tidy — CI often runs that way on purpose.

Optional: `go mod vendor` only if you need a checked-in `vendor/` tree. Default SSG path: commit `go.mod` + `go.sum`, skip `vendor/` unless the checklist says otherwise.

```bash
go install golang.org/x/vuln/cmd/govulncheck@latest
govulncheck ./...
```

Low-noise: reachable vulns via call stacks. Point at the [govulncheck tutorial](https://go.dev/doc/tutorial/govulncheck). Do not turn this into a 20m security course — show the command, read one finding style if any, move on.

### Failure modes worth a grin

| What you see | What you probably did | Fix |
| --- | --- | --- |
| Classmate cannot build | Forgot to commit `go.sum` / `go.mod` | Commit both |
| `go mod download` “did nothing” to `go.mod` | Wrong add-dep command | `go get` or import + build, then `tidy` |
| Import path 404 energy | Guessed a GitHub URL | Copy path from pkg.go.dev |
| Proxy / download weirdness | Odd `GOPROXY` | Check `go env GOPROXY` (default `https://proxy.golang.org,direct`) |
| Toolchain download fails oddly | `GOSUMDB=off` / offline policy | Fix sumdb / proxy settings; don’t disable verification casually |
| `go: updates to go.mod needed; to update it: …` / `-mod=readonly` | Hand-edited `go.mod` or new import without tidy | `go mod tidy`, then rebuild; don’t fight readonly in CI |
| Classmate gets weird local paths / missing modules | Committed `replace => ../local` or a `go.work` | Drop homework `replace` / don’t commit `go.work` unless the checklist says so |

### On the job (quick)

- Treat `go.mod` as source of truth for **versions**; `go.sum` as the **checksum ledger**. Never hand-edit sum; never “clean” by deleting it.
- Pin tagged semver for shared work; `@latest` / pseudo-versions are demo glue.
- Private modules: `GOPRIVATE` / `GONOSUMDB`, never `GOSUMDB=off`.
- Upgrade intentionally (`go get path@vX.Y.Z` or `go list -m -u all`) — not blind `go get -u ./...` on deadline day.

### When *not* to optimize (~90s)

Modules make deps boring on purpose. Do **not** “optimize” by:

- Vendoring early for the SSG
- Adding a third-party lib for a one-liner the stdlib already does
- Chasing “latest everything” before green builds
- Hand-crafting the module graph (`exclude` / `retract` / deep `replace`) as a student fix
- Disabling verification (`GOSUMDB=off`) to make errors go away
- Micro-managing transitive pins unless you own the breakage
- Optimizing for monorepo `go.work` workflows on Day 5

**Close the TT:** Modules make third-party code boring — in a good way. Next: catch a bad module file, then put modules on your SSG.

## [**10m**] 🌴 BREAK

## [**10m**] 🔍 Activity: My Favorite No (Broken `go.mod`)

**My Favorite No** — whole-class analysis of one anonymous wrong answer. No pile-on critique.

### Paste this artifact (Zoom chat or shared doc)

```text
# anonymous "favorite no" — do not roast the author

module makesite

go 1.22

require rsc.io/quote v1.5.2

# notes from the student:
# 1) deleted go.sum on purpose "to clean up the repo"
# 2) ran: go mod download github.com/some/markdown@latest
# 3) confused why go.mod did not gain a markdown require
# 4) classmate cloned the repo and hit integrity / reproducibility pain
#    (missing sum → "updates needed" friction; not mysterious version
#     drift from deleting go.sum alone — versions come from go.mod + MVS)
```

### Solo (4m)

In notes, list:

1. What did they do **correctly**?
2. What is the **mistake** (there may be more than one)?
3. What **exact commands / files** fix it?

### Structured discussion (6m, 2 students)

- Student A: what’s correct (60–90s)
- Student B: mistakes + fix (60–90s)
- Instructor: tie back to Telephone + TT (2–3m)

### Instructor answer key (do not paste whole)

**Correct-ish:** They created a module (`module` + `go` lines) and have a real `require`.

**Mistakes:**

1. Deleting `go.sum` destroys integrity / reproducibility — missing sum → friction / “updates needed,” not mysterious version drift by itself. **`go.mod` requirements + MVS select versions; `go.sum` authenticates downloaded bits.**
2. `go mod download …` is the wrong add-dep tool — won’t invent a require the way students expect.
3. Weak module path: `module makesite` (no domain/repo path) is a soft mistake for a GitHub SSG — prefer something like `github.com/YOU/makesite`.
4. `go 1.22` is far below the supported classroom toolchain (not illegal with `GOTOOLCHAIN=auto`, but a teachable “minimum language line” cue — aim at currently supported majors).

**Fix path:** restore/regenerate `go.sum` with `go mod tidy` (and/or re-`go get` the intended modules), commit `go.mod` **and** `go.sum`, use `go get path@version` (or import + build) to add libs, copy import paths from pkg.go.dev, prefer a domain-style module path and a supported `go` line.

## [**45m**] 🧪 Lab Time: Solo Zoom Work

**Independent work** first; **guided** check-ins at the end. Stay in the main Zoom room. Mute is fine. Chat if stuck. Optional 2-person “ask classmate once in chat before the instructor.”

### Success criteria (everyone)

Finish the first item on the [makesite v1.2 checklist](https://github.com/Tech-at-DU/makesite#v12): add Go Modules support to your SSG.

You are done with core when:

1. `go.mod` exists at the SSG project root with a sensible `module` line
2. `go.sum` exists and is non-empty after a real build
3. `go build` / `go run .` works
4. Both files are committed
5. You pasted your `module` line into Zoom chat

### Suggested sequence

1. `cd` to your SSG project root (where `main.go` lives).
2. `go mod init github.com/YOUR_USERNAME/YOUR_SSG_REPO`
3. `go build` or `go run .`
4. `go mod tidy`
5. `git add go.mod go.sum && git commit -m "chore: add go module files"`
6. Paste the `module` line into Zoom chat

### Common blockers

| Blocker | Try this |
| --- | --- |
| `go mod init` complains a module already exists | Open the existing `go.mod`; don’t fight it — tidy and commit |
| Build fails on missing packages | Fix imports first; then `go mod tidy` |
| Empty or missing `go.sum` | Run `go mod tidy` after a successful build path |
| Wrong module path typo | Rename carefully; import paths must match the module path prefix |
| `-mod=readonly` / “updates to go.mod needed” | Run `go mod tidy` locally; don’t hand-edit sum or fight CI readonly |
| Import/`go get` path 404 for a known `/v2` module | Copy the full path from pkg.go.dev, including `/v2` (or `/v3`…) |

### Stretch (if you finish early)

1. **Evaluate a module (notes checklist):** path (incl. `/v2+` if shown), version you’d pin, license, one risk (maintenance / API churn / vuln note). On pkgsite, write down **one concrete type or function signature** you would call.
2. Optional: `go get` it only if it clearly helps the SSG — then `go mod tidy` and rebuild.
3. Optional: `govulncheck ./...` and note clean vs finding.

### Guided check-ins (last ~10m)

- Student A screen share: `go.mod`, `go.sum`, successful build
- Student B screen share: same
- Remaining minutes: keep coding

## [**05m**] 🌃 Exit Ticket

**Solo in Zoom chat / quick form:**

1. Did you accomplish what you set out to do today?
2. Paste your `module` line (or `blocked: …` + one blocker).
3. In one line: what does `go.mod` decide vs what does `go.sum` prove?
4. Who helped you today / who did you help? (classmate or instructor)

### After class

- Finish any unfinished v1.2 modules checklist item before next class.
- Skim [Managing dependencies](https://go.dev/doc/modules/managing-dependencies) if `go get` vs `tidy` still feels fuzzy.

## 📚 Resources & Credits

- [**How to Write a Self-Paced Lesson**](https://docs.google.com/document/d/16O2xGeNSayfQ1cC03fwMKzBA25c5yrnMrrQ5QfKyY50) — check-in, warm-up menu, exit ticket patterns.
- [**Lesson Template**](https://docs.google.com/document/d/1Tr3wg6LeU_ZVkeKhyf4p1uPfpUgkgnuj4rUFJCfRrKM) — LO / pre-measurement / independent + guided work / measurement.
- [**Managing dependencies**](https://go.dev/doc/modules/managing-dependencies) — official `go get` / `tidy` / proxy workflow.
- [**Using Go Modules**](https://go.dev/blog/using-go-modules) — init / get / tidy walkthrough.
- [**Go toolchains**](https://go.dev/doc/toolchain) — `go` vs `toolchain` / `GOTOOLCHAIN`.
- [**pkg.go.dev**](https://pkg.go.dev) — search, docs, versions, vuln signals.
- [**govulncheck tutorial**](https://go.dev/doc/tutorial/govulncheck) — reachable vulns.
- [**Modules reference — Minimal version selection (MVS)**](https://go.dev/ref/mod#minimal-version-selection) — optional depth on how Go picks versions.
- [**Modules reference — Major version suffixes (`/v2+`)**](https://go.dev/ref/mod#major-version-suffixes) — optional depth on semantic import versioning.
- [**makesite v1.2**](https://github.com/Tech-at-DU/makesite#v12) — lab checklist.
