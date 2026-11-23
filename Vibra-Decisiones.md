# Vibra — Architecture Decisions and Project Summary

> iOS music streaming app + .NET 11 backend. Demo project for an iOS course.
> *Todo el catálogo, o solo lo que amas.*

Document date: 2026-11-22
(English translation of the original Spanish draft. Stale items were corrected to match `vibra-backend/CLAUDE.md`; see section 14.)

---

## 1. Project summary

**Vibra** is a music streaming app with a native iOS client (SwiftUI) and an ASP.NET Core 11 backend. It offers two ways to access music:

- **Subscription** (monthly or annual): access to the whole catalog.
- **Purchase** of a track or album: access in perpetuity.

Payments are **simulated** (demo project). The music for the course was acquired legally and is for personal use.

**Stack:** SwiftUI · Swift 6 · StoreKit 2 · AVFoundation · WidgetKit/ActivityKit · ASP.NET Core 11 · PostgreSQL via Dapper + Npgsql · BindSharp (Result pattern) · Valkey (optional) · FFmpeg · OpenAPI.

---

## 2. Decisions made (quick summary)

| # | Decision | Choice |
|---|---|---|
| 1 | Backend style | Modular monolith (not microservices) |
| 2 | Database | PostgreSQL accessed with **Dapper + Npgsql** (raw parameterized SQL, **no EF Core**) |
| 3 | Audio storage | Object storage (Azure Blob / S3 / R2) + CDN; never in the DB |
| 4 | Audio format and delivery | **FLAC (lossless)** as master and main format, served with HTTP range + signed URL. AAC only for 30 s previews and as an optional "Normal" quality. HLS is an extra |
| 5 | API contract | REST/JSON + OpenAPI; Swift client generated with `swift-openapi-generator` |
| 6 | Real time | Optional SignalR (sync "now playing") |
| 7 | Auth | ASP.NET Core Identity + short-lived JWT + rotating refresh tokens. Email/password in the demo (Sign in with Apple dropped: needs a paid account) |
| 8 | iOS architecture | MVVM + repositories, modules as local Swift packages, `@Observable`, Swift 6 strict concurrency |
| 9 | Monetization | Entitlements in the backend; simulated payment with **StoreKit 2 + local `.storekit` file** |
| 10 | Repositories | Multi-repo: `/home/Gianfranco/repos/vibra-ios`, `vibra-backend`, `vibra-contracts`, `vibra-db`, and `vibra-docs` (decision documentation) |
| 11 | Name | **Vibra** |
| 12 | Cache | **Valkey** (Redis-protocol compatible), optional |
| 13 | HTTP layer | MVC controllers (thin, via a shared `FunctionalController`); **Minimal APIs are not used** |
| 14 | Error handling | BindSharp `Result<T, TError>` everywhere; no exceptions for expected failures; RFC 7807 Problem Details at the HTTP edge |

---

## 3. General architecture

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
  │ Streaming│History│Search │  └───────▲────────┘
  │ Billing                  │          │ writes
  └───┬────────┬─────────┬───┘     ┌────┴─────────┐
  PostgreSQL  Valkey   Job queue─▶ │ Transcoding  │
  (metadata)  (cache)              │ worker       │
                                   │ (FFmpeg)     │
                                   └──────────────┘
