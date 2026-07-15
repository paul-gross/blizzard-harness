# Tooling

The external tools a blizzard agent drives beyond the winter CLI, and the surface that owns each one.
The winter CLI itself is workspace-owned (`workspace:/context/winter-cli/index.md`) and is not re-routed here.

Parent: [../index.md](../index.md).

| Tool surface | When to read |
|--------------|--------------|
| [./mise.md](./mise.md) | Running a repo task or discovering a code repo's command surface — the `mise` front door `blizzard` and `blizzard-mock` share |
| [../verification/blizzard.md#tools](../verification/blizzard.md#tools) | Standing up the scenario a verification needs — the setup tools the matrix owns |
| The `blizzard` repo's `docs/ci.md` | Watching or debugging a GitHub Actions run with `gh` — the in-repo operator reference for the `gh run` commands |
| `winter-github:/index.md` | Filing or refining a GitHub issue with `gh` — the issue skills and conventions |
