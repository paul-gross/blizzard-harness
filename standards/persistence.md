# Persistence

The rules a store schema and its migrations are held to.
These sit under the facts-only store invariant (`bzh:facts-not-status`, [../architecture/system-shape.md](../architecture/system-shape.md)); this file governs the *SQL surface* and the *migration policy* that invariant is realized through.
Each rule follows the slot skeleton owned by `winter-canon:/rule-shape.md` (`canon:rule-shape`).

## Stay inside SQLAlchemy's portable surface (`bzh:sql-portable`)

**Rule.** The hub store and the runner store each run on **sqlite or postgres**, selected by configuration; neither daemon may depend on backend-specific behavior.
sqlite is the fast local default and what tests run against; postgres support is held by staying inside SQLAlchemy's portable surface — **not** by a second test matrix.

**Why.** One portable schema means postgres support costs a configuration switch, not a duplicated test suite, and the fast sqlite default keeps the local and CI loops cheap. A backend-specific feature (a postgres-only type, a sqlite-only pragma relied on for correctness) forks the schema and forces the second matrix the rule exists to avoid.

**Detect.** A dialect-specific column type, function, or `text()` SQL that only one backend supports; a test asserting behavior that holds on postgres but not sqlite (or vice versa); a code path branching on the configured backend.

**Do.** Model with SQLAlchemy's portable types and expressions; run tests against sqlite; treat postgres as the same schema under a different URL.

**Don't.** Reach for a postgres-only type or a sqlite-only pragma that changes correctness — the schema is no longer one portable surface and now needs two test runs.

## Migrations are manual, Alembic, CLI-driven (`bzh:manual-migrations`)

**Rule.** Schema change is **Alembic**, applied manually through the CLI — never automatically at daemon startup.

- **Two migration trees**, one per store (hub, runner): independent schemas, independent lifecycles.
- Every revision has explicit `upgrade()` and `downgrade()`; applying is idempotent at the revision level, and scripts stay inside the portable SQL surface (`bzh:sql-portable`) so one tree serves sqlite and postgres.
- `blizzard hub init <dir>` creates and migrates the runtime environment and is idempotent; `blizzard hub migrate` applies pending revisions (`migrate --down <rev>` reverses). The same verbs exist under `blizzard runner`.
- **Daemons never migrate.** At startup each daemon compares the store's revision to the one it expects and **refuses to run on mismatch**, naming the exact migrate command.

**Why.** A version skew that fails loud — naming the command to fix it — beats a daemon that silently rewrites a schema out from under running data; manual, CLI-driven migration keeps the schema change an explicit, reviewable, reversible step. Two trees keep the hub's and runner's schemas from coupling their release cadences.

**Detect.** An `alembic upgrade` (or `create_all`) call on a daemon's startup path; a revision missing a real `downgrade()`; a single migration tree spanning both stores; dialect-specific migration SQL.

**Do.** Write paired `upgrade`/`downgrade` revisions per store tree; apply them through `blizzard hub migrate` / `blizzard runner migrate`; let the daemon refuse to start on revision mismatch.

**Don't.** Auto-migrate on boot, or ship a revision whose `downgrade()` is a stub — an unattended migration can corrupt data, and a stubbed downgrade makes a rollback impossible.

**Gap.** No Alembic trees, `blizzard` CLI, or store schemas exist in the seed yet; the `init`/`migrate` verbs and the startup revision check land with the app scaffold (phase 5).