```

**Key principle:** audio bytes never pass through the API. The API hands out short-lived signed URLs and the app streams directly from storage/CDN.

---

## 4. Backend (.NET 11)

**Version:** .NET 11 was released on 10 November 2026. It is an STS (Standard Term Support) release with two years of support; .NET 10 remains the LTS.

**Important note:** ASP.NET Core 11 generates **OpenAPI 3.2 documents by default** (breaking change). Verify which spec version `swift-openapi-generator` supports and configure the .NET document to emit an older version if needed.

### Modules

| Module | Responsibility |
|---|---|
| Identity | Registration, login, refresh tokens |
| Catalog | Artists, albums, tracks, genres, prices, `store_product_id` |
| Library | Playlists, likes, followed artists |
| Streaming | Signed URLs, preview URLs, ingest/upload |
| History | Recently played and play counts |
| Search | Search (Postgres full-text + `pg_trgm`) |
| **Billing** | Plans, subscriptions, purchases, payments, entitlements |

### Structure per module
`Domain` · `Application` (use cases, vertical slices) · `Infrastructure` (Dapper repositories + SQL, storage) · `Api` (controllers, error mapper).
Modules communicate through public interfaces or in-process events, never through another module's repositories, row models or SQL. Enforce with architecture tests (NetArchTest).

### Code conventions
- **Result pattern (BindSharp)** is mandatory in Infrastructure, Application services and controllers. Expected failures (not found, conflict, validation, payment declined) are never thrown.
- **Controllers only, thin:** each action is one expression delegating to a service via the shared `FunctionalController`. One `IErrorHttpMapper<TError>` per controller converts module errors to Problem Details. Never expose exception messages, SQL or stack traces.
- **Data access:** SQL constants per repository, always parameterized; snake_case in Postgres, PascalCase row models; unique/foreign-key violations (SQLSTATE `23505`/`23503`) become specific errors.
- **Time:** inject `TimeProvider`; never `DateTime.UtcNow` (the demo needs time travel for renewals).
- **Migrations:** one set per module; migration tool **not chosen yet** (Dapper has no built-in migrations).
- **NuGet:** pre-approved BindSharp, Dapper, Npgsql; ask before adding others.

### API conventions
- `/v1` versioning.
- Cursor pagination.
- Errors with Problem Details (RFC 7807).
- ETags on catalog reads.
- `Idempotency-Key` required on payment operations.
- Stable, descriptive camelCase `operationId`s (they become Swift method names).

### Other
- **Cache:** optional Valkey (hot catalog, sessions). Use a Redis-compatible client such as StackExchange.Redis; ask before adding.
- **Background jobs:** `BackgroundService` + `Channels`, or Hangfire if a dashboard is wanted.
- **Local dev:** .NET Aspire or docker-compose (API, Postgres, Valkey, blob emulator).
- **Observability:** OpenTelemetry (traces and metrics).
- **Deployment:** Docker container (Azure Container Apps, Fly.io or similar) with managed Postgres and blob storage.
- **Testing:** xUnit + Testcontainers (real Postgres, never an in-memory fake).

---

## 5. Storage

| Data | Where | Notes |
|---|---|---|
| Metadata, users, playlists, billing | PostgreSQL | Source of truth |
| Audio | Private object storage | Short-lived signed URLs |
| Cover art | Same storage, via CDN | 2–3 sizes generated at ingest |
| Cache/sessions | Valkey | Optional |

**Ingest flow:** pre-signed upload URL → upload of the `.flac` → storage event → worker (FFmpeg/ffprobe) validates the file, extracts metadata (Vorbis tags, duration) and embedded cover art, generates the 30 s AAC preview (and optionally a 256 kbps AAC rendition) and marks the track `Ready`.

### Audio format: FLAC

- **Decision:** the demo catalog is small, so **lossless FLAC** is served directly. `AVPlayer` plays FLAC natively on iOS, including background playback and lock screen controls.
- **Size:** a FLAC is roughly 20–40 MB per ~4 min song. With ~15 songs that is hundreds of MB of storage: fine for a demo, but keep it in mind for bandwidth (mobile data) and offline downloads.
- **Previews (30 s):** generated as AAC (lighter) for the `403` response carrying `previewUrl`.
- **"Normal" quality (optional):** ~256 kbps AAC rendition for mobile data or data saving. "Lossless" could be offered only to subscribers as a subscription differentiator; **this is an idea, not a decision**.
- **Lossless sources:** transcoding an MP3 to FLAC improves nothing. Obtain truly lossless sources with a free license, or use original tracks, and confirm the provider offers FLAC or WAV (WAV converts to FLAC with FFmpeg).
- **HLS:** optional. Apple supports FLAC inside HLS with fMP4, but for a small catalog progressive download with range requests is enough (verify with a test file at the start).

---

## 6. Communication

- **REST/JSON over HTTPS** for all request/response traffic.
- **OpenAPI** as the contract; generated Swift client (`openapi.json` produced by the backend CI and committed to `vibra-contracts`).
- **Audio:** range requests (or HLS) directly from CDN/storage.
- **SignalR (stretch):** "now playing" sync across devices. In .NET 11, SignalR can refresh authentication without reconnecting.
- **gRPC/GraphQL:** rejected (more friction on iOS with no clear benefit).
- **Push (APNs):** rejected in the demo because it requires a paid account; use local notifications if needed.

---

## 7. iOS

**Stack:** SwiftUI, Swift 6 (strict concurrency), async/await, Observation (`@Observable`). Target a minimum iOS version one below the current one (verify against your Xcode and devices).

### Layers
`View → ViewModel (@Observable, @MainActor) → Repository (protocol in Domain) → Networking + Persistence`.
Features depend on `Domain` and `DesignSystem`, never on each other. Navigation goes through a coordinator/router in the app target.

### Organization (repo `vibra-ios`)

```
Vibra/                       ← thin target: composition root, DI, navigation
Packages/
  Domain/                    ← models, repository protocols (no dependencies)
  Networking/                ← generated client, auth interceptor, token refresh
  Persistence/               ← SwiftData (or GRDB): cache and downloads
  Playback/                  ← PlaybackService, queue, Now Playing, remote commands
  Store/                     ← StoreKitService, entitlements
  DesignSystem/
  Features/
    Auth/  Home/  Search/  Library/  Player/  Store/  Subscription/  Settings/
