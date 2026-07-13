# blizzard-harness

Blizzard's conventions harness — the rules every piece of blizzard code and every blizzard agent context is held to.
blizzard-harness derives from **winter-canon**: the canon defines what any harness must be, and this repo is blizzard's instance of one.
It follows `winter-harness` in **style** — domain-organized convention directories, routing hubs, a verifiability matrix, and architecture guidance — while carrying **blizzard's own rules**, not winter's.

## Path notation

Files here are addressed with the `blizzard-harness:` prefix — for example, `blizzard-harness:/architecture/repository-access.md`.
Resolve to the on-disk path via the `# Winter Extensions` block in workspace `CLAUDE.md`; the local directory name varies (`./.winter/ext/harness/`, `./blizzard-harness/`, …).
The universal substrate this harness derives from is addressed with the `winter-canon:` prefix — e.g. `winter-canon:/harness-structure.md`.

## Rule ids

Every blizzard-harness rule carries a stable `bzh:<slug>` id in its heading — the citation handle a plan, a review finding, or a cross-reference uses.
The `bzh:` scheme is this harness's own, parallel to the canon's `canon:<slug>` (`winter-canon:/rule-shape.md` owns the slot skeleton and the stable-id scheme both follow).
Ids are stable: citations depend on them, so renaming or removing a rule's id is a breaking change. A rule's heading is the id's single home — there is no separate id registry to keep in sync.

## Domains

| Domain | When to read |
|--------|--------------|
| [architecture/](./architecture/index.md) | Planning a change to blizzard's structure, or reviewing a plan — the CLEAN layering, repository access, the deterministic-shell/pluggable-seam shape, and the crash-correctness requirements the daemons are built to honor |
| [standards/](./standards/index.md) | Writing or reviewing finished code — the Python and Angular toolchains, logging, persistence, and the generated client a change is held to |
| [verification/](./verification/index.md) | Planning how a change will be proven, or verifying one — the verifiability matrix: the four test tiers, tier rules, and the per-component commands |
| [exemplars/python/repo_pattern.py](./exemplars/python/repo_pattern.py) | Building a repository — the reference shape for the Protocol-seam + internal-adapter + injected-error pattern the architecture rules require |
| [CONTRIBUTING.md](./CONTRIBUTING.md) | Committing to this repo — commit format, delivery, and the pre-push expectation |

## Domains not yet present

The canonical harness domain set (`winter-canon:/harness-structure.md`) also names `domain/`, `workflows/`, and `tooling/`.
These are **gaps**, named here rather than silently omitted:

- **`domain/`** — the business/domain model with no technical detail. Its content lives today in the discovery corpus (`blizzard-discovery` repo, `domain/` and the decision log); it graduates into this harness when the model stabilizes.
- **`workflows/`** — deterministic agentic processes toward a goal (feature delivery, release). The cross-repo delivery flow lives in `workspace:/context/project/contributing.md` and `workspace:/context/worktree-ops.md`; a blizzard-specific `workflows/` domain is unfilled.
- **`tooling/`** — additional external tools an agent drives through the harness. Unfilled; add one when blizzard gains a tool an agent invokes beyond the winter CLI.
