# Artifacts and delivery

What work produces and how it lands: the artifact kinds, the chunk's artifact series, and the merge queue.
Definitions, with the enforceable invariant written in the slot skeleton owned by `winter-canon:/rule-shape.md` (`canon:rule-shape`).
Part of the [domain model](./index.md).

## Artifact

A node-step's durable output, stored at the hub and fed into later nodes' work.
Two kinds:

| Kind | Carries |
|------|---------|
| commit pointer | A repository, a branch name, and a commit hash — the branch is pushed to the forge **before** the artifact is submitted, so the pointer never dangles. A chunk touching five repos submits five pointers. |
| asset | Text or a blob — a review's findings, a spike write-up. |

- **The hash is authoritative.** Branches move, so the hash pins the state that was actually verified; the branch name serves only to detect work committed ahead of it. There is deliberately no fencing at the branch ref: a zombie clobbering a branch can lose work, never land wrong work (`bzh:epoch-fencing` in [execution.md](./execution.md)).
- **Self-describing provenance.** An artifact knows the chunk, the exact node, and the attempt that produced it.

## Never code, never transcripts (`bzh:never-code`)

**Rule.** The hub stores **references** to work, never the work: a commit pointer is the closest any hub-side model gets to code, and no model anywhere stores agent-session transcripts.

**Why.** The forge is already the durable owner of code and the runner's machine of its sessions; a hub that holds only references stays small, safe to centralize, and safe to expose to the board.

**Detect.** A design or schema persisting file contents, diffs, patches, or session transcripts at the hub; an artifact carrying code instead of a pointer to it; a PM item's contents stored rather than read through.

**Do.** Push the branch to the forge, then submit the repository, branch, and commit hash as the pointer artifact.

**Don't.** Attach a diff or the worker's transcript as an asset artifact "for review convenience" — the review reads the pushed branch instead.

## The chunk's artifact series

A chunk's artifacts accumulate as an **append-only, versioned series** per node and artifact name.

- **Committed with the step, atomically.** Artifacts land in the same fenced write as the step's transition — or its gate decision — so a rejected step's artifacts never exist and can never drift from the movement record. There is no separate artifact submission.
- **Append, never overwrite.** Re-running a node adds new entries under the new attempt; earlier entries remain as history.
- **Reads resolve to the newest entry.** Later nodes fetching an artifact by name get the latest attempt's version; the shadowed history stays available.
- **Series key on the node *name*.** After a migration or a re-published graph, a re-run of `build` keeps appending to the same series (`bzh:ids-exact-names-correlate` in [graphs.md](./graphs.md)); the exact producing node is on each artifact's provenance.

## Delivery and the merge queue

The **merge queue** is the hub's single-writer delivery lane: the hub-executed deliver node lands one chunk's commit-pointer artifacts at a time, strictly first-in-first-out, fenced like any other step.

- **Per-repo landing with reconciliation.** A multi-repo chunk lands serially per repo, each land its own recorded fact; a retry after a partial delivery skips what already landed.
- **Conflict is a judged outcome, not an error.** A merge or rebase conflict routes the chunk along a conflict edge within its graph ([graphs.md](./graphs.md)); the partial lands are retained for the redelivery's reconciliation.
- **PR mode** delivers by opening a pull request instead of merging directly: the chunk is awaiting the external merge, and the PR reaching a terminal state — merged or closed — completes the chunk either way.
- The holding runner **keeps the chunk's environments throughout delivery**, until the outcome is known.

## See also

- [./work.md](./work.md) — the transition an artifact commits with, and the `done` status delivery derives.
