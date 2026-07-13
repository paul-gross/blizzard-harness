# Verifiability matrix — blizzard

An inventory of the verification methods for the blizzard ecosystem — each entry one way a skill or agent may assert a blizzard change is correct.
Conforms to the canon concept at `winter-canon:/verifiability-matrix.md`.

Method ids follow the canon's scheme at `winter-canon:/verifiability-matrix.md#method-identifiers`: commands and manual methods are `<scope>:<method>` (a manual method's method name is `manual`); tools are unscoped under a flat `tool:`.
Scopes here: **`blizzard`** for the app repo's Python QA and the daemon-level tiers, **`web`** for the Angular workspace checks, **`blizzard-mock`** for the mock-fleet repo.

## Seeded ahead of the code — every row is a named gap

As of **P5** the `blizzard` app scaffold is built — the two daemons (hub, runner) with their HTTP APIs, the CLI, both Alembic stores, the Angular frontend workspace, and the one-wheel build entrypoint — so its Python-QA, frontend, and wheel/gate rows are real commands (joining the `blizzard-mock` rows made real in P4). What remains ahead of its code is the **P6** upper tier: the service and e2e tiers and the kill-9 crash sweep, which need the walking-skeleton loop, the steppable daemons, and the mock hub/runner.
This matrix is therefore seeded **ahead** of the code it describes: a row with a live method states its real command, while a row for code that does not exist yet states its *intended* command or exercise and carries a **Gap (phase N)** marker naming the bootstrap phase that makes it real.
The gaps are recorded rather than omitted so a planner can see the full verification surface and know which methods it must help build.
When a phase lands its methods, drop the Gap marker from those rows in the same change.
Bootstrap phases: **P3** service manifests, **P4** `blizzard-mock` fleet, **P5** `blizzard` scaffold, **P6** the walking-skeleton acceptance loop.

## Commands

Verification that runs as a single command — exit 0 is the pass signal.

