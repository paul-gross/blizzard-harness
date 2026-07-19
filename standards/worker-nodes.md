# Worker command nodes

The authoring contract for a worker node — `executor: runner` (the default, [../domain/graphs.md](../domain/graphs.md) §Node) — that declares a `produces:` asset: the attach instruction its prompt owes the worker, the fallback and hub-side backstop that exist when it doesn't, and the identity env the worker's `blizzard runner attach` call reads.
[../domain/graphs.md](../domain/graphs.md) §Node and [../domain/artifacts.md](../domain/artifacts.md) own the concepts — the node-level `produces:` list and the asset artifact kind; this file owns the prompt-authoring obligation and hub backstop a graph author is held to, the same relationship [./hub-nodes.md](./hub-nodes.md) has to `executor: hub`.
Each rule follows the slot skeleton owned by `winter-canon:/rule-shape.md` (`canon:rule-shape`).

## The attach instruction (`bzh:worker-node-attach-instruction`)

**Rule.** A worker node's `produces:` list names assets the *worker* is expected to submit ([../domain/graphs.md](../domain/graphs.md) §Node); for every name on that list, the node's prompt (and its judgement prompt, when the two disagree on where the instruction lands) must instruct the worker to run `blizzard runner attach --name <exact-produces-name>`, reading the asset's content from stdin. A node declaring several names attaches each under its own name — one `attach` call per name — rather than leaning on the judgement-assessment fallback (`bzh:worker-node-attach-fallback` below) to cover more than one.

**Why.** The completion assembly has no file convention for an asset — it can only submit what the worker explicitly attaches or, failing that, alias the whole node's judgement assessment to a name — so an un-instructed worker silently produces the weaker fallback instead of the artifact the graph actually asked for, and a multi-name node loses per-name provenance entirely if it relies on that alias.

