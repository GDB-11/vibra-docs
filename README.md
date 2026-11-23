# Vibra Docs

Decision records for **Vibra**, a music streaming demo (native iOS app + ASP.NET Core 11 backend) built for an iOS course. *Todo el catálogo, o solo lo que amas.*

Split from the original `Vibra-Decisiones.md`. Implementation-level rules live in each sibling repo's `CLAUDE.md`; this repo holds the *why*.

## Documents

| File | Content |
|---|---|
| [overview.md](overview.md) | Project summary, stack, architecture diagram |
| [ios-native-tools.md](ios-native-tools.md) | The 3 native iOS tools required by the course |
| [build-order.md](build-order.md) | Suggested build order |
| [naming-and-copy.md](naming-and-copy.md) | Name, tagline, descriptions |
| [open-questions.md](open-questions.md) | Single pending / to-verify list |
| [sprints/sprint-1.md](sprints/sprint-1.md) | Sprint 1 export (Sprintmodus, Spanish) |

## Decisions

| # | Decision | Status |
|---|---|---|
| 001 | [Modular monolith](decisions/001-modular-monolith.md) | Decided |
| 002 | [PostgreSQL with Dapper + Npgsql, DbUp migrations](decisions/002-postgres-dapper-npgsql.md) | Decided |
| 003 | [Audio in Cloudflare R2 + CDN; Hetzner VPS](decisions/003-audio-storage-cdn.md) | Decided |
| 004 | [Audio format: FLAC for everyone](decisions/004-audio-format-flac.md) | Decided (tiers Rejected, HLS Idea) |
| 005 | [REST/JSON + OpenAPI + generated Swift client](decisions/005-rest-openapi-swift-client.md) | Decided |
| 006 | [SignalR "now playing" sync](decisions/006-signalr-now-playing.md) | Idea |
| 007 | [Auth: Identity + JWT + refresh tokens](decisions/007-auth-identity-jwt-refresh.md) | Decided |
| 008 | [iOS: MVVM + local Swift packages](decisions/008-ios-mvvm-local-packages.md) | Decided |
| 009 | [Entitlements computed in the backend](decisions/009-entitlements-backend.md) | Decided |
| 010 | [Simulated payments: StoreKit 2 + `.storekit`](decisions/010-simulated-payments-storekit2.md) | Decided |
| 011 | [Multi-repo layout](decisions/011-multi-repo-layout.md) | Decided |
| 012 | [Valkey cache](decisions/012-valkey-cache.md) | Decided (optional) |
| 013 | [MVC controllers, no Minimal APIs](decisions/013-controllers-not-minimal-apis.md) | Decided |
| 014 | [Result pattern (BindSharp)](decisions/014-result-pattern-bindsharp.md) | Decided |
| 016 | [Podman as container runtime](decisions/016-podman-containers.md) | Decided |
| 015 | Name: **Vibra** (see [naming-and-copy.md](naming-and-copy.md)) | Decided |
| R01 | [Microservices](decisions/r01-microservices.md) | Rejected |
| R02 | [gRPC / GraphQL](decisions/r02-grpc-graphql.md) | Rejected |
| R03 | [APNs push](decisions/r03-apns-push.md) | Rejected |
| R04 | [Sign in with Apple](decisions/r04-sign-in-with-apple.md) | Rejected |
| R05 | [CarPlay](decisions/r05-carplay.md) | Rejected |
| R06 | [Backend-side fake payment gateway](decisions/r06-backend-fake-payments.md) | Rejected |

Never silently change a `Decided` item: propose and ask first.
