# Open questions and items to verify

All planning decisions are closed (2026-11-23). What remains is verification, to be done as **Sprint 1 spike tasks**, plus two small decisions.

## To verify (Sprint 1 spikes)

| Item | Related |
|---|---|
| OpenAPI 3.1 chosen (3.2 is preliminary in the generator); confirm the generator processes the real .NET 11 export pinned to 3.1 | [005](decisions/005-rest-openapi-swift-client.md) |
| `AVPlayer` plays and seeks a remote FLAC through R2/CDN with signed access and range requests | [003](decisions/003-audio-storage-cdn.md), [004](decisions/004-audio-format-flac.md) |
| Signed URLs vs CDN custom domain on R2 (presigned URLs use the S3 endpoint) | [003](decisions/003-audio-storage-cdn.md) |
| Current R2 free-tier limits and Cloudflare terms for audio on the free plan | [003](decisions/003-audio-storage-cdn.md) |
| DbUp compatibility with .NET 11 | [002](decisions/002-postgres-dapper-npgsql.md) |
| Rip format of the purchased CD (FLAC or WAV) | [004](decisions/004-audio-format-flac.md) |
| Exact version number of "latest iOS" against installed Xcode | [008](decisions/008-ios-mvvm-local-packages.md) |

## Still to decide

| Item | Related |
|---|---|
| Which features to cut if time runs short (Library, offline downloads) | [build-order.md](build-order.md) |
| Apply the Podman / DbUp / R2 wording changes to `vibra-backend/CLAUDE.md` (from the backend repo; draft given 2026-11-23) | [016](decisions/016-podman-containers.md) |
| Scope of `vibra-db` (cross-cutting SQL only, or nothing); a Sprint 1 task | [002](decisions/002-postgres-dapper-npgsql.md) |

## Closed (2026-11-23)
Sprint length (3 × 8 days) · 3 native tools confirmed · DbUp · Hetzner VPS + Cloudflare R2/CDN · lossless for everyone · Simulator only · latest iOS · skeletons in Sprint 1 · catalog from owned CD.
