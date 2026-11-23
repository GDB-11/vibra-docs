# 001 — Modular monolith

**Status:** Decided

## Context
One ASP.NET Core 11 backend for a course demo with seven functional areas.

## Options considered
- Modular monolith (chosen)
- Microservices → see [R01](r01-microservices.md)

## Choice
Modular monolith. Modules: Identity, Catalog, Library, Streaming, History, Search, Billing.

| Module | Responsibility |
|---|---|
| Identity | Registration, login, refresh tokens |
| Catalog | Artists, albums, tracks, genres, prices, `store_product_id` |
| Library | Playlists, likes, followed artists |
| Streaming | Signed URLs, preview URLs, ingest/upload |
| History | Recently played and play counts |
| Search | Postgres full-text + `pg_trgm` |
| Billing | Plans, subscriptions, purchases, payments, entitlements |

Per module: `Domain` · `Application` (vertical-slice use cases) · `Infrastructure` (Dapper + SQL, storage) · `Api` (controllers, error mapper).

## Consequences
- Modules talk through public interfaces or in-process events, never through another module's repositories, row models or SQL. Enforced with architecture tests (NetArchTest).
- Ingest/transcoding runs in a separate worker (`Vibra.Worker`).
- Implementation rules: `vibra-backend/CLAUDE.md`.
