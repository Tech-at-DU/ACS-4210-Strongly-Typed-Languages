# Day 5: Fast Functionality Via 3rd Party Libraries

| **Elapsed** | **Time** | **Activity** |
| ----------- | -------- | ------------------------- |
| 0:00 | 0:05 | Why / Objectives |
| 0:05 | 0:45 | Overview |
| 0:50 | 0:10 | BREAK |
| 1:00 | 0:25 | In Class Activity I |
| 1:25 | 0:25 | In Class Activity II |
| 1:50 | 0:05 | Wrap Up |

<details>
<summary><strong>Instructor prep</strong> (before class)</summary>

- [ ] Confirm students have a **supported** Go (`go version` → current supported majors; as of Sep 2026: **1.27.x / 1.26.x**).
- [ ] Tabs open: [pkg.go.dev](https://pkg.go.dev), [makesite v1.2](https://github.com/Tech-at-DU/makesite#v12).
- [ ] Throwaway demo folder ready (`/tmp/quotecheck`).
- [ ] Optional: `go install golang.org/x/vuln/cmd/govulncheck@latest` once for a fast flash.

</details>

## Why You Should Know This (2 min)

Your SSG only “works on your laptop” until a classmate clones it. **Modules** pin *what* you depend on and *which version*, so builds stop being a scavenger hunt.

## Learning Objectives (3 min)

1. Define `package` vs `module` and name what `go.mod` / `go.sum` each record.
2. Initialize a module with `go mod init` and add a dependency with `go get` + `go mod tidy`.
3. Evaluate a library on [pkg.go.dev](https://pkg.go.dev) before adopting it (path, version, docs, license).
4. Commit both `go.mod` and `go.sum` so a classmate’s build matches yours.

## Overview/TT (45 min)

### Package vs Module (5 min)

| Term | Meaning | Example |
| --- | --- | --- |
| Package | Folder of Go files with one `package` name | `encoding/json` |
| Module | Versioned unit of distribution (`go.mod` at the root) | `github.com/you/makesite` |

**Minimal Version Selection (MVS):** for each module path, Go takes the highest version *anyone required* — not “always grab latest from the internet.”

### Creating a Module (5 min)

```bash
go mod init github.com/GITHUB_USERNAME/GITHUB_REPO_NAME
```

The GitHub repo does not have to exist yet. **Discuss why.**

### Live Demo: `init` → `get` → `run` (15 min)

```bash
mkdir /tmp/quotecheck && cd /tmp/quotecheck
go mod init example.com/quotecheck
go get rsc.io/quote@v1.5.2
```

```go
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
go mod tidy
go run .
go list -m all
```

Open `go.mod` and `go.sum` together.

| File | Role |
| --- | --- |
| `go.mod` | Module path, `go` line, `require` (and rare `replace` / `exclude` / `retract` / `toolchain`) |
| `go.sum` | Checksums so Go can prove downloaded bits match |

**Commit both.** Deleting `go.sum` is not “cleaning” the repo.

### Adding Dependencies — command roles (8 min)

| Command | Use it for |
| --- | --- |
| `go get path@version` | Add/update a require for *this* module |
| `go mod tidy` | Make `go.mod` / `go.sum` match imports |
| `go mod download` | Fill the module *cache* only — not how you add a library to this module |
| `go install path@version` | Install a *tool* for you — not a dependency of this module |

Even `go mod download path@version` only fills cache; it still does not add a require the way `go get` (or import + tidy) does.

### Evaluating on pkg.go.dev (7 min)

1. Prefer the **stdlib** if it already solves the problem
2. Copy the import path (include `/v2+` when pkgsite shows it)
3. Check version, license, docs
4. Write down **one concrete type or function** you would call

### Failure modes + when *not* to optimize (5 min)

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `-mod=readonly` / “updates to go.mod needed” | Hand-edited `go.mod` or new import without tidy | `go mod tidy`, then rebuild |
| Import/`go get` 404 for a known `/v2` module | Missing major-version suffix | Copy full path from pkg.go.dev |
| Classmate can’t build | Missing `go.sum`, leftover `replace`, or uncommitted `go.mod` | Commit both files; drop local-only `replace` / `go.work` |

**Do not “optimize” by:** vendoring early, `GOSUMDB=off`, adding a lib for a one-liner the stdlib does, or blind `go get -u ./...` on deadline day.

## BREAK (10 min)

## In Class Activity I (25 min)

1. Open your **makesite** (SSG) repo.
2. If you do not have a module yet: `go mod init github.com/YOUR_USER/YOUR_REPO`.
3. Add **one** third-party dependency you actually need (or `rsc.io/quote` while exploring).
4. Run `go mod tidy`. Confirm `go.mod` and a non-empty `go.sum`.
5. Commit **both** files. Push.

## In Class Activity II (25 min)

1. On [pkg.go.dev](https://pkg.go.dev), evaluate **one** library relevant to your SSG.
2. Write in your README (or Slack):
   - import path (incl. `/v2+` if shown)
   - version you’d pin
   - license
   - one risk (API churn / maintenance / vuln)
   - one concrete type or function signature you’d call
3. Prefer stdlib if it already covers the need — say so explicitly.

### Stretch Challenges

1. What breaks for a classmate if you commit a `replace => ../local` or a `go.work` file by accident?
2. What does `-mod=readonly` mean in CI, and how do you fix “updates to go.mod needed”?
3. When would you *not* add a third-party dependency?

## Wrap Up (5 min)

Paste in Slack / Zoom:

1. Your `module` line from `go.mod`
2. One sentence: what does `go.mod` decide vs what does `go.sum` prove?

## Additional Resources

1. **[Using Go Modules](https://go.dev/blog/using-go-modules)** — canonical walkthrough.
2. **[Managing dependencies](https://go.dev/doc/modules/managing-dependencies)** — `get` / `tidy` / upgrade habits.
3. **[pkg.go.dev](https://pkg.go.dev)** — discover and evaluate modules.
4. **[Modules reference — MVS](https://go.dev/ref/mod#minimal-version-selection)** — optional depth.
5. **[Major version suffixes (`/v2+`)](https://go.dev/ref/mod#major-version-suffixes)** — optional depth.
6. **[makesite v1.2](https://github.com/Tech-at-DU/makesite#v12)** — SSG milestone this unlocks.
