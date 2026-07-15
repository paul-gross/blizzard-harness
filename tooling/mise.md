# The mise task front door (`bzh:mise`)

The code repos — `blizzard` and `blizzard-mock` — front their command surface with [mise](https://mise.jdx.dev): a `mise.toml` at each repo root pins the toolchain (`[tools]`, e.g. the uv version) and declares every routine command as a named, described task.
Agents, CI scripts, and humans invoke the same entrypoints, so a command proven in one context is identical in the others.

- **Discover** a repo's tasks with `mise tasks` from that repo's root — each task carries a description stating what it runs and what it needs.
- **Run** one with `mise run <task>`, also from the repo root. Tasks are repo-local; there is no workspace-level mise surface, and the markdown repos (`blizzard-harness`, `blizzard-discovery`) carry no `mise.toml`.
- **Trust** a fresh worktree on first use: when mise reports the repo's config untrusted, run `mise trust` in that repo root once.

What each verification task *asserts* — and the stable method id a plan or review cites it by — is owned by the verifiability matrix ([../verification/blizzard.md](../verification/blizzard.md)).
The mapping from tasks to the remote CI gates is the `blizzard` repo's `docs/ci.md`.
