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
A non-rule leaf — a procedure or taxonomy file (`winter-canon:/rule-shape.md` §File kinds) — carries a file-level `bzh:<slug>` id in its title instead.
Ids are stable: citations depend on them, so renaming or removing a rule's id is a breaking change. A rule's heading is the id's single home — there is no separate id registry to keep in sync.

## Domains

| Domain | When to read |
|--------|--------------|
| [domain/](./domain/index.md) | Establishing or asserting correctness of behavior — planning against what blizzard's concepts are and how they behave, or reviewing and verifying work against that model, with no technical detail |
| [architecture/](./architecture/index.md) | Planning a change to blizzard's structure, or reviewing a plan — the CLEAN layering, repository access, the deterministic-shell/pluggable-seam shape, and the crash-correctness requirements the daemons are built to honor |
| [standards/](./standards/index.md) | Writing or reviewing finished code — the code-quality rules a change is held to, from the Python and Angular toolchains to what a value looks like on the wire |
| [verification/](./verification/index.md) | Planning how a change will be proven, or verifying one — the verifiability matrix: the four test tiers, tier rules, and the per-component commands |
| [workflows/](./workflows/index.md) | Reasoning about how work reaches `master` — feature delivery is blizzard-orchestrated, not agent-driven — or carrying out the release cut, the one deterministic sequence an agent still drives |
| [exemplars/python/repo_pattern.py](./exemplars/python/repo_pattern.py) | Building a repository — the reference shape for the Protocol-seam + internal-adapter + injected-error pattern the architecture rules require |
| [tooling/](./tooling/index.md) | Driving an external tool beyond the winter CLI — the repo task runner, `gh` for CI runs and issues, or the verification-scenario setup tools |
| [CONTRIBUTING.md](./CONTRIBUTING.md) | Committing to this repo — commit format, delivery, and the pre-push expectation |
