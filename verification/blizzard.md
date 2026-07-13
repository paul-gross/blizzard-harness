# Verifiability matrix — blizzard

An inventory of the verification methods for the blizzard ecosystem — each entry one way a skill or agent may assert a blizzard change is correct.
Conforms to the canon concept at `winter-canon:/verifiability-matrix.md`.

Method ids follow the canon's scheme at `winter-canon:/verifiability-matrix.md#method-identifiers`: commands and manual methods are `<scope>:<method>` (a manual method's method name is `manual`); tools are unscoped under a flat `tool:`.
Scopes here: **`blizzard`** for the app repo's Python QA and the daemon-level tiers, **`web`** for the Angular workspace checks, **`blizzard-mock`** for the mock-fleet repo.

## Seeded ahead of the code — every row is a named gap

The `blizzard` and `blizzard-mock` repos are README-only seeds at this point in the bootstrap; the frontend, the daemons, and the mock fleet do not exist yet.
This matrix is therefore seeded **ahead** of the code it describes: each row states the method's stable id and its *intended* command or exercise, and carries a **Gap (phase N)** marker naming the bootstrap phase that makes it real.
The gaps are recorded rather than omitted so a planner can see the full verification surface and know which methods it must help build.
When a phase lands its methods, drop the Gap marker from those rows in the same change.
Bootstrap phases: **P3** service manifests, **P4** `blizzard-mock` fleet, **P5** `blizzard` scaffold, **P6** the walking-skeleton acceptance loop.

## Commands

Verification that runs as a single command — exit 0 is the pass signal.

| Method | Command |
|--------|---------|
| blizzard:build | `uv sync` — install the `blizzard` project and its `dev` group. **Gap (P5)** — no `pyproject.toml` in the seed. |
| blizzard:lint | `uv run ruff check .` ([../standards/python.md](../standards/python.md)). **Gap (P5)**. |
| blizzard:format | `uv run ruff format --check .` ([../standards/python.md](../standards/python.md)). **Gap (P5)**. |
| blizzard:typecheck | `uv run pyright` ([../standards/python.md](../standards/python.md)). **Gap (P5)**. |
| blizzard:unit-test | `uv run pytest` — the unit tier: one class or function in isolation ([tiers](#test-tiers)). **Gap (P5)**. |
| blizzard:component-test | `uv run pytest` (component-marked) — a domain slice wired with real internal collaborators, doubles only at the seams ([tiers](#test-tiers)). **Gap (P5)**. |
| blizzard:service-test | The service tier: a running hub or runner's HTTP API exercised from outside, all seams bound to the mock fleet — Playwright. **Gap (P6)**. |
| blizzard:e2e | The e2e tier: hub + runner + web app through the browser and CLI, fully local with every seam bound to the mock fleet — Playwright, a separate project from the service tier. **Gap (P6)**. |
| blizzard:crash-sweep | The kill-9 sweep: a pytest-driven runner iterating the crash-point registry, running the daemons as real subprocesses and asserting the invariant checker after each armed crash ([../architecture/crash-correctness.md](../architecture/crash-correctness.md)). **Gap (P6)** — depends on the steppable loop, injected clock, registry, and checker existing. |
| web:lint | `npm run lint` — eslint over the Angular workspace ([../standards/frontend.md](../standards/frontend.md)). **Gap (P5)**. |
| web:unit-test | `npm run test` — vitest, the frontend unit/component tier ([../standards/frontend.md](../standards/frontend.md)). **Gap (P5)**. |
| web:client-drift | Regenerate the openapi-ts client and fail on any uncommitted diff ([../standards/frontend.md](../standards/frontend.md), `bzh:generated-client`). **Gap (P5)**. |
| blizzard-mock:build | `uv sync` in the `blizzard-mock` repo. **Gap (P4)**. |
| blizzard-mock:lint | `uv run ruff check .` ([../standards/python.md](../standards/python.md)). **Gap (P4)**. |
| blizzard-mock:typecheck | `uv run pyright` ([../standards/python.md](../standards/python.md)). **Gap (P4)**. |
| blizzard-mock:unit-test | `uv run pytest` — unit coverage of the mock forge, mock harness, and mock-data CLI. **Gap (P4)**. |

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
**Gap (P6)** — this loop *is* the walking skeleton; once it passes it becomes the standing e2e smoke test (`blizzard:e2e`).

## Tools

Setup an agent uses to stand up the scenario a verification needs — not assertions of correctness themselves.

| Tool | Use |
|------|-----|
| tool:service-up | `winter service up <env> --wait` — bring up the verification stack for a feature env, port-band isolated. **As of P3** this brings up a **per-env postgres** (its own container + band port, `--wait` gated on a real `pg_isready` healthcheck); parallel envs isolate with zero port collisions. The mock-forge (P4), `blizzard hub` (P5), and `blizzard runner` (P6) are reserved slots in the manifests — declared but empty, so still **gaps** until those phases fill them. |
| tool:mock-fleet | The `blizzard-mock` fleet — mock GitHub forge (bare git repos), mock coding harness (prompt-is-the-program), mock hub, mock runner — bound at the seams so a scenario runs with no tokens or network. **Gap (P4)**. |
| tool:mock-data | The mock-data CLI — seed the hub and runner stores into a known world for the upper tiers and the crash sweep. **Gap (P4)** — grows alongside the domain models it operates on. |
| tool:fixture-workspace | The fixture-workspace scaffold — bare `file://` origins plus a generated winter workspace, the environment the service and e2e tiers and the sweep run against. **Gap (P4)**. |

## See also

- [../architecture/crash-correctness.md](../architecture/crash-correctness.md) — the four daemon requirements the `blizzard:crash-sweep` method exercises.
- [../standards/python.md](../standards/python.md) and [../standards/frontend.md](../standards/frontend.md) — the toolchains the command rows run.
- The discovery corpus (`blizzard-discovery` repo, `implementation/testing.md`, `implementation/mocking.md`, `implementation/verification.md`) — the tier and mock-fleet design in full.