```

### Technical points
- **Playback:** a global `PlaybackService` (wrapping `AVPlayer`/`AVQueuePlayer`), independent of views. `AVAudioSession` with `.playback` category and the audio background mode. Integration with `MPNowPlayingInfoCenter` and `MPRemoteCommandCenter`. Handle interruptions and route changes.
- **Offline:** `URLSession` with a background configuration (`AVAssetDownloadURLSession` if HLS is used). Store in SwiftData the entitlement each download came from (subscription downloads lock on expiry; purchased ones are kept).
- **Security:** tokens only in the Keychain; token refresh inside a single actor so concurrent 401s trigger one refresh.
- **Testing:** Swift Testing for ViewModels and repositories; a few XCUITests (login → play).
- **Localization:** user-facing strings go through `String(localized:)`; UI is in Spanish.

---

## 8. Monetization (simulated)

### Model
- **Subscription** monthly/annual → access to the whole catalog while active.
- **Purchase** of a track or album → perpetual access.

### Access rule (`IEntitlementService.CanPlay(userId, trackId)`)
A user can play the full track if **any** of these holds:
1. Active subscription (`status = Active` and `now < current_period_end`).
2. Purchase of the track.
3. Purchase of the album containing the track.

If none holds: **403 + 30 s `previewUrl`** (freemium experience and a useful paywall).

### Decided edge cases
- Expired subscription: catalog access is lost; **purchases are always kept**.
- Simulated refund: `Payment = Refunded` and the `Purchase` is removed (the only case where a purchase is deleted).
- Album that gains tracks later: a purchased album includes everything it has, today and tomorrow.
- Buying an album while owning single tracks: accepted with no discount, showing "You already have X of N" (prorated discount = optional extra).
- Double purchase: blocked by `UNIQUE(user_id, item_type, item_id)`.

### Data model (Billing module)

```
Plan: id, code (monthly|annual), price_cents, currency, interval
Subscription: id, user_id, plan_id, status (Active|Canceled|PastDue|Expired),
              current_period_start/end, cancel_at_period_end
Purchase: id, user_id, item_type (Track|Album), item_id, price_cents,
          payment_id, purchased_at   UNIQUE(user_id, item_type, item_id)
Payment: id, user_id, amount_cents, status (Succeeded|Failed|Refunded),
         idempotency_key (UNIQUE), kind (Subscription|Purchase), created_at
