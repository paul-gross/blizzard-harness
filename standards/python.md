# Python toolchain

The packaging, lint, format, and type-check toolchain every blizzard Python change — the hub, the runner, the CLI, and the `blizzard-mock` fleet — is held to.
Follows the slot skeleton owned by `winter-canon:/rule-shape.md` (`canon:rule-shape`).

## uv, ruff, pyright (`bzh:python-toolchain`)

**Rule.** Python is packaged with **uv** and held to **ruff** (lint *and* format) and **pyright**; the quality gates run from the first commit, not retrofitted.
The commands a change must pass:

| Check | Command |
|-------|---------|
| Install | `uv sync` — installs the project and its `dev` group |
| Lint | `uv run ruff check .` |
| Format | `uv run ruff format --check .` (write with `uv run ruff format .`) |
| Typecheck | `uv run pyright` |
| Test | `uv run pytest` (the unit and component tiers — [./frontend.md](./frontend.md) and [../verification/blizzard.md](../verification/blizzard.md) own the browser tiers) |

**Why.** One packaging tool and one lint/format/type toolchain, gated from the first commit, keeps quality cheap to satisfy — the cost of a clean tree is paid continuously rather than as a later cleanup — and gives every agent the same commands to run before pushing regardless of which blizzard component it touched.

**Detect.** A second formatter (black, autopep8) or packaging tool (poetry, pip-tools) introduced alongside these; Python changes pushed without the commands above passing.

**Do.** Run `uv run ruff check . && uv run ruff format --check . && uv run pyright && uv run pytest` before pushing a Python change.

**Don't.** Add black or isort "as well as ruff" — ruff owns both lint and format, and a second formatter is a second opinion that fights the first.

**Gap.** No `pyproject.toml` exists in the `blizzard` or `blizzard-mock` seeds yet; these commands land when the uv projects are scaffolded (phase 5). [../verification/blizzard.md](../verification/blizzard.md) tracks each as a named gap until then.
