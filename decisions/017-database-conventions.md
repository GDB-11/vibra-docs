# 017 — Database conventions

**Status:** Decided (2026-11-26)

## Context
No decision covered database conventions. Nothing had been migrated yet. These rules replace the earlier "snake_case in Postgres" consequence of [002](002-postgres-dapper-npgsql.md). Table structure: [018](018-database-schema.md).

## Options considered

| Topic | Chosen | Not chosen |
|---|---|---|
| Table and column names | Singular, PascalCase | snake_case (no quoting needed) |
| Cross-module references | Internal ID with a real FK | Internal ID without FK (independent module migrations); public UUID without FK (cleanest boundary, 16-byte keys) |
| `CreatedAt` | Database default | Set by the app via `TimeProvider` |

## Choice

| Item | Rule |
|---|---|
| Schemas | One per module, lowercase (`identity`, `catalog`, ...) |
| Tables, columns | Singular, PascalCase, **always double-quoted** in SQL: `catalog."Track"` (unquoted names fold to lowercase) |
| PK | `"<Table>Id" bigint GENERATED ALWAYS AS IDENTITY` (`"TrackId"`), never in the API, URLs or JWT |
| Public code | `"<Table>Code" uuid NOT NULL DEFAULT gen_random_uuid()` (`"TrackCode"`, UUIDv4), unique; only on rows the API exposes |
| FK columns | Same name as the referenced PK (`Track."AlbumId"` → `Album."AlbumId"`) |
| Indexes and constraints | snake_case, always named: `pk_track`, `fk_track_album_id`, `ux_track_track_code`, `ix_track_album_id`, `ck_track_status` |
| FKs | Indexed; `ON DELETE RESTRICT` by default, `CASCADE` only for owned children (playlist entries, tokens) |
| Types | `text`; money as `integer` cents + `char(3)`; enums as `text` + `CHECK` (pass strings from Dapper) |
| Audit columns | Every table: `"CreatedAt"` and `"UpdatedAt"`, both `timestamptz NOT NULL DEFAULT now()`. A `BEFORE UPDATE` trigger sets `UpdatedAt`; each module creates the trigger function in its own schema |
| UTC | **Always UTC:** only `timestamptz` for instants (never `timestamp`); database and connection time zone set to `UTC`; C# values have zero offset; the API uses ISO 8601 `Z`. Calendar dates (`ReleaseDate`) are `date` |
| Business time | `ExpiresAt`, `PurchasedAt`, `PlayedAt`, billing periods: no default, set by the app via `TimeProvider` (UTC). Queries compare them to a `@Now` parameter, never SQL `now()` |
| Exception | DbUp journals (`<module>_schemaversions`) keep that naming |

- `CreatedAt` and `UpdatedAt` are audit only: they use the real clock and ignore demo time travel. **Never use them in business rules.**
- **Cross-module FKs** are allowed in DDL. Migrations run in a fixed order, in the host and in Testcontainers: Identity → Catalog → Streaming → Library → History → Billing.
- The module boundary still holds in code: no module queries another module's tables; it resolves IDs through public interfaces. The FKs are the only database coupling.
- **JWT `sub` = `"UserCode"`.** The existing `FunctionalController` already passes a `Guid`; services resolve it to `UserId` through Identity.

## Consequences
- Hand-written SQL must quote every identifier; C# raw string literals keep it readable.
- Column names match row-model property names, so `DefaultTypeMap.MatchNamesWithUnderscores` is no longer needed.
- Module migrations are no longer independent: the order above is mandatory.
- **Drift with `vibra-backend`:** its `CLAUDE.md` says snake_case and has no UTC or audit-column rules; `BaseDatabaseService` sets `MatchNamesWithUnderscores`. Update from the backend repo.

## Open items
- None.
