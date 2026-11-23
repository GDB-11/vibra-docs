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
- Dapper has no built-in migrations. **Migration tool: DbUp** — plain `.sql` files embedded per module, run in order, tracked in a journal table (one journal per module). Runs in-process, so compose (Podman) and Testcontainers tests use the same path.
- Alternatives considered: dbmate/Flyway (external CLI, awkward per-module sets), FluentMigrator/Evolve (more ceremony), hand-rolled scripts (would reimplement DbUp's tracking).
- DbUp is a new NuGet package: **ask before adding** (this decision records the choice; the approval to add it is given at that time).
- `vibra-db` is left with only cross-cutting SQL (e.g. `pg_trgm` extension, seed data) or nothing; exact scope is a Sprint 1 task.
- Details: `vibra-backend/CLAUDE.md` (Data access).

## Open items
- **To verify:** DbUp compatibility with .NET 11 (spike in Sprint 1).

## Corrections
Original Spanish draft said EF Core 11 / `DbContext`; replaced by Dapper + Npgsql.
