# Overview

**Vibra** is a music streaming app with a native iOS client (SwiftUI) and an ASP.NET Core 11 backend. Two ways to access music:

- **Subscription** (monthly or annual): access to the whole catalog.
- **Purchase** of a track or album: access in perpetuity.

Payments are **simulated**. The music for the course was acquired legally and is for personal use.

**Stack:** SwiftUI · Swift 6 · StoreKit 2 · AVFoundation · WidgetKit/ActivityKit · ASP.NET Core 11 · PostgreSQL via Dapper + Npgsql · BindSharp (Result pattern) · Valkey (optional) · FFmpeg · OpenAPI.

## Architecture

```
                    ┌─────────────────────────────┐
                    │        iOS app (SwiftUI)    │
                    └──┬──────────┬───────────┬───┘
         REST/JSON     │          │ HTTP range│ SignalR (optional)
         (metadata)    │          │ (HLS opt.)│
                       ▼          ▼           ▼
  ┌──────────────────────────┐  ┌────────────────┐
  │ ASP.NET Core 11 API      │  │ CDN + Object   │
  │ modular monolith         │  │ storage (audio,│
  │ Identity│Catalog│Library │  │ cover art)     │
  │ Streaming│History│Billing│  └───────▲────────┘
  │                          │          │ writes
  └───┬────────┬─────────┬───┘     ┌────┴─────────┐
  PostgreSQL  Valkey   Job queue─▶ │ Transcoding  │
  (metadata)  (cache)              │ worker       │
                                   │ (FFmpeg)     │
                                   └──────────────┘
```

**Key principle:** audio bytes never pass through the API. The API hands out short-lived signed URLs and the app streams directly from storage/CDN.

## Backend notes not covered by a decision

- **.NET 11** was released on 10 November 2026. STS release (two years of support); .NET 10 remains LTS.
- Local dev: .NET Aspire or compose on **Podman** (API, Postgres, Valkey) ([016](decisions/016-podman-containers.md)). No blob emulator needed to decide yet; R2 is S3-compatible.
- Observability: OpenTelemetry. Deployment: Podman containers on a Hetzner VPS (API, worker, Postgres, Valkey); audio and cover art in Cloudflare R2 + CDN ([003](decisions/003-audio-storage-cdn.md)).
- Background jobs: `BackgroundService` + `Channels`, or Hangfire if a dashboard is wanted.
- Testing: xUnit + Testcontainers (real Postgres, never in-memory); Testcontainers on Podman is **to verify**.
- CI/CD: GitHub Actions (backend: build, test, publish container); Xcode Cloud or Actions for iOS.

Backend code conventions (time via `TimeProvider`, SQL rules, API conventions, NuGet policy) are owned by `vibra-backend/CLAUDE.md`; they are not duplicated here.

## Sources

- .NET 11 announcement: https://devblogs.microsoft.com/dotnet/?p=61205
- InfoQ, .NET 11 release: https://infoq.com/news/2026/11/dotnet-11-release
- See [decision 010](decisions/010-simulated-payments-storekit2.md) for the StoreKit / Apple account sources.