```
In Catalog, `Track` and `Album` add `price_cents`, `currency` and `store_product_id`.

Money is always integer cents + ISO 4217 currency. Timestamps are ISO 8601 UTC.

### Endpoints

```
GET    /v1/plans
POST   /v1/subscriptions                  { planCode, paymentMethod }   (Idempotency-Key)
DELETE /v1/subscriptions/current          → cancels at period end
POST   /v1/purchases                      { itemType, itemId, paymentMethod }   (Idempotency-Key)
GET    /v1/me/entitlements
GET    /v1/me/purchases
GET    /v1/tracks/{id}/stream             → 200 signed URL | 403 + previewUrl
POST   /v1/storekit/transactions          { productId, transactionId, expiresDate }   (demo mode)
```

### Simulated payment decision: StoreKit 2 + `.storekit` file
StoreKit 2 with local configuration was chosen (option B). The backend sits behind `IPaymentGateway`; the demo implementation is `StoreKitDemoGateway`.

**Flow:**
1. Define in the `.storekit` file an auto-renewable subscription (one *subscription group* with monthly and annual) and **non-consumable** products for purchases.
2. The app purchases with `Product.purchase()` and listens to `Transaction.updates`.
3. The app sends the transaction to `POST /v1/storekit/transactions`.
4. The backend, **in demo mode only (`Billing:DemoMode=true`, fail closed if missing), trusts the client** and creates the `Subscription` or `Purchase`.

In production, step 4 would verify the JWS with the App Store Server API. **Document this as a conscious limitation.**

**Limitation:** in StoreKit every purchase is a distinct product. Define in the `.storekit` only a few albums/tracks (for example 5 and 10) and map them via `store_product_id`.

**Demo trick:** in the `.storekit` file, use *Editor > Subscription Renewal Rate > Monthly Renewal Every 30 Seconds* to watch renewals and expirations live.

**If simulated in the backend instead (option A, not chosen):** `FakePaymentGateway` with test cards (`4242…` approves, `…0002` declines, `…9995` approves then fails on renewal), injectable `TimeProvider`, a renewal `BackgroundService`, and dev-only endpoints `/dev/clock/advance` and `/dev/subscriptions/{id}/fail-renewal`. Dev-only endpoints, if ever added, must be registered only in the Development environment.

### Is a paid Apple account needed?
- **Local StoreKit Testing in Xcode (Simulator): no.** It works without App Store servers and without configuring the app in App Store Connect.
- **Sandbox, TestFlight and App Store: yes** (Apple Developer Program, US$99/year; sandbox requires the paid apps agreement).
- **Real device with a free account:** a free account gives a "Personal Team" that cannot sign builds with paid capabilities (in-app purchase, Sign in with Apple, remote push, iCloud). It was not confirmed whether local StoreKit works on a real iPhone with a free account: **test it early** and do not add the "In-App Purchase" capability if Xcode complains.
- An app installed with a free account stops opening after 7 days: reinstall from Xcode the day before the demo.
- If it were ever published: Apple requires In-App Purchase for digital content (guideline 3.1.1).

---

## 9. The 3 native iOS tools (course requirement)

| # | Tool | What to implement | What to research and document |
|---|---|---|---|
| 1 | **StoreKit 2** | Monthly/annual subscription, album and track purchase, paywall, `Transaction.updates`, restore purchases, renewal/expiration states | Differences from StoreKit 1, product types, transaction verification, testing with `StoreKitTest`, guideline 3.1.1 |
| 2 | **AVFoundation + MediaPlayer** | Background `AVPlayer`, `AVAudioSession`, lock screen and Control Center with `MPNowPlayingInfoCenter`/`MPRemoteCommandCenter`, interruptions | Audio session lifecycle, HLS vs progressive download, audio routes (AirPods/Bluetooth), why the player lives outside the views |
| 3 | **WidgetKit + ActivityKit** | "Now playing" widget and Live Activity (lock screen and Dynamic Island) driven from the app | Widget refresh budget, app–extension data sharing, Live Activity limits, local vs push-updated |

**Reserve:** App Intents (Siri, Shortcuts, Spotlight) if time runs short or if the course counts WidgetKit and ActivityKit as one.

**Dropped to avoid depending on a paid account or special permissions:** push notifications (APNs), CarPlay (requires Apple authorization), Sign in with Apple.

**Verify in Xcode when creating the widget target:** App Groups and Background Modes with a free account (details could not be confirmed).

---

## 10. Repository organization (multi-repo)

Decision #10 is multi-repo. Each repo has its own `CLAUDE.md` with implementation rules; this repo records the *why*.

| Repo | Contents |
|---|---|
| `vibra-backend` | `src/Vibra.Api` (host), `src/Modules/{Identity,Catalog,Library,Streaming,History,Search,Billing}/`, `src/Vibra.Worker`, `tests/`, `scripts/export-openapi.sh`, docker-compose or Aspire AppHost |
| `vibra-ios` | `Vibra/` app target, `Packages/` (see section 7), `Vibra.storekit`, `scripts/sync-contract.sh` |
| `vibra-contracts` | `openapi.json` (generated by backend CI), `CHANGELOG.md`, `.spectral.yaml`; semantic-version tags |
| `vibra-db` | Database project; **scope not decided** (see pending items, given Dapper has no built-in migrations) |
| `vibra-docs` | Decisions (this document), later sprint exports |

**Contract workflow:** backend change merged → backend CI exports the spec and opens a PR in `vibra-contracts` → review `oasdiff` + CHANGELOG → merge and tag → in `vibra-ios` run `./scripts/sync-contract.sh` and rebuild.

**CI/CD:** GitHub Actions (backend: build, test, publish container); Xcode Cloud or Actions for iOS.

---

## 11. Build order

1. **Walking skeleton:** login, list tracks from the API and play a fixed track with background audio and lock screen controls. *(The player is the riskiest piece: do it first.)*
2. **Catalog:** artists, albums, search, cover art from CDN, with prices.
3. **Library:** playlists, likes, history.
4. **Billing:** plans, track/album purchase, subscription, entitlements endpoint and 403 with preview.
5. **Paywall and Store in iOS** (StoreKit 2), with 30-second accelerated renewal for the demo.
6. **Ingest** (FLAC validation, metadata, AAC previews), offline downloads, widget + Live Activity, and extras (HLS, Normal/Lossless quality, SignalR, App Intents).

---

## 12. Name and descriptions

**Name:** Vibra
**Tagline:** *Todo el catálogo, o solo lo que amas.*

**Short description (user-facing, shown in Spanish in the app):**
> **Vibra** is a music streaming app for iOS with two ways to enjoy music: subscribe (monthly or annual) to access the whole catalog, or buy songs and albums to keep forever. It plays in the background, can be controlled from the lock screen, and shows what is playing in a widget and the Dynamic Island.

**Technical description (README):**
> **Vibra** is a demo music streaming project made of a native SwiftUI iOS app and an ASP.NET Core 11 backend built as a modular monolith.
>
> **What it demonstrates**
> - **Streaming:** audio served from object storage through signed URLs, with background playback.
> - **Simulated monetization:** auto-renewable subscriptions and perpetual purchases with StoreKit 2, and a backend entitlements module that decides what each user can listen to, with 30 s previews for locked content.
> - **Native iOS tools:** StoreKit 2, AVFoundation + MediaPlayer, and WidgetKit + ActivityKit.
>
> **Stack:** SwiftUI · Swift 6 · StoreKit 2 · AVFoundation · WidgetKit · ASP.NET Core 11 · Dapper · PostgreSQL · FFmpeg · OpenAPI.
>
> *Educational project. Payments are simulated and the music is freely licensed.*

> Before publishing anything under the name "Vibra", check availability on the App Store and GitHub (short names are often taken). Not an issue for a course demo.

---

## 13. Pending items and points to verify

- [ ] Confirm whether the course has a specific list or criteria for the "3 native tools".
- [ ] Verify whether local StoreKit works on a real iPhone with a free account (Simulator is certain).
- [ ] Verify App Groups and Background Modes with a free account when creating the widget.
- [ ] Check which OpenAPI version `swift-openapi-generator` supports and configure .NET 11 accordingly.
- [ ] Decide the minimum iOS version based on your Xcode and devices.
- [ ] Choose the hosting/storage provider (Azure Blob, S3 or R2).
- [ ] Gather a small freely licensed catalog **in real FLAC or WAV** (not converted MP3s).
- [ ] Test early that `AVPlayer` plays and seeks a remote FLAC with a signed URL and range requests.
- [ ] Decide whether to offer "Normal" (AAC) quality and whether "Lossless" is subscriber-only.
- [ ] Choose the database migration tool (Dapper has none) and decide the scope of `vibra-db`.
- [ ] Decide whether to generate the repository skeletons (folders, Billing module, `.storekit`, README).
- [ ] Decide sprint length and, once decisions are closed, break the work into sprints (see `CLAUDE.md`).

---

## 14. Corrections applied in this version

Compared with the original Spanish draft:

- EF Core 11 / `DbContext` → **Dapper + Npgsql** (no EF Core); removed "EF Core 11 requires .NET 11".
- Redis → **Valkey**.
- "Minimal API endpoint groups" → **controllers** (Minimal APIs are forbidden).
- Monorepo layout (old section 10) → **multi-repo**, consistent with decision #10.
- Typo "el 403" fixed.
- Added decisions 12–14 and code conventions taken from `vibra-backend/CLAUDE.md`.
- HLS removed from the "decided" streaming description (it is optional).

---

## Sources

- .NET 11 announcement: https://devblogs.microsoft.com/dotnet/?p=61205
- InfoQ, .NET 11 release: https://infoq.com/news/2026/11/dotnet-11-release
- Apple, setting up StoreKit Testing in Xcode: https://developer.apple.com/documentation/xcode/setting-up-storekit-testing-in-xcode.md
- Apple, testing in-app purchases in Xcode: https://developer.apple.com/documentation/storekit/testing-in-app-purchases-in-xcode.md
- WWDC23 (sandbox and paid apps agreement): https://developer.apple.com/videos/play/wwdc2023/10142
- Expo, Apple Developer Program free vs paid: https://expo.dev/blog/apple-developer-program-explained-free-vs-paid
- Apple forums (Push and iCloud on personal teams): https://developer.apple.com/forums/thread/811929
- Flavio Copes, free vs paid account: https://flaviocopes.com/apple-developer-free-vs-paid/
