# 003 — Audio in private object storage + CDN

**Status:** Decided, including provider (**Cloudflare R2 + its CDN**, free tier, 2026-11-23). Backend hosting: **Hetzner VPS**.

## Context
Audio files are large; the API should not carry them.

## Options considered
- **Cloudflare R2 + CDN (chosen):** S3-compatible, so the ingest flow below stays as written; free egress and a free storage tier.
- Hetzner Object Storage: S3 model, one provider, but no CDN.
- Files on the VPS disk behind nginx/Caddy: cheapest, but would need polling instead of storage events and `secure_link` signing.
- Azure Blob / S3: not chosen.
- Audio in the DB: rejected by principle.

## Choice
Audio in private object storage with short-lived signed URLs, delivered via CDN. Never in the DB, **never through the API**.

| Data | Where | Notes |
|---|---|---|
| Metadata, users, playlists, billing | PostgreSQL | Source of truth |
| Audio | Private object storage | Short-lived signed URLs |
| Cover art | Same storage, via CDN | 2–3 sizes generated at ingest |
| Cache/sessions | Valkey | Optional ([012](012-valkey-cache.md)) |

**Ingest flow:** pre-signed upload URL → upload `.flac` → storage event → worker (FFmpeg/ffprobe) validates, extracts metadata (Vorbis tags, duration) and embedded cover art, generates the 30 s AAC preview → track marked `Ready`.

## Consequences
- Signed URL lifetimes short; never log full signed URLs.
- Hosting: a cheap **Hetzner VPS** (about US$7/month, user's estimate) runs the API, worker, Postgres and Valkey as Podman containers ([016](016-podman-containers.md)). R2 holds audio and cover art.

## Open items (all **to verify**, Sprint 1 spike with #4/#8)
- Current R2 free-tier limits and pricing.
- Cloudflare terms for serving audio through its CDN on the free plan.
- Short-lived signed URLs: R2 presigned URLs work on the S3 endpoint, not on a CDN custom domain. Confirm how signed access is combined with CDN caching (e.g. Worker or token validation) and that `AVPlayer` range/seek works through it.
