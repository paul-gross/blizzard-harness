# Execution — runners, tenure, and fencing

Who runs a chunk and how exactly-once holds: the hub/runner responsibility split, acquisition and routes, leases and epochs, and what failure does to each.
Definitions, with the enforceable invariant written in the slot skeleton owned by `winter-canon:/rule-shape.md` (`canon:rule-shape`).
Part of the [domain model](./index.md).

## The responsibility split

- **The hub** orchestrates the fleet's work: it owns chunks, graphs, artifacts, and the registry, and it grants work. It never holds code or transcripts (`bzh:never-code` in [artifacts.md](./artifacts.md)) and never reaches into a runner's machine.
- **A runner** executes work on its own machine, bound to one prepared workspace: it claims chunks, acquires environments, drives workers through node-steps, and reports the facts. All contact is runner-initiated — the hub pushes nothing into a dev box.
- **Operator controls are declarative state, not commands.** Pausing a runner appends a fact the runner reads and adheres to on its own contact — new leases stop, in-flight chunks run to completion. There is no directive queue.

A runner's registry entry derives everything observable: liveness derives from its most recent contact, and paused-ness from the newest pause/resume fact — never stored flags (`bzh:facts-not-status` in [../architecture/system-shape.md](../architecture/system-shape.md)).

## Acquisition and the route

**Acquisition** is the hub granting a ready chunk to exactly one runner — the one point of genuine cross-runner contention, and where fleet exactly-once is upheld.
The claim is **claim-by-route**: the runner peeks the hub-ordered queue, acquires the environments, and posts the complete route; the hub accepts exactly one claim per chunk.

The **route** is the locator fact that makes every held chunk findable: chunk → runner → workspace → environment.
The environment identifier is opaque to the hub — it knows *which* environment, never what an environment is.

- **Runner stickiness.** Consecutive node-steps of a chunk run on the holding runner — no re-queue between nodes.
- **A route ends** with the chunk's terminal outcome, or by **detach**: a superadmin's forcible release, after which the chunk re-derives ready and the old runner is fenced out by the next claim.

## Lease and epoch

A **lease** is one node-step attempt: one lease, one agent session, renewed by heartbeat, kept dormant while the chunk parks on a human ([humans.md](./humans.md)).
It is the runner's single-writer bookkeeping — the runner has no rival on its own machine — in deliberate contrast to acquisition, the contended grant at the hub.
Distinct concepts: acquisition decides *who holds the chunk*; the lease records *one attempt at one node*.

Each lease mints a fresh **epoch** — a counter that only rises across a chunk's attempts — and the epoch is the fence every state-advancing write is checked against.

The worker's **heartbeat** is a side effect of its tool use — no agent cooperation required — and is what keeps its lease alive.

## Stale attempts are fenced out (`bzh:epoch-fencing`)

**Rule.** Every state-advancing write for a chunk — a transition, a decision, its artifacts — carries the epoch of the lease that produced it, and a write below the chunk's newest epoch is rejected, never recorded; a terminal fact (stopped, delivered) rejects every later state-advancing write regardless of epoch.

**Why.** A worker presumed dead can wake and write after its successor started; fencing makes the successor authoritative — the zombie's late writes bounce — so a zombie can lose work but never land wrong work, without requiring anyone to kill processes reliably.

**Detect.** A state-advancing write path with no epoch check; a reap, reassignment, or forced-migration design that assumes the old holder is really dead instead of fencing it out; the epoch check skipped once a chunk is terminal.

**Do.** Reassignment mints new leases above a floor set over the old holder's newest epoch, so the old holder's in-flight submission is rejected on arrival.

**Don't.** Accept a transition because the submitting runner still holds the route — route tenure is not attempt fencing.

## What a worker receives

A worker session never discovers its work — the runner primes it with the **node envelope**, assembled by the hub for the chunk's current node:

- **The node's prompt and configuration** — the base prompt plus the taken edge's arrival context ([graphs.md](./graphs.md)).
- **The chunk's relevant artifacts** — earlier steps' outputs, resolved newest-first from the artifact series ([artifacts.md](./artifacts.md)).
- **The runner's machine-local context** — which environments the chunk holds and where they live on this machine; the hub contributes none of this.

Prompting is **two-phase**: the envelope's content instructs the work, and when the worker declares done, the judgement prompt is delivered into the same session to elicit the verdict ([graphs.md](./graphs.md) §Judgement and choices).
The envelope is also how change reaches a worker rather than being inferred: a migration shows up as the next envelope's new graph and node ([work.md](./work.md) §Migration), and an answered ask re-enters as the session resuming with the answer delivered into it ([humans.md](./humans.md)).

## Failure and recovery

- **Reap** expires a lease whose holder is gone — stale heartbeat, or a dead process. Reap ends the **attempt** — retry, or escalate on exhaustion ([humans.md](./humans.md)) — never by itself the chunk's tenure or its environments.
- **Reassignment** moves a held chunk to another runner — the supported exception to stickiness: the new runner rebuilds the environment from the chunk's commit artifacts, mints leases above the hub-supplied epoch floor, and may adopt unsubmitted in-progress work it finds ahead of the last submitted artifact commit.
- **Detach** forcibly releases a chunk from its runner: the route is released, the chunk re-derives ready, and the next claim's epoch floor fences the old runner.

## See also

- [./work.md](./work.md) — the transitions these leases produce and the statuses tenure derives.
- [../architecture/crash-correctness.md](../architecture/crash-correctness.md) — how the daemons are built and tested so these semantics survive `kill -9`.
- The discovery corpus (`blizzard-discovery` repo, `design/domain/fleet.md`, with the loop in `design/runner/loop.md`, the machine-local facts in `design/runner/store.md`, and the worker spawn in `design/harness-adapters.md`) — the property tables and the `D-007`/`D-021`/`D-024`/`D-027`/`D-035`/`D-038`/`D-043`/`D-044`/`D-049`/`D-063`/`D-072`/`D-080`/`D-082`/`D-088` decisions this file derives from.
