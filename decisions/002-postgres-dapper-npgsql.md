# 002 — PostgreSQL with Dapper + Npgsql

**Status:** Decided, including migration tool (**DbUp**, 2026-11-23).

## Context
Need a relational store for metadata, users, playlists and billing.

## Options considered
- Dapper + Npgsql with raw parameterized SQL (chosen)
- EF Core (excluded; the original draft's EF Core references were corrected, see [Corrections](#corrections))

## Choice
PostgreSQL, accessed with Dapper + Npgsql. **No EF Core.** Search uses Postgres full-text + `pg_trgm`.

## Consequences
- SQL constants per repository, always parameterized; snake_case in Postgres, PascalCase row models.
- Unique/foreign-key violations (SQLSTATE `23505`/`23503`) become specific errors.
- Dapper has no built-in migrations. **Migration tool: DbUp** — plain `.sql` files authored in `vibra-db` (one folder per module) and synced into each module as embedded resources, run in order, tracked in a journal table (one journal per module). Runs in-process, so compose (Podman) and Testcontainers tests use the same path.
- Alternatives considered: dbmate/Flyway (external CLI, awkward per-module sets), FluentMigrator/Evolve (more ceremony), hand-rolled scripts (would reimplement DbUp's tracking).
- DbUp is a new NuGet package: **ask before adding** (this decision records the choice; the approval to add it is given at that time).
- **`vibra-db` scope (decided 2026-11-23, "everything database related lives here, for order"):** source of truth for (1) DbUp migrations per module, including `CREATE EXTENSION IF NOT EXISTS pg_trgm` in Search's first migration; (2) dev/demo seed SQL; (3) the Postgres/Valkey compose file and container init scripts; (4) schema docs / ER diagram per module.
- Layout: `migrations/<module>/NNNN_name.sql` · `seeds/` · `compose/` · `docs/` · `README.md`.
- **Consumption: sync script.** `vibra-backend/scripts/sync-db.sh` copies each module's `.sql` into that module's Infrastructure as embedded resources (same pattern as `sync-contract.sh`). DbUp still runs in-process, so there is one migration path for compose and Testcontainers. One migration set per module; never mix modules in a migration.
- **Drift with `vibra-backend/CLAUDE.md`** (migrations "embedded per module", compose in the backend): ask which wins and update it from the backend repo.
- Details: `vibra-backend/CLAUDE.md` (Data access).

## Open items
- **To verify:** DbUp compatibility with .NET 11 (spike in Sprint 1).
- **To verify:** `pg_trgm` can be created by the DB owner on the chosen Postgres image (trusted extension since PG13).

## Corrections
Original Spanish draft said EF Core 11 / `DbContext`; replaced by Dapper + Npgsql.
