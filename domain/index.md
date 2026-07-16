# Domain — blizzard

Blizzard's **domain model**: what the concepts are, how they behave, and how they intertwine — with no technical detail.
Conforms to the `domain/` slot of the canon harness shape (`winter-canon:/harness-structure.md`).
This is the correctness reference: read it when planning a change against intent, or when reviewing or verifying behavior against the model — where the companion [architecture/](../architecture/index.md) domain governs how the *code* is structured, this domain governs how the *concepts* work.

The model was distilled from the discovery corpus (`blizzard-discovery` repo) — worth reading for the property-level design detail and the `D-xxx` provenance behind a shape, and each file here names its corpus counterpart in its See also. The corpus never governs a change and is never updated to match one: `workspace:/context/project/index.md` owns that policy.
This domain owns the behavioral statement an agent plans and reviews against. Where it and the **code** disagree, the code is current and this file is the one to fix.

Parent: [../index.md](../index.md).

| Doc | When to read |
|-----|--------------|
| [./work.md](./work.md) | Reasoning about a unit of work — what a chunk is, the statuses it can be in, how transitions move it, and how migration re-pins it across graphs |
| [./graphs.md](./graphs.md) | Reasoning about the workflow definition — graphs and their immutability, nodes, edges, judgements and choices, and the ids-exact/names-correlate philosophy |
| [./execution.md](./execution.md) | Reasoning about who runs a chunk — the hub/runner responsibility split, acquisition and routes, leases and epochs, what a worker session is primed with, and how tenure survives failure |
| [./artifacts.md](./artifacts.md) | Reasoning about what work produces and how it lands — the artifact kinds, the never-code rule, the chunk's artifact series, and delivery through the merge queue |
| [./humans.md](./humans.md) | Reasoning about where people enter the loop — asks, gate decisions, escalation, takeover, and the two parked conditions they produce |

## See also

- The discovery corpus (`blizzard-discovery` repo, `design/domain/`) — the property-level design model these files distill, and `decisions/log.md` — the `D-xxx` entries every behavior here derives from.
- The corpus `glossary.md` — the one-line term index for cold readers; the files here carry the full behavioral treatment.
