# Open questions and items to verify

The initial planning decisions were closed on 2026-11-23. What remains is verification, mainly as **Sprint 1 spike tasks**, plus the items under "Still to decide".

## To verify (Sprint 1 spikes)

| Item | Related |
|---|---|
| OpenAPI 3.1 chosen (3.2 is preliminary in the generator); confirm the generator processes the real .NET 11 export pinned to 3.1 | [005](decisions/005-rest-openapi-swift-client.md) |
| `AVPlayer` plays and seeks a remote FLAC through R2/CDN with signed access and range requests | [003](decisions/003-audio-storage-cdn.md), [004](decisions/004-audio-format-flac.md) |
| Signed URLs vs CDN custom domain on R2 (presigned URLs use the S3 endpoint) | [003](decisions/003-audio-storage-cdn.md) |
| Current R2 free-tier limits and Cloudflare terms for audio on the free plan | [003](decisions/003-audio-storage-cdn.md) |
| Rip format of the purchased CD (FLAC or WAV) | [004](decisions/004-audio-format-flac.md) |
| Exact version number of "latest iOS" against installed Xcode | [008](decisions/008-ios-mvvm-local-packages.md) |
| CDN domain setup for `vibra-images`; cover widths (proposal: 300, 600, 1200 px) | [003](decisions/003-audio-storage-cdn.md) |
| Whether local StoreKit transaction IDs repeat after clearing transactions in Xcode | [018](decisions/018-database-schema.md) |

## Still to decide

| Item | Related |
|---|---|
| Which features to cut if time runs short (Library, offline downloads) | [build-order.md](build-order.md) |
| Apply the Podman / DbUp / R2 wording changes to `vibra-backend/CLAUDE.md` (from the backend repo; draft given 2026-11-23) | [016](decisions/016-podman-containers.md) |
| Update `vibra-backend/CLAUDE.md` for migrations/compose living in `vibra-db` and `scripts/sync-db.sh` (from the backend repo) | [002](decisions/002-postgres-dapper-npgsql.md) |
| Keep or remove the Search module, now nearly empty (also touches the backend skeleton) | [001](decisions/001-modular-monolith.md), [018](decisions/018-database-schema.md) |
| Update `vibra-backend` (from that repo): `CLAUDE.md` naming, UTC and audit columns, `Payment.idempotency_key`, `UNIQUE(user_id, item_type, item_id)`, Search ownership, migration order; remove `MatchNamesWithUnderscores` from `BaseDatabaseService` | [017](decisions/017-database-conventions.md), [018](decisions/018-database-schema.md) |
| Set the database time zone to UTC in `vibra-db/compose` (from that repo) | [017](decisions/017-database-conventions.md) |

## Closed (2026-11-23)
DbUp compatible with .NET 11 (spike, 2026-11-23) · Sprint length (3 × 8 days) · 3 native tools confirmed · DbUp · Hetzner VPS + Cloudflare R2/CDN · lossless for everyone · Simulator only · latest iOS · skeletons in Sprint 1 · catalog from owned CD.
