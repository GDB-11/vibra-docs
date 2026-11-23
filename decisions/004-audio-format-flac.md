# 004 — Audio format: FLAC

**Status:** Decided (FLAC for everyone, 30 s AAC previews). Lossless tiers and "Normal" AAC: **Rejected** (2026-11-23). HLS remains an **Idea**.

## Context
Small demo catalog; `AVPlayer` plays FLAC natively on iOS, including background playback and lock screen controls.

## Options considered
- Lossless FLAC served directly (chosen)
- AAC "Normal" quality (~256 kbps) — rejected: no quality distinction
- Lossless as subscriber-only tier — rejected: there are only subscribers and purchasers, both get lossless
- HLS (Apple supports FLAC in HLS with fMP4) — *Idea*

## Choice
**FLAC** as master and only full-track format, for every user who is entitled (subscriber or purchaser), served with HTTP range + signed URL. **30 s AAC previews** for the `403` response carrying `previewUrl`. No quality tiers.

## Consequences
- Size: roughly 20–40 MB per ~4 min song; ~15 songs = hundreds of MB. Fine for a demo; mind mobile data and offline downloads.
- Transcoding an MP3 to FLAC improves nothing: need truly lossless sources (FLAC, or WAV converted with FFmpeg) with a free license.
- For a small catalog, progressive download with range requests is enough; HLS is optional (verify with a test file at the start).

## Ideas (not decided)
- HLS.

## Catalog source
A purchased CD owned by the user, ripped to lossless. **To verify:** the rip format (FLAC or WAV is fine; WAV converts with FFmpeg). No optional 256 kbps AAC rendition is generated.

## Open items
- Spike (Sprint 1): `AVPlayer` plays and seeks a remote FLAC with a signed URL and range requests. See [open-questions.md](../open-questions.md).

## Corrections
HLS removed from the "decided" streaming description (it is optional).
