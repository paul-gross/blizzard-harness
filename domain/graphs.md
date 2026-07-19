# Workflow graphs

The definition chunks travel: graphs, nodes, edges, judgements, and choices.
Definitions, with the enforceable invariant written in the slot skeleton owned by `winter-canon:/rule-shape.md` (`canon:rule-shape`).
Part of the [domain model](./index.md).

## Graph

One identity, two parts:

- **An immutable definition** — the nodes, edges, prompts, and judgements. Every edit creates a new graph; an existing definition never changes, so anything pinned to it can trust it forever.
- **Mutable operational metadata beside it** — `enabled`, the auto-migrate policy, and the migration-target pointer. The metadata is the graph's only mutable surface.

There is **no graph family or version tree**: graphs are standalone, and any graph may migrate its chunks to any graph so long as the node mapping gets them over.
What looks like "versions of a workflow" is emergent — a chain of migration-target pointers between graphs that happen to share a name.

- **`enabled` gates exactly one thing: being auto-migrated *to*.** While a graph is disabled no chunk drifts into it — and nothing else changes: its own chunks continue, explicit requests may still target it. Graphs are created enabled.
- **The migration target may point anywhere.** Ordinarily the graph's re-published successor; pointing at a differently-named graph retires this workflow into another one.

## Node

One station in one graph — "build", "review", "deliver".
A node belongs to exactly one immutable graph; same-named nodes in different graphs are distinct nodes correlated only by name.
There is no node *type*: what a type would encode is structural — a gate is a node whose judgement is human-judged, and a node whose `executor` is the hub itself is one the hub runs directly, no agent, ever ([standards/hub-nodes.md](../standards/hub-nodes.md)).

| Facet | Meaning |
|-------|---------|
| name | The cross-graph correlator — what migration landing, artifact series, and runner-side gate configuration key on. |
| executor | Who runs the node's steps: a runner (the default), or the hub itself — a hub-executed node declares its steps as `run:`, a shipped command list, never an agent turn ([standards/hub-nodes.md](../standards/hub-nodes.md)). The shipped `deliver` node is one instance of this shape, not a distinct kind. |
| prompt | The node's invariant identity — what a worker at this station is asked to do; the arriving edge contributes arrival context on top. |
| checks | The deterministic checks (tests, lint) whose results inform the exit judgement. |
| judgement | How the node's exit is judged and the choices it can produce — see [Judgement and choices](#judgement-and-choices). |
| retries | The bounded failure budget — crashes, verdict-less exits, reaps — and where exhaustion escalates; a *judged* failure edge never consumes it. |
| produces | The artifact names the node is expected to submit ([artifacts.md](./artifacts.md)). |
| session | Whether the node's steps resume the chunk's agent session or start cold — resumed by default; review-style nodes opt into cold eyes. |

## Edge

A directed, outcome-keyed connection between two nodes of the **same graph** — cross-graph movement is a migration, never an edge (`bzh:migration-not-transition` in [work.md](./work.md)).

- **Keyed by exactly one choice** of the source node's judgement, and every choice a judgement can produce has exactly one edge — resolution is checked when the graph is created, so an edge can never dangle and a judgement can never select nothing.
- **Exactly one entry node** per graph; **cycles are intentional** — build → review → build is the shape, not a validation error.
- An edge carries **arrival context**: prose appended to the target node's prompt so the worker knows how it got there.

## Judgement and choices

The evaluation at a node's exit that selects the outgoing edge.

- **Judged by the worker** (the default): elicited when the worker declares done, informed by the checks it ran; the worker selects exactly one of the node's choices. A missing or unparseable selection is a **failure, not a judgement** — it consumes a retry rather than an edge.
- **Judged by a human**: the structural mark of a gate — the person renders the judgement by picking from the same choices, presented as buttons ([humans.md](./humans.md)).
- **Judged by a hub-executed node's own script**: its declared `run:` steps select one of the node's authored choices by exit code and stdout, the same fused choice/edge shape a worker's judgement uses — e.g. the shipped `deliver` node's script selecting `landed` or `conflict` ([artifacts.md](./artifacts.md), [standards/hub-nodes.md](../standards/hub-nodes.md)).

A **choice** is one selectable outcome of one node's judgement, scoped to that judgement — never a global registry: `pass` in build and `pass` in review are different choices that happen to share a name.
Each choice keys exactly one outgoing edge; its description is what sharpens a worker's judgement and what a gate renders as button text.

- **A choice may target another graph.** A choice's `to:` normally names a sibling node or the reserved terminal; it may instead name `graph:<name>` — a cross-graph **migration** target. Taking that choice re-pins the chunk to the named graph and re-queues it at the landing node (name-matched, else the target's entry), recording a migration fact, never a transition — so `bzh:migration-not-transition` in [work.md](./work.md) still holds (the authored choice carries the target; the *movement* is a migration, not an edge to another graph's node, which stays forbidden). A choice may also carry an optional `model:` the migration re-pins the chunk to. The target is resolved by **name** when taken (late-bound, the same binding ingest uses, `bzh:ids-exact-names-correlate`), so authoring a choice whose target graph is not yet minted is a mint-time warning, not an error.

## Ids are exact, names correlate (`bzh:ids-exact-names-correlate`)

**Rule.** Exact references — a transition's nodes, an artifact's provenance, an edge's choice — carry **ids**, which pin one immutable definition; continuity across graphs — migration landing, artifact series, runner-side gate selection — keys on **names**. Never the reverse.

**Why.** An id names exactly one thing in exactly one immutable graph, so an exact reference can never dangle or drift; a name is the only thing same-purpose entities in different graphs share, so it is the only key correlation can use.

**Detect.** A design that matches an exact reference by name — two graphs' `build` nodes conflated — or that correlates across graphs by id, such as a migration expecting the target graph to contain the same node id.

**Do.** A transition records the exact ids of its two nodes; a deferred migration lands the chunk on the target graph's node whose *name* matches the one it left.

**Don't.** Key an artifact series on the node id — the series would break at every migration or re-published graph, though the work is the same station's.

## See also

- [./work.md](./work.md) — the chunk that travels this definition, and the migration that moves it between definitions.
- [../standards/hub-nodes.md](../standards/hub-nodes.md) — the technical authoring schema for a hub-executed node: the `run:` step shape, the injected env-var contract, the outcome protocol, and the per-step idempotence rule.