**Scope.** Only asset-producing worker nodes carry this obligation — a git-commit-covered `produces:` name (one a repo's pushed commit already satisfies, `bzh:worker-node-produces-backstop` below) needs no attach, and a hub node's step-level `produces` marker (`./hub-nodes.md` `bzh:hub-node-run-shape`, Scope) is recorded by the executor itself, never by a worker.

**Detect.** A worker node's `produces:` list naming an asset with no `attach --name <that name>` string anywhere in its prompt text; a multi-asset node's prompt instructing one attach and expecting it to cover every declared name.

**Do.** `review.md` (the shipped `review` node's prompt): *"run `blizzard runner attach --name review-findings` with the content on stdin"* — matching the node's own `produces: [review-findings]` in `src/blizzard/hub/graphs/default.yaml` exactly. Every packaged graph's asset-producing node is held to this exact-name match by a standing unit-tier guard (`tests/test_packaged_prompts_attach.py`, cited in `blizzard:unit-test`, [../verification/blizzard.md](../verification/blizzard.md)).

**Don't.** A prompt telling the worker to "write the findings as your judgement answer" or to a file — neither reaches the attach path, so the node falls back silently (`bzh:worker-node-attach-fallback`).

## The judgement-assessment fallback (`bzh:worker-node-attach-fallback`)

**Rule.** A `produces:` name with no pushed git commit and no explicit `attach` is not left empty: the completion assembly falls back to submitting the worker's judgement assessment as that name's asset content, unattached. This fallback is a legitimate landing artifact — nothing rejects its content — but it is a fallback, never the intended path, and it cannot express more than one artifact: a node with several `produces:` names and one un-attached judgement assessment aliases every missing name to the same text, losing the per-name distinction the node declared.

**Why.** The engine cannot invent content a worker never produced, so it degrades to the one piece of prose every judged node already has rather than failing the attempt outright — but that degradation is exactly the gap `bzh:worker-node-produces-backstop` exists to catch.

**Detect.** A node with two or more `produces:` names where the prompt instructs at most one `attach` call, or none.

**Do.** Author the prompt so every declared name gets its own `attach --name <name>` instruction (`bzh:worker-node-attach-instruction`), leaving the fallback to cover only a genuinely un-instructed or non-compliant run.

**Don't.** Design a multi-asset node assuming the fallback will "sort itself out" per name — it aliases one assessment across every missing name, not one fallback per name.

## The `produces_mode` backstop (`bzh:worker-node-produces-backstop`)

**Rule.** The hub config's `produces_mode` (mirroring `route_token_mode`'s shape) gates what an unattached `produces:` name costs at completion time: under `enforce`, a submission with one or more `produces:` names lacking an explicit attachment is **rejected** outright; the shipped default is `warn`, which only logs the gap and lets the submission proceed on the fallback. A name a pushed git commit already covers is satisfied by that commit and never counts as missing, attached or not.

**Why.** `enforce` exists as a rollout brake an operator opts into once every packaged and custom graph's prompts are known to instruct the attach correctly — landing it as the default would reject completions from graphs nobody has audited yet.

**Detect.** A custom graph's worker node declaring `produces:` with a prompt that doesn't satisfy `bzh:worker-node-attach-instruction`, running under a hub configured `produces_mode = "enforce"` — every completion from that node fails until the prompt is fixed.

**Do.** Flip `produces_mode` to `enforce` only after auditing every graph the hub runs against `bzh:worker-node-attach-instruction` — the packaged graphs already pass this audit via the standing guard cited above.

**Don't.** Ship a new custom worker node with a `produces:` name and an un-instructed prompt against a hub already running `produces_mode = "enforce"` — expect every completion from that node to be rejected, not silently degraded.

## The worker's identity env (`bzh:worker-node-attach-env`)

**Rule.** The `blizzard runner attach` CLI call the worker runs reads its identity from the spawn environment, not from arguments the node's prompt has to thread:

| Var | Carries |
|-----|---------|
| `BLIZZARD_LEASE_ID` | The worker's current lease id — which chunk/node/attempt the attachment is recorded against. |
| `BLIZZARD_RUNNER_URL` | The runner's local API `attach` posts to. |
| `BLIZZARD_LEASE_TOKEN` | The lease's minted capability token, sent as `X-Blizzard-Lease-Token` — authorizes the call; sent only when present. |

**Why.** A prompt only ever names `--name <produces-name>` — the identity triple is the runner's own concern, injected once at spawn, so a node's author never has to carry or leak a lease id or token into authored prose.

**Scope.** These three vars are shared by every worker CLI the runner spawns with (`ask`, `heartbeat`, `session-end`), not `attach`-specific; `attach` is the one that also reads `BLIZZARD_LEASE_TOKEN`, since it durably records content rather than a soft-fail signal.

**Detect.** A node prompt instructing the worker to pass a lease id, runner URL, or token explicitly — there is nothing in the `attach` CLI's own signature (`src/blizzard/runner/cli.py`) for such an argument to bind to.

**Do.** *"run `blizzard runner attach --name <name>` with the content on stdin"* — the CLI resolves lease id, runner URL, and token itself.

**Don't.** *"run `blizzard runner attach --name <name> --lease <lease-id>`"* — no such flag exists; the worker cannot supply an identity the CLI already has.

## See also

- [../domain/graphs.md](../domain/graphs.md) — the conceptual node model: the `executor` facet and the node-level `produces:` list this file's prompt obligation attaches to.
- [../domain/artifacts.md](../domain/artifacts.md) — the asset artifact kind an attach submits, and the commit-pointer kind a `produces:` name may instead be satisfied by.
- [./hub-nodes.md](./hub-nodes.md) — the parallel authoring contract for `executor: hub`, including the Scope note distinguishing its step-level `produces` marker from the node-level list this file governs.
- [../verification/blizzard.md](../verification/blizzard.md) — `blizzard:unit-test`, whose packaged-prompt attach guard (`tests/test_packaged_prompts_attach.py`) proves `bzh:worker-node-attach-instruction` against every packaged graph.