| Method | Command |
|--------|---------|
| blizzard:build | `uv sync` — install the `blizzard` project and its `dev` group (run from the repo root). |
| blizzard:lint | `uv run ruff check .` ([../standards/python.md](../standards/python.md)). |
| blizzard:format | `uv run ruff format --check .` ([../standards/python.md](../standards/python.md)). |
| blizzard:typecheck | `uv run pyright` ([../standards/python.md](../standards/python.md)). |
| blizzard:unit-test | `uv run pytest -m unit` — the unit tier: one class or function in isolation ([tiers](#test-tiers)). Bare `uv run pytest` runs the unit + component default suite. |
| blizzard:component-test | `uv run pytest -m component` — a domain slice wired with real internal collaborators, doubles only at the seams ([tiers](#test-tiers)). |
| blizzard:gate | `mise run gate` (`./scripts/ci-gate.sh`) — the local PR-to-master merge-gate parity: ruff format --check + ruff check + pyright + pytest, the OpenAPI spec-drift check, then eslint + vitest + generated-client drift over `web/`. One command reproduces the whole `pr` workflow before pushing. |
| blizzard:wheel | `mise run build` (`./scripts/build-wheel.sh`) — the one build entrypoint (D-061): builds both Angular apps into `src/blizzard/static/{hub,runner}`, builds the single wheel (`uv build --wheel`) embedding those assets plus both migration trees, then installs it into a clean **node-free** venv and runs `blizzard --version` — proving the released artifact needs no Node. `BLIZZARD_VERSION` overrides the wheel version (dev builds / tag releases). |
| blizzard:wheel-smoke | The exit-criterion serve smoke on the built wheel (node-free venv): `blizzard hub init <dir>` (idempotent, store migrated to head) then `blizzard hub host --dir <dir> --port <p>` serves the embedded mission-control board (`GET /` → the Angular `index.html` + hashed JS bundles, deep routes fall back to it) with `GET /api/health` → `200`; the same for `blizzard runner init`/`host` (the local-panel shell). This is the **P5 exit criterion** (`blizzard-discovery` repo, `implementation/bootstrap.md`). |
| blizzard:ci | `gh run watch --repo paul-gross/blizzard <run-id> --exit-status` — watch a GitHub Actions run (the `push` merge-gate on master, or the `pr` gate on a PR) to completion and exit non-zero if it failed; the authoritative remote gate. List runs with `gh run list --repo paul-gross/blizzard`; inspect a failure with `gh run view --repo paul-gross/blizzard <run-id> --log-failed` (the workflows and this watch loop are documented in the `blizzard` app repo's `docs/ci.md`). |
| blizzard:service-test | The service tier: a running hub or runner's HTTP API exercised from outside, all seams bound to the mock fleet — Playwright. **Gap (P6)**. |
| blizzard:e2e | The e2e tier: hub + runner + web app through the browser and CLI, fully local with every seam bound to the mock fleet — Playwright, a separate project from the service tier. **Gap (P6)**. |
| blizzard:crash-sweep | The kill-9 sweep: a pytest-driven runner iterating the crash-point registry, running the daemons as real subprocesses and asserting the invariant checker after each armed crash ([../architecture/crash-correctness.md](../architecture/crash-correctness.md)). **Gap (P6)** — depends on the steppable loop, injected clock, registry, and checker existing. |
| web:lint | `npm run lint` in `web/` — eslint over the Angular workspace, all four projects ([../standards/frontend.md](../standards/frontend.md)). |
| web:unit-test | `npm run test` in `web/` — vitest, the frontend unit/component tier over all four projects ([../standards/frontend.md](../standards/frontend.md)). |
| web:client-drift | `npm run generate:client` in `web/` (openapi-ts codegen from `openapi/{hub,runner}.openapi.json`), then fail on any uncommitted diff in `web/` ([../standards/frontend.md](../standards/frontend.md), `bzh:generated-client`). The Python half regenerates the specs via `uv run blizzard-export-openapi --out-dir openapi` and fails on a diff in `openapi/`. |
| blizzard-mock:build | `uv sync` in the `blizzard-mock` repo. |
| blizzard-mock:lint | `uv run ruff check .` ([../standards/python.md](../standards/python.md)). |
| blizzard-mock:format | `uv run ruff format --check .` ([../standards/python.md](../standards/python.md)). |
| blizzard-mock:typecheck | `uv run pyright` ([../standards/python.md](../standards/python.md)). |
| blizzard-mock:unit-test | `uv run pytest` — the default suite: unit + component coverage of the mock forge, the fixture-workspace scaffold, the mock coding-harness engine + façades, and the mock-data CLI (component tiers drive real git and a real `winter ws init`). |
| blizzard-mock:e2e | `uv run pytest -m e2e` — the fleet acceptance proof (`blizzard-mock` repo, `tests/test_acceptance_loop_e2e.py`): a scripted prompt run through the mock harness in a fixture-workspace env lands a commit the mock forge merges to bare `main`, all seams real — the **P4 exit criterion** (`blizzard-discovery` repo, `implementation/bootstrap.md`). Skipped without a discoverable winter source. |

## Test tiers

Four tiers, all used — each answers a different question, and none substitutes for another.
The mocks the upper tiers bind are owned by `blizzard-mock` (P4); the tier *rules* below are the standard those tests are held to.

| Tier | Method | Scope | Tooling |
|------|--------|-------|---------|
| **Unit** | `blizzard:unit-test` | One class or function in isolation. | pytest |
| **Component** | `blizzard:component-test` | A domain slice or subsystem wired with real internal collaborators, test doubles only at the seams. | pytest |
| **Service** | `blizzard:service-test` | A running hub or runner's HTTP API exercised from outside, seams bound to the mock fleet. | Playwright |
| **E2E** | `blizzard:e2e` | The full system — hub, runner, web app — through the browser and CLI, fully local with every seam bound to the mock fleet. | Playwright (separate project from the service tier) |

### Tier rules

- **Service and e2e tests never spend real tokens and never touch the network.** The harness seam binds a mock coding harness; the work-source and delivery seams bind the mock GitHub forge; the workspace seam binds mocks or local fixtures.
- **One-sided service tests use the mock counterpart.** Runner service tests run against the mock hub; hub service tests against the mock runner — edge cases come from driving the mock's levers, not from contriving the real daemon into rare states.
- **Test data is set up through the mock-data CLI and fixtures** (`tool:mock-data`), not ad-hoc SQL.
- **Tests run against sqlite.** Postgres support is a configuration concern held by staying inside SQLAlchemy's portable surface (`bzh:sql-portable`), not a second test matrix.
- **Crash correctness is an orthogonal dimension, not a fifth tier** — the kill-9 sweep (`blizzard:crash-sweep`) and its four architectural requirements are [../architecture/crash-correctness.md](../architecture/crash-correctness.md). The unit tier covers each step function's idempotency in isolation; the component tier drives steps in-process against the virtual clock; the sweep is the only piece needing real subprocesses and real signals.

## Manual testing

### blizzard:manual — the acceptance loop end-to-end
Surface: the walking skeleton — one chunk traveling ingest → acquire → mock-scripted commit → deliver → landed in a bare origin, with `done` derived from facts.
Setup: a fixture-workspace env (`tool:fixture-workspace`) with the hub, the runner, and the mock fleet bound; postgres or sqlite up via the service stack (`tool:service-up`).
Pass: the chunk lands in the bare origin and every store invariant holds, run fully locally with no tokens and no network.
**Gap (P6)** — this loop *is* the walking skeleton; once it passes it becomes the standing e2e smoke test (`blizzard:e2e`). Its P4 precursor already runs: `blizzard-mock:e2e` exercises the same ingest-less push→PR→merge→land arc with the mock fleet alone (no `blizzard` code), proving the seams before the daemons that drive them exist.

### blizzard-mock:manual — the live wired-service forge over a real fixture
Surface: the winter-wired mock forge (`tool:service-up`, band `+1`) fronting a real fixture workspace's per-env bare origins — the same single git truth the daemons will bind to, exercised out of process rather than in-test.
Setup — mint a fixture at the path the forge reads (`$BZ_FORGE_REPOS_DIR = ${BLIZZARD_MOCK_SCRATCH_ROOT}/${WINTER_ENV}/origins`), then bring the stack up. Run from the workspace root:
```
BLIZZARD_MOCK_SCRATCH_ROOT=/tmp/blizzard-mock/fixtures WINTER_ENV=alpha \
  sh -c 'cd alpha/blizzard-mock && uv run blizzard-mock-fixture mint --env alpha'
winter service up alpha --wait
```
The fixture's winter source resolves by walking up from the `blizzard-mock` worktree to the workspace root — **do not pass `--winter-source $PWD`**: inside a `cd … && …` subshell `$PWD` expands *after* the `cd`, so it names the `blizzard-mock` checkout (which has no `tools/winter-cli`) and minting fails. Let the walk-up default resolve it, or set `$BLIZZARD_MOCK_WINTER_SOURCE` to the workspace root explicitly.
Pass: `curl -fs localhost:${BZ_FORGE_PORT:-4421}/healthz` returns `ok`, and `curl -fs localhost:${BZ_FORGE_PORT:-4421}/repos/blizzard/toy-api` returns `200` with `"default_branch": "main"` — the live forge fronts the minted origins. Leave services down after (`winter service down alpha`; remove the fixture with `blizzard-mock-fixture destroy --env alpha`).

## Tools

Setup an agent uses to stand up the scenario a verification needs — not assertions of correctness themselves.

| Tool | Use |
|------|-----|
| tool:service-up | `winter service up <env> --wait` — bring up the verification stack for a feature env, port-band isolated. **As of P3** this brings up a **per-env postgres** (its own container + band port, `--wait` gated on a real `pg_isready` healthcheck); **as of P4** it also brings up the **mock GitHub forge** (band `+1`, from the env's `blizzard-mock` worktree, `--wait` gated on the forge's uvicorn ready-line log probe, fronting `$BZ_FORGE_REPOS_DIR` — the fixture workspace's per-env bare origins). Parallel envs isolate with zero port collisions. As of **P5** the `blizzard hub` / `blizzard runner` daemon binaries and their embedded boards exist and serve node-free (`blizzard:wheel-smoke`), but their tmux service slots (bands `+2`/`+3`) stay **commented** in the manifest — the live per-env daemons are wired here in **P6**, so `service-up` does not yet bring them up. |
| tool:mock-fleet | The `blizzard-mock` fleet. **As of P4** the mock GitHub forge (bare git repos, `blizzard-mock-forge`), the fixture-workspace scaffold, and the mock coding harness (prompt-is-the-program, `mock-claude-code`/`mock-codex`/`mock-opencode`) are built and bound at the seams so a scenario runs with no tokens or network. The **mock hub** and **mock runner** — the counterpart mocks the daemons are built against — remain **gaps** until the daemons need them (P5/P6). |
| tool:mock-data | The mock-data CLI (`blizzard-mock-data`) — seed the hub and runner stores into a known world for the upper tiers and the crash sweep. The CLI **skeleton** is built (P4); it **grows alongside the domain models it operates on**, so seeding real store state is a **Gap (P5+)** until those models exist. |
| tool:fixture-workspace | The fixture-workspace scaffold (`blizzard-mock-fixture`) — mints bare `file://` origins plus a generated, disposable winter workspace, the environment the service and e2e tiers and the sweep run against. **Built (P4).** |

## See also

- [../architecture/crash-correctness.md](../architecture/crash-correctness.md) — the four daemon requirements the `blizzard:crash-sweep` method exercises.
- [../standards/python.md](../standards/python.md) and [../standards/frontend.md](../standards/frontend.md) — the toolchains the command rows run.
- The discovery corpus (`blizzard-discovery` repo, `implementation/testing.md`, `implementation/mocking.md`, `implementation/verification.md`) — the tier and mock-fleet design in full.
