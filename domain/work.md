# Work — the chunk and its lifecycle

The unit of work at the center of the model: the chunk, the statuses it derives, the transition record it moves by, and the migration that re-pins it across graphs.
Definitions, with the enforceable invariant written in the slot skeleton owned by `winter-canon:/rule-shape.md` (`canon:rule-shape`).
Part of the [domain model](./index.md).

## Chunk

The hub's unit of orchestrated work: it wraps one or more backlog items from the backing project-management system by pointer, travels a workflow graph, and accumulates artifacts, questions, and decisions as it goes.

- **The PM item is the durable referent; the chunk is ephemeral.** An unacquired chunk may be discarded or grouped away, and re-ingesting the same item mints a fresh chunk — nothing of value lives only on an unstarted chunk. An item already wrapped by a live chunk cannot be ingested again.
- **The item's contents are never stored.** A chunk holds pointers; reads pass through to the backing system.
- **Pinned to exactly one immutable graph.** The pin is set at mint and changes only when a [migration](#migration) applies — never silently.
- **Nothing on it is a stored status.** Its current node derives from its newest accepted transition, and its status derives from its recorded facts (`bzh:facts-not-status` in [../architecture/system-shape.md](../architecture/system-shape.md)).
- **Held by at most one runner at a time** — see [execution.md](./execution.md) for acquisition, tenure, and fencing.

## Statuses

The derived conditions a chunk can be in.
A chunk has exactly one status at a time — the conditions are checked in a fixed precedence, and none is ever a stored column.
The exact fact vocabulary and derivation queries are owned by the corpus (`blizzard-discovery` repo, `design/domain/events.md`); this table is the behavioral meaning.

| Status | Meaning |
|--------|---------|
| `ready` | Ingested into a chunk and unclaimed — in the hub's queue with no live route. |
| `running` | Claimed by a runner and being worked. |
| `delivering` | In the hub's own hands — queued for or undergoing delivery, or awaiting an external merge; the holding runner keeps its environments until the outcome is known. |
| `waiting_on_human` | Parked on **invited** human input — an open ask or an unresolved gate decision ([humans.md](./humans.md)); the reap clock is stopped. |
| `needs_human` | Parked on **failure** — retries exhausted, or a worker died without a verdict; a person must requeue or take over ([humans.md](./humans.md)). |
| `stopped` | Abandoned by an operator — terminal at any point after acquisition, artifacts and history retained. |
| `done` | Delivered — the chunk's commit artifacts landed through the merge queue, or its PR reached a terminal state externally ([artifacts.md](./artifacts.md)). |

Discarding or grouping an unacquired chunk is not a status: the chunk is simply gone from every listing, because the PM item remains the durable referent.

## Transition

One entry in a chunk's append-only movement record: the fact that a judgement at a node's exit selected an edge and the chunk moved along it.

- **Every transition is fully formed.** The judgement is what selects the edge — the worker's verdict, the hub's own machinery at a hub node, or a human's choice at a gate — so there is no unjudged movement and no transition without a judgement.
- **At a gate there is no transition until the human decides.** The node-step's completion lands as an open decision, and the resolving choice writes the transition referencing it ([humans.md](./humans.md)).
- **Atomic with the step's artifacts.** A node-step's transition and its artifacts are committed as one write — a rejected transition's artifacts never exist ([artifacts.md](./artifacts.md)).
- **Fenced.** A transition carries its attempt's epoch and a stale one is rejected, never recorded (`bzh:epoch-fencing` in [execution.md](./execution.md)) — the movement record only ever advances on a live attempt's say-so.
- **Authored by the holder.** The holding runner reports the chunk's transitions; the hub's own executor authors them for hub-executed nodes.

## A migration is never a transition (`bzh:migration-not-transition`)

**Rule.** Movement between graphs is a **migration** — its own recorded fact re-pinning the chunk — never a transition: a transition moves a chunk along an edge within its pinned graph, and no edge crosses graphs.

**Why.** Transitions are judged, fenced movement within one immutable definition; letting one span graphs would re-route in-flight work without the explicit intent, record, and fencing that migration provides.

**Detect.** A design or change in which a transition's two nodes belong to different graphs, an edge targets a node of another graph, or a chunk's graph pin changes with no migration record.

**Do.** Re-pin via a migration record — deferred to the chunk's next transition, or forced with a fresh epoch — landing on the name-matched node, or the target's entry node when no name matches.

**Don't.** Add a cross-graph edge, or update a chunk's pinned graph in place with no record of the re-pin.

## Migration

The explicit re-pin of a chunk from one immutable graph to another.

- **Intent and fact are separate.** A migration *request* is intent; a migration *record* is the applied fact. A chunk's pending migration derives — a request no record references — and the newest unapplied request wins; superseded requests never produce a record.
- **Two modes.** *Deferred* applies at the chunk's next transition — so a parked chunk never migrates until it resumes. *Forced* applies immediately — the hotfix path: it revokes the in-flight attempt with a fresh epoch, and the abandoned work is redone under the new graph.
- **Standing drift is graph metadata, not per-chunk requests.** "Always move on" lives on each graph as its auto-migrate policy and migration-target pointer ([graphs.md](./graphs.md)); a chunk's pending auto-migration derives from its pinned graph's pointer, one hop per transition, so chains self-correct as pointers change.
- **Landing is by name.** The chunk lands on the target graph's node whose name matches the one it sat at (`bzh:ids-exact-names-correlate` in [graphs.md](./graphs.md)); with no match, the target's entry node.

## See also

- [./graphs.md](./graphs.md) — the immutable definition a chunk travels and the metadata its auto-migration derives from.
- [./execution.md](./execution.md) — who holds a chunk, the lease behind each node-step attempt, and the epoch its transitions are fenced by.
- The discovery corpus (`blizzard-discovery` repo, `design/domain/work.md` and `design/domain/events.md`) — the property tables, fact vocabulary, and derivation queries, and the `D-004`/`D-024`/`D-027`/`D-033`/`D-034`/`D-037`/`D-045`/`D-047`/`D-067` decisions this file derives from.
