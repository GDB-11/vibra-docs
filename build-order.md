# Build order

1. **Walking skeleton:** login, list tracks from the API and play a fixed track with background audio and lock screen controls. *(The player is the riskiest piece: do it first.)*
2. **Catalog:** artists, albums, search, cover art from CDN, with prices.
3. **Library:** playlists, likes, history.
4. **Billing:** plans, track/album purchase, subscription, entitlements endpoint and 403 with preview.
5. **Paywall and Store in iOS** (StoreKit 2), with 30-second accelerated renewal for the demo.
6. **Ingest** (FLAC validation, metadata, AAC previews), offline downloads, widget + Live Activity, and extras (HLS, Normal/Lossless quality, SignalR, App Intents).

## Sprint plan (decided 2026-11-23)

Total time: 24 days starting 2026-11-23 (day 1), ending 2026-12-16. **3 sprints of 8 days.** Numbering starts at 1.

| Sprint | Days | Focus |
|---|---|---|
| 1 | Nov 23–30 | Repo skeletons, DbUp migrations, login, background FLAC playback, spikes (`AVPlayer` + R2 signed URL, OpenAPI version, DbUp on .NET 11), catalog |
| 2 | Dec 1–8 | Billing and entitlements, StoreKit 2 paywall, 403 + preview |
| 3 | Dec 9–16 | Ingest, widget + Live Activity, polish, demo prep and buffer |

**Cut candidates if time runs short:** Library (playlists, likes, history) and offline downloads. Which to drop is not decided.

Sprint exports are only generated when explicitly requested (see `CLAUDE.md`). Estimates (points, hours) and assignees are given only on request. Sprint 1 has them: 74 points, 137 hours, assignee Gianfranco Diaz Badoino.
