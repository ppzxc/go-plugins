# go-coder Skill Rewrite Design

**Date:** 2026-03-24
**Status:** Approved

## Context

The current `go-coder` skill (349 lines, 11 sections) was assembled from 3 books and organized by topic. This rewrite replaces it entirely with a practical scenario-based structure grounded in 5 canonical Go books. The goal is a skill that activates usefully during real code writing, review, and refactoring — not just as a reference index.

**Source books:**
- `[GoBook]` — *The Go Programming Language*, Donovan & Kernighan
- `[LearningGo]` — *Learning Go* 2nd Ed., Jon Bodner
- `[ConcurrencyInGo]` — *Concurrency in Go*, Katherine Cox-Buday
- `[Mistakes]` — *100 Go Mistakes and How to Avoid Them*, Teiva Harsanyi
- `[CloudNative]` — *Cloud Native Go*, Matthew Titmus

## Approach

**B: Practical scenario-based structure.** Organized around the real-world Go development workflow: design types → handle errors → propagate context → write concurrent code → structure a service → test it → make it resilient → optimize it. Each pattern cites its source book and chapter/mistake number.

## Section Design

### Section 1: Types & Interfaces
**Sources:** `[GoBook]` Ch.6-7, `[LearningGo]` Ch.6-8, `[Mistakes]` #5-#8

- Accept interfaces, return structs — enables loose coupling without locking callers
- Keep interfaces small (1-3 methods) — `io.Reader`, `io.Writer` as models
- Implicit implementation — design interfaces at the consumer, not the producer
- Embedding for composition — prefer over inheritance patterns
- Generics — type constraints, when to use functions vs methods, avoid premature generalization

### Section 2: Error Handling
**Sources:** `[GoBook]` Ch.5, `[LearningGo]` Ch.9, `[Mistakes]` #48-#53

- `%w` preserves the chain for `errors.Is`/`errors.As`; `%v` intentionally breaks it
- Sentinel errors for identity checks; custom types when callers need structured data
- `errors.Is` for value comparison; `errors.As` for type extraction
- Goroutine error collection — use `errgroup` (see Section 4 for full comparison with WaitGroup)
- Panic only for unrecoverable programmer bugs (nil callback, invariant violation)
- `errors.Join` for combining multiple errors (Go 1.20+) `[Mistakes]` #50

### Section 3: Context
**Sources:** `[GoBook]` Ch.8, `[LearningGo]` Ch.12, `[Mistakes]` #60-#63

- Always first parameter, never stored in structs
- `WithTimeout`/`WithDeadline` for I/O and external calls; `WithCancel` for lifecycle control
- Context values: use unexported key types to avoid collisions; only for request-scoped data
- Cancellation propagates automatically — goroutines must check `ctx.Done()`
- Pair `WithCancel` with `defer cancel()` immediately

### Section 4: Concurrency
**Sources:** `[GoBook]` Ch.8-9, `[ConcurrencyInGo]` Ch.2-5, `[Mistakes]` #57-#73

- Goroutine lifecycle: never start a goroutine without knowing when it exits
- Pipeline pattern: chain stages with channels, use `done` channel for cancellation
- Fan-out / fan-in: distribute work, merge results
- `errgroup` for parallel tasks that can fail; `sync.WaitGroup` for fire-and-forget (primary coverage of errgroup — Section 2 cross-references here)
- Data race detection: always run `go test -race`; races are undefined behavior `[Mistakes]` #58
- Mutex for protecting shared state; channel for transferring ownership
- Goroutine leak patterns: blocked send/receive, missing cancellation check
- `sync.Once` for one-time initialization; `sync.Pool` for reusable allocations

### Section 5: Service Design
**Sources:** `[CloudNative]` Ch.2-5, `[LearningGo]` Ch.14, `[GoBook]` Ch.10

- `cmd/` for binaries, `internal/` for private packages, `pkg/` for exported libraries
- Package names: lowercase, singular, no stutter (`user.User` → `user.Record`)
- HTTP handlers: accept dependencies via closure or receiver, not global state
- Middleware chaining: `func(http.Handler) http.Handler` signature
- Functional Options for optional config; Config Struct when >3 required fields
- Constructor naming: `New` for the primary type, `NewXxx` for alternatives

### Section 6: Testing
**Sources:** `[LearningGo]` Ch.13, `[Mistakes]` #82-#84

- Table-driven with `t.Run` — each case is named and independently reportable
- `t.Helper()` in every helper function — failure points to the caller, not the helper
- `t.Cleanup()` over `defer` — runs even when test calls `t.FailNow()`
- `t.Parallel()` at the top of each case in table-driven tests — capture loop variable first
- Manual mock via interface — avoid mock generators for simple cases
- Benchmarks: call `b.ResetTimer()` after setup; use `b.ReportAllocs()` for allocation tracking
- Include a Quick Decision Reference table at the end summarizing key choices (errgroup vs WaitGroup, mutex vs channel, sentinel vs custom error, functional options vs config struct)

### Section 7: Resilience
**Sources:** `[CloudNative]` Ch.5-8, `[ConcurrencyInGo]` Ch.5

- Retry with exponential backoff + jitter — prevents thundering herd on recovery
- Circuit breaker: Closed → Open → Half-Open state machine; fail fast when open
- Graceful shutdown: catch `SIGTERM`/`SIGINT`, stop accepting, drain in-flight, exit
- Rate limiting: token bucket with `time.Ticker` or `golang.org/x/time/rate`
- Health check endpoint: `/healthz` (liveness) and `/readyz` (readiness) separation

### Section 8: Performance & Pitfalls
**Sources:** `[Mistakes]` #20-#29, #39, #47, #95-#97, `[GoBook]` Ch.3

- Slice append: always reassign (`s = append(s, v)`); pre-allocate with known length
- Backing array aliasing: `s2 := s1[2:4]` shares memory — copy when independence needed
- nil vs empty slice: `var s []T` (nil) vs `s := []T{}` (non-nil, empty) — JSON differs
- Map: never write to nil map; pre-allocate with `make(map[K]V, n)`; iteration order is random
- `strings.Builder` for repeated concatenation — avoids O(n²) allocations
- `defer` in loops: defers stack until function returns, not until loop iteration ends
- Goroutine leak via slice: returning sub-slice of large array keeps entire backing array alive
- Escape analysis: use `go build -gcflags="-m"` to see heap escapes; minimize pointer returns

## File to Rewrite

`/home/ppzxc/projects/go-skills/plugins/go-coder/skills/go-coder/SKILL.md`

## Verification

1. Install plugin: `/plugin install go-coder`
2. Invoke: `/go-coder` — skill should load with new structure
3. Trigger on Go code write/review — frontmatter `description` should match activation context
4. Check all 8 sections render correctly in Claude Code
