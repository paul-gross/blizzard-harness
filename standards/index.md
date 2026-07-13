# Standards

The code-quality conventions a finished blizzard change is held to — the toolchains, logging, and persistence rules, consulted both while writing the affected code and when reviewing whether it is up to standard before it lands.
Where [architecture/](../architecture/index.md) governs how code is *structured and designed*, this domain governs the *code-quality details* of the result.

The commands these standards name are proven by the verifiability matrix at [../verification/blizzard.md](../verification/blizzard.md); several are **gaps** until the code repos are scaffolded (phase 5), and the matrix marks each as such.

Parent: [../index.md](../index.md).

| Standard | When to read |
|----------|--------------|
| [./python.md](./python.md) | Writing or reviewing Python — the uv / ruff / pyright toolchain and the commands a change must pass |
| [./frontend.md](./frontend.md) | Writing or reviewing the Angular apps — the eslint-no-prettier / vitest toolchain and the committed, drift-checked generated API client |
| [./logging.md](./logging.md) | Adding a log call or picking a level — the structlog call-site and level conventions |
| [./persistence.md](./persistence.md) | Touching a store schema or a migration — the portable-SQL rule and the manual-Alembic migration policy |
