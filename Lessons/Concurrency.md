<!-- Run as a slideshow: reveal-md Lessons/Concurrency.md -w -->
# Concurrency — Goroutines & Practical Applications

⭐️ **GOAL:** Leave able to explain how Go runs concurrent work with goroutines (and how that differs from parallelism), sketch a simple channel handoff, and ship a Slack bot that uses goroutines for live event handling.

<!-- omit in toc -->
## ⏱ Agenda

- [[**5m**] Attendance &amp; Announcements](#5m-attendance--announcements)
- [[**15m**] ☀️ Warm Up](#15m-️-warm-up)
- [[**30m**] 📚 TT: Goroutines Through Illustrations](#30m--tt-goroutines-through-illustrations)
- [[**10m**] 🌴 Break](#10m--break)
- [[**20m**] 💻 Activity 1: Slack App and goslackit Setup](#20m--activity-1-slack-app-and-goslackit-setup)
- [[**35m**] 💻 Activity 2: Goroutine Slackbot Challenges](#35m--activity-2-goroutine-slackbot-challenges)
- [[**5m**] Wrap Up](#5m-wrap-up)

<!-- > -->

<!-- omit in toc -->
## 🏆 Objectives

*By the end of this session, you'll be able to&hellip;*

1. **Define** concurrency in Go: lightweight goroutines scheduled by the runtime, not “one OS thread per task.”
2. **Contrast** concurrency (structure for dealing with lots of things at once) vs parallelism (doing lots of things at the *same* time).
3. **Sketch** a channel send/receive handoff and name one blocking pitfall (send with no receiver, or vice versa).
4. **Ship** a running Slack bot from [goslackit](https://github.com/droxey/goslackit) with the starter TODOs completed and tested live in Slack.

<!-- > -->

## [**5m**] Attendance &amp; Announcements

Today is **goroutines in practice**. TT walks [Learning Go's Concurrency Through Illustrations](../Resources/GoConcurrencyVisualized.md). Lab is a real Slack bot: configure the app, clone the starter, knock out the `TODO` challenges while the bot listens on a channel.

Have a terminal with a supported Go ready (`go version`). Prefer **one machine** for the bot token + `go run` path today so OAuth and env stay in one place.

<!-- > -->

## [**15m**] ☀️ Warm Up

Before we wire Slack, pick the **commands** your bot should own. Concurrent services earn their keep when work arrives as a stream of events — messages, reactions, slash commands — and you handle them without blocking the whole process on one slow path.

**Warm-up prompt (think → chat → share):**

1. Name **one or two bot commands** you’d actually use on a team (status check, random picker, deploy ping, standup reminder, …).
2. In one sentence: if the bot waits synchronously on a slow API call *inside* the message handler, what happens to the next Slack event?

Whiteboard three buckets if chat stays quiet:

| Concurrent-friendly | Blocks the handler | Stretch later |
| --- | --- | --- |
| Reply to a channel message | Sync HTTP with no timeout | Slash commands + modals |
| Fan out “react + log + reply” | Busy-loop while waiting | Multi-workspace install |
| Listen on a channel, exit clean | Ignoring disconnects | Heroku worker process |

> **📈 PROTIP:** Write the command verbs first. The goroutine wiring is easier when you already know what “done” looks like in Slack.

<!-- > -->

## [**30m**] 📚 TT: Goroutines Through Illustrations

**Next action:** Walk the illustrated mental model → goroutines → channels → one blocking story → lab brief.  
**Done when:** You can say concurrency ≠ parallelism, sketch `go f()` plus a channel handoff, and name why a send with no receiver stalls.

Present and discuss: **[Learning Go's Concurrency Through Illustrations](../Resources/GoConcurrencyVisualized.md)**  
Source article: [Trevor Blanarik on Medium](https://medium.com/@trevor4e/learning-gos-concurrency-through-illustrations-8c4aff603b3)

### 1. Single-Threaded vs Concurrent Work (~6m)

The illustrations start with a mine: find ore → mine → smelt, one worker doing each step in order. That is fine until work piles up or waits on I/O.

Say:

> “Concurrency is how you **structure** a program so it can make progress on many tasks. Parallelism is when those tasks actually run at the same time on multiple cores. Go gives you cheap goroutines so the structure is easy — the runtime decides how to map them onto threads.”

> **💬 ASK AUDIENCE:** A bot handles Slack events and also polls a slow third-party API. Is “run both in one process” automatically parallelism?

<details>
<summary>Answer</summary>

**Not automatically.** You may be concurrent (two goroutines making progress across waits) on one core. Parallelism needs multiple cores actually executing at once. Rob Pike’s line still holds: concurrency is about structure.

</details>

### 2. Goroutines — `go` Starts Work (~8m)

- `go f()` starts `f` on a new goroutine and returns immediately.
- Goroutines are cheap relative to OS threads; create many, don’t treat them like processes.
- The `main` goroutine exiting ends the program — background work can vanish if nothing keeps `main` alive.

```go
package main

import (
  "fmt"
  "time"
)

func beep(id int) {
  fmt.Println("beep", id)
}

func main() {
  go beep(1)
  go beep(2)
  time.Sleep(50 * time.Millisecond) // demo only — real code uses sync / channels
  fmt.Println("main done")
}
```

> **‼️ BE AWARE:** `time.Sleep` in demos hides a real design choice. Production code coordinates with channels, `WaitGroup`, or context cancel — not sleep.

> **💬 ASK QUESTION:** You `go` a handler that sends a Slack reply, then `main` returns immediately. What happens to that reply?

<details>
<summary>Answer</summary>

**It may never send.** When `main` returns, the process exits. Keep the process alive (HTTP server, Slack RTM/Socket Mode loop, worker `for { select … }`, etc.).

</details>

### 3. Channels — Hand Off Values (~8m)

Channels move values between goroutines. Unbuffered send waits for a receive (and vice versa). Buffered channels add capacity before blocking.

```go
func main() {
  ch := make(chan string)

  go func() {
    ch <- "event:hello" // blocks until someone receives
  }()

  msg := <-ch
  fmt.Println(msg)
}
```

Illustrations to call out on screen (from the resource):

- Blocking on send / receive
- Unbuffered vs buffered
- `range` over a channel until close
- Non-blocking `select` with `default` (name it; don’t over-teach today)

> **💬 QUICK CHECK:** Unbuffered `ch <- x` with nobody receiving. What does the sender do?

<details>
<summary>Answer</summary>

**Blocks** until a receiver is ready (or the program deadlocks if nobody will ever receive).

</details>

### 4. Why a Slack Bot Needs This (~5m)

Slack delivers a stream of events. A solid pattern:

1. One goroutine (or library loop) **reads** events.
2. Handlers **react** without freezing the whole listener on every slow call.
3. Shared state (if any) is protected — channels or sync — not “hope.”

Today’s starter ([goslackit](https://github.com/droxey/goslackit)) already leans on goroutines. Your job: configure the Slack app, paste the token, complete the `TODO` challenges, and prove it live in channel.

> **📈 TIP:** Keep [Writing Slackbots with Goroutines](https://x-team.com/blog/writing-slackbots-with-goroutines/) open while you work the TODOs — it matches this lab’s story.

### 5. Lab Brief (~3m)

Activity 1 = Slack app + clone + `.env` + `go run`. Activity 2 = hunt `TODO`, finish challenges, test in Slack. Heroku deploy is **optional** after class / finished early — not the session gate.

<!-- > -->

## [**10m**] 🌴 Break

Stand up. Leave a browser tab on [Create a Slack App](https://api.slack.com/apps?new_app=1) and the [Bot Users](https://api.slack.com/bot-users) docs.

<!-- > -->

## [**20m**] 💻 Activity 1: Slack App and goslackit Setup

> **✅ DONE WHEN:** A Slack app exists with bot scopes saved, the bot is installed to the workspace, [goslackit](https://github.com/droxey/goslackit) is forked and cloned, `.env` has `BOT_OAUTH_ACCESS_TOKEN=…`, and `go run main.go` starts without an immediate crash.

### Create a Slack App

1. Keep the **[Slack Bot User](https://api.slack.com/bot-users)** docs open while you configure.
2. Open **[Create a Slack App](https://api.slack.com/apps?new_app=1)**.
3. Enter a **name that fits the problem you’re solving**.
4. Select **your team’s Slack workspace** from the Workspace drop-down.
5. Click **Create App**.
6. On the sidebar, under Features, open **Bot Users** (or the current **App Home** / bot section — Slack’s UI moves; follow the Bot User docs if labels differ).
7. Give your bot a **display name** and a **default username**.
8. Click **Save Changes**.
9. Open **OAuth & Permissions**. Under **Scopes**, add at least: `channels:history`, `channels:read`, and `channels:write` (adjust if the starter README lists newer bot scopes). Save.
10. Click **Install App to Workspace**. Grant the scopes. Prefer installing so the bot can work in the shared practice channel (historically `#golang-slackbots` — use whatever channel your cohort agreed on). Match your OAuth consent dialog to the team’s expected scopes:

<p align="center">
  <img src="img/oauth-enable.png" height="350">
</p>

### Setup Project

1. **Fork** the [goslackit starter](https://github.com/droxey/goslackit).
2. **Clone your fork** and `cd` into the repo. Prefer **one computer** for token + run today.
3. Create `.env` from the sample: `cp .env.sample .env`.
4. **Paste** the Bot User OAuth token into `.env` after `BOT_OAUTH_ACCESS_TOKEN=`.
5. Run `go run main.go` to start the bot process. If it fails immediately, paste the error in chat and pair with a teammate.

> **‼️ BE AWARE:** Never commit `.env` or paste tokens into the lesson chat history. Rotate the token if it leaks.
> **FINISHED EARLY?** Brainstorm the exact Slack message you’ll send to prove each `TODO` before Activity 2 starts.

<!-- > -->

## [**35m**] 💻 Activity 2: Goroutine Slackbot Challenges

> **✅ DONE WHEN:** You found all three `TODO` challenges in the starter, completed them with a teammate as needed, `go run main.go` stays up, and you manually verified bot behavior in Slack for each challenge.

If you get stuck, use: **[Writing Slackbots with Goroutines](https://x-team.com/blog/writing-slackbots-with-goroutines/)**.

1. Search the project for `TODO` — there are **three** challenges in the comments.
2. Complete each one. Watch terminal output from `go run main.go` as you go.
3. Manually **test the bot in Slack** after each challenge — don’t wait until the end.
4. Stretch challenge in the repo is optional if you finish early.

> **📈 PROTIP:** Prove one `TODO` in Slack before starting the next. Concurrent bugs compound when you batch unverified changes.
> **FINISHED EARLY?** Jump to **After Class: Heroku Deployment** below, or help a teammate get `go run` green.

### After Class / Finished Early — Heroku Deployment (Optional)

Deploying the bot means it keeps running when your laptop sleeps.

**Sample script** (replace placeholders):

```bash
git clone git@github.com:USERNAME/goslackit.git PROJECT_NAME
cd PROJECT_NAME
heroku create PROJECT_NAME
heroku config:set BOT_OAUTH_ACCESS_TOKEN=YOUR_BOT_TOKEN
git push heroku master
heroku ps:scale worker=1
```

> **‼️ BE AWARE:** Heroku dyno types and `master` vs `main` default branch change over time — follow current Heroku + repo defaults if a command fails.

<!-- > -->

## [**5m**] Wrap Up

Say out loud:

1. **Concurrency** structures many tasks; **parallelism** runs them at the same time.
2. **`go f()`** starts work; **`main` exiting** can kill it.
3. **Channels** hand off values; unbuffered send/receive **rendezvous**.
4. Lab path: Slack app → goslackit → `TODO`s → live channel proof → optional Heroku.

Optional notes card:

```text
Shipped today:
Stuck on:
Next session first 15m:
```

<!-- > -->

## Additional Resources

1. **[Learning Go's Concurrency Through Illustrations](../Resources/GoConcurrencyVisualized.md)** — in-repo illustrated walkthrough (TT source).
2. **[Trevor Blanarik — Learning Go’s Concurrency Through Illustrations](https://medium.com/@trevor4e/learning-gos-concurrency-through-illustrations-8c4aff603b3)** — original article.
3. **[Rob Pike — Concurrency is not Parallelism (2012 slides)](https://talks.golang.org/2012/waza.slide)** — classic distinction.
4. **[Golang Bootcamp — Concurrency](http://www.golangbootcamp.com/book/concurrency)** — chapter with runnable samples.
5. **[Learn Go with Tests — Concurrency](https://github.com/quii/learn-go-with-tests/blob/master/concurrency.md)** — TDD-style practice.
6. **[A Complete Journey with Goroutines](https://medium.com/@riteeksrivastava/a-complete-journey-with-goroutines-8472630c7f5c)** — deeper goroutine tour.
7. **[Concurrency in Go: Visualization with Example](https://medium.com/@dmrajkarthick.2012/concurrency-in-go-visualization-with-example-8deaf2cf3ee6)** — visual goroutine walkthrough.
8. **[Writing Slackbots with Goroutines](https://x-team.com/blog/writing-slackbots-with-goroutines/)** — lab companion.
9. **[Slack API — Bot Users](https://api.slack.com/bot-users)** — bot configuration reference.
10. **[goslackit starter](https://github.com/droxey/goslackit)** — fork target for Activities 1–2.

## For Curriculum Authors

<details>
<summary>For Curriculum Authors</summary>

### In Class

| | |
| --- | --- |
| **Next action** | Open this file → Agenda → Attendance → Warm Up command brainstorm → TT illustrations → Break → Activity 1 Slack app → Activity 2 TODOs. |
| **Done when** | At least one bot in the room replies in Slack from a completed TODO, or a clear stuck-point with a pasted error and next fix. |

- Warm-up is think → chat → share; no breakouts required.
- TT should **screenshare** `Resources/GoConcurrencyVisualized.md` (or the Medium original). Stay on illustrations; do not turn TT into a full channels deep-dive.
- Behind at ~0:40? Cut non-blocking `select` mention; protect Activity 1 token setup.
- Activity 2 gate is three TODOs + live Slack proof — Heroku is optional after class / finished early only.
- Confirm the practice channel name day-of; source historically used `#golang-slackbots`.

### Facilitator Notes

- Prefer speakable TT. Pike’s concurrency ≠ parallelism line once is enough.
- Fix the historical double-paren Create-a-Slack-App link (already corrected in this draft).
- Slack’s UI labels drift (Bot Users vs App Home). Point at official Bot User docs when the sidebar differs.
- Never collect roster names or tokens in the plan or shared notes.

### Expert Follow-Ups

- Optional after-session: `select`, context cancel, Socket Mode vs older RTM, `sync.WaitGroup`.
- Re-check Slack OAuth scope names day-of against the goslackit README before projecting Activity 1.

</details>
