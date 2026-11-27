# 018 — Database schema

**Status:** Decided (2026-11-26)

## Context
Initial table structure per module, following [017](017-database-conventions.md). Object storage keys referenced here are defined in [003](003-audio-storage-cdn.md).

## Options considered

| Topic | Chosen | Not chosen |
|---|---|---|
| Search data | Indexes on Catalog tables; Catalog serves search; Search module removed | A `search."SearchDocument"` copy filled by Catalog events (keeps the module, must be kept in sync) |
| Purchased item | Nullable `TrackId` and `AlbumId` | Polymorphic `item_id` (cannot have an FK) |

## Choice
Every table, join tables included, has `<Table>Id`, `CreatedAt` and `UpdatedAt` (not repeated below). ★ = also has `<Table>Code`. A `…Id` column that names a table is an FK; `StoreProductId` and `StoreTransactionId` are StoreKit strings.

### `identity`

| Table | Main columns and rules |
|---|---|
| `"User"` ★ | `Email`, `NormalizedEmail` (unique), `PasswordHash`, `DisplayName`, `SecurityStamp`, `ConcurrencyStamp`, `LockoutEnd`, `AccessFailedCount`. `User` is a reserved word; fine when quoted |
| `"RefreshToken"` | `UserId`, `TokenHash` (unique), `TokenFamily` (uuid), `ExpiresAt`, `RevokedAt`, `ReplacedByRefreshTokenId` |

No role tables. ASP.NET Core Identity ([007](007-auth-identity-jwt-refresh.md)) needs custom Dapper stores, since there is no EF Core.

### `catalog`

| Table | Main columns and rules |
|---|---|
| `"Artist"` ★ | `Name` |
| `"Album"` ★ | `ArtistId` (primary artist; "Various Artists" for compilations), `Title`, `ReleaseDate`, `CoverKey` (uuid, nullable), `PriceCents`, `Currency`, `StoreProductId` (unique, nullable) |
| `"Track"` ★ | `AlbumId`, `Title`, `DiscNumber`, `TrackNumber` (unique per album and disc), `DurationMs`, `Status` (`Pending`, `Ready`, `Failed`), `PriceCents`, `Currency`, `StoreProductId` |
| `"TrackArtist"` | `TrackId`, `ArtistId`, `Position` (credits and features) |

### `streaming`

| Table | Main columns and rules |
|---|---|
| `"AudioFile"` | `TrackId`, `Kind` (`Full`, `Preview`), `FileKey` (uuid, unique), `SizeBytes`, `DurationMs`, `SampleRateHz`, `BitDepth` (nullable), `Channels`; unique (`TrackId`, `Kind`) |
| `"IngestJob"` ★ (Sprint 3) | `TrackId`, `UploadKey` (uuid), `Status`, `ErrorMessage` |

### `library`

| Table | Main columns and rules |
|---|---|
| `"Playlist"` ★ | `UserId`, `Name` |
| `"PlaylistTrack"` | `PlaylistId`, `TrackId`, `Position`; the same track may repeat |
| `"TrackLike"` | `UserId`, `TrackId`, unique together (`Like` is reserved) |
| `"ArtistFollow"` | `UserId`, `ArtistId`, unique together |

### `history`

| Table | Main columns and rules |
|---|---|
| `"PlayEvent"` | `UserId`, `TrackId`, `PlayedAt`, `PlayedMs`; recently played and play counts are queries |

### `billing`

| Table | Main columns and rules |
|---|---|
| `"Plan"` ★ | `Interval` (`Monthly`, `Annual`; unique), `PriceCents`, `Currency`, `StoreProductId` (maps StoreKit transactions to a plan) |
| `"Subscription"` ★ | `UserId`, `PlanId`, `Status` (`Active`, `Canceled`, `PastDue`, `Expired`), `CurrentPeriodStart`, `CurrentPeriodEnd`, `CancelAtPeriodEnd`; one `Active`/`PastDue` per user (partial unique index) |
| `"Payment"` ★ | `UserId`, `AmountCents`, `Currency`, `Status` (`Succeeded`, `Failed`, `Refunded`), `Kind` (`Subscription`, `Purchase`), `IdempotencyKey` (unique), `StoreTransactionId` (nullable) |
| `"Purchase"` ★ | `UserId`, `ItemType` (`Track`, `Album`), `TrackId`, `AlbumId`, `PriceCents`, `Currency`, `PaymentId`, `PurchasedAt` |

- `Purchase`: a `CHECK` requires exactly the one of `TrackId` / `AlbumId` that matches `ItemType`. Partial unique indexes on (`UserId`, `TrackId`) and (`UserId`, `AlbumId`) block double purchase ([009](009-entitlements-backend.md)).

### Column and constraint details (approved 2026-11-26)

| Topic | Rule |
|---|---|
| Nullable columns | Only `User."LockoutEnd"`, `RefreshToken."RevokedAt"` and `"ReplacedByRefreshTokenId"`, `Album."CoverKey"`, `"StoreProductId"` on `Album` and `Track`, `AudioFile."BitDepth"`, `Payment."StoreTransactionId"`, `Purchase."TrackId"` and `"AlbumId"`. All others are `NOT NULL`, including `IngestJob."ErrorMessage"` unless decided otherwise with that table |
| `StoreProductId` | Unique on `Album`, `Track` and `Plan`; required on `Plan` |
| Cascade | `RefreshToken` → `User`, `PlaylistTrack` → `Playlist`, `TrackArtist` → `Track`. All other FKs are `RESTRICT`, except the next row |
| Token self-reference | `ReplacedByRefreshTokenId` uses `NO ACTION` (exception to [017](017-database-conventions.md)): it is checked at the end of the statement, so it does not block cascading a user's tokens |
| Extra uniques | `TrackArtist` (`TrackId`, `ArtistId`). `PlaylistTrack."Position"` is not unique, to keep reordering simple |
| Extra checks | Prices, amounts, sizes, `DurationMs` and `PlayedMs` not negative; `DiscNumber` and `TrackNumber` positive |
| Lockout | No `LockoutEnabled` column; the Dapper Identity store can treat lockout as always enabled |

### Search and genres
- **There is no Search module** (removed, [001](001-modular-monolith.md)). Catalog's first migration runs `CREATE EXTENSION IF NOT EXISTS pg_trgm`; trigram (and full-text) indexes go on `Artist."Name"`, `Album."Title"` and `Track."Title"`; Catalog serves search. Both are GIN indexes; full-text uses `to_tsvector('simple', …)` (no language stemming), and queries must use the same expression.
- **Genres:** omitted. The original draft listed them under Catalog; add them if a screen needs them.

## Consequences
- Changes the data model of [009](009-entitlements-backend.md): `Purchase.item_id` split in two, `Plan.code` dropped.
- Seeds use fixed `<Table>Code`, `FileKey` and `CoverKey` values, so files uploaded by hand still match after a database reset.
- Sprint 1: the DbUp and Catalog migration tasks follow 017 and this decision (the sprint file is unchanged).
- Initial migrations are in `vibra-db` (2026-11-26), one per module. `IngestJob` is not included (Sprint 3). They were applied with `psql` on PostgreSQL 18, not yet through DbUp.

## Open items
- **To verify:** whether local StoreKit transaction IDs repeat after clearing transactions in Xcode. Until then, no unique constraint on `StoreTransactionId`.
- `IngestJob."Status"` values are not defined yet (Sprint 3).
- **To verify:** the migrations run through DbUp, including the `$$` function bodies (DbUp substitutes `$name$` variables).
