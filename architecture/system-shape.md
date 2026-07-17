# System shape

The macro-shape invariants of the running system — the split between the deterministic code that coordinates and the intelligent work it drives, the seam every external system sits behind, and the store-schema rule that makes crash recovery correct.
These are the foundations the daemon loops (`bzh:steppable-loop` … in [./crash-correctness.md](./crash-correctness.md)) are built on.
Each rule follows the slot skeleton owned by `winter-canon:/rule-shape.md` (`canon:rule-shape`).

## Deterministic shell, intelligent core (`bzh:deterministic-shell`)

**Rule.** Coordination — the runner tick, the hub coordinator, the workflow transitions, the store reads and writes — is **deterministic** code with no model calls; the **intelligent** work (the harness doing a node-step) is confined to the leaf where a worker runs, behind the harness seam.
Orchestration decides *what* to run and *where*; it never itself reasons with an LLM.

**Why.** A deterministic shell is replayable, unit-testable without tokens, and crash-recoverable — the whole crash-correctness harness depends on the loop being a pure function of (store, clock, seams). Pushing model judgement into the coordinator would make the loop non-deterministic and untestable, and would spend tokens on control flow.

**Detect.** A model call, a prompt, or an LLM client inside a runner loop step, the hub coordinator, a transition, or a store method; orchestration logic that branches on freshly-generated model output rather than on a parsed verdict fact.

**Do.** The worker (harness seam) produces a verdict; the coordinator reads the parsed verdict fact from the store and picks the workflow edge deterministically.

**Don't.** The coordinator prompts a model to decide the next node — control flow is now non-deterministic and cannot be replayed under the crash sweep.

## Every external system behind a seam (`bzh:pluggable-seams`)

**Rule.** Every external system is reached only through an interface — a seam — with the reference stack as its first binding: the work source (at the hub), the workspace provider, the coding harness, delivery (the forge), and the human channel are all Protocols, and their concrete bindings (GitHub, winter, Claude Code) are swappable adapters selected by configuration.
A seam is the external-system application of dependency inversion (`bzh:dependency-inversion`).

**Why.** A seam is what lets tests bind the mock fleet in place of the real stack — the entire service and e2e strategy runs seams-mocked, spending no tokens and touching no network — and lets a runner swap winter worktrees for plain worktrees without touching the loop. Code that calls a vendor SDK directly cannot be tested without that vendor and cannot be re-bound.

**Detect.** A vendor SDK, the GitHub API, or a `claude`/harness binary invoked directly from a loop step, the domain, or a store — rather than through an injected seam Protocol; a test that cannot run without a real external system because no seam exists to bind a mock to.

**Do.** The runner depends on `IWorkspaceProvider`, `IHarness`, and the forge seam; production binds winter / Claude Code / GitHub, tests bind the `blizzard-mock` fleet.

**Don't.** A FILL step that shells out to `claude -p` directly — the loop can no longer be exercised against the mock harness and every service test now needs real tokens.

## Store facts, derive status (`bzh:facts-not-status`)

**Rule.** Both daemons' stores hold only durable **facts** — a thing that definitely happened at a definite time (a lease created, a heartbeat received, a transition recorded, a verdict parsed).
A chunk's **status** is always *derived* by query from those facts, never written as a column.

**Why.** Written status lies after a crash: a process that wrote `running` and then died reports `running` forever, while a status derived from "last heartbeat 20 minutes ago and the pid is dead" tells the truth however the process ended — this single rule is what makes crash recovery correct rather than aspirational, and is what the invariant checker (`bzh:invariant-checker`) asserts against.

**Detect.** A `status` / `state` column written by application code; a derived condition (running, waiting, stalled, done) persisted rather than computed from underlying fact rows at read time.

**Do.** Persist `Lease`, `Heartbeat`, `Transition`, `Verdict` rows; compute chunk status by querying them (last heartbeat age, pid liveness, latest transition).

**Don't.** Write a `chunk.status = "running"` column and update it as the chunk moves — the column outlives the truth the instant the process dies.

## See also

- [./crash-correctness.md](./crash-correctness.md) — the four daemon requirements built on `bzh:facts-not-status` and `bzh:deterministic-shell`.
- [../standards/persistence.md](../standards/persistence.md) — `bzh:sql-portable`, the portable-SQL rule the facts-only stores are held to.
