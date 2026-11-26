# 001 — Modular monolith

**Status:** Decided

## Context
One ASP.NET Core 11 backend for a course demo with six modules.

## Options considered
- Modular monolith (chosen)
- Microservices → see [R01](r01-microservices.md)

## Choice
Modular monolith. Modules: Identity, Catalog, Library, Streaming, History, Billing.

| Module | Responsibility |
|---|---|
| Identity | Registration, login, refresh tokens |
| Catalog | Artists, albums, tracks, prices, `StoreProductId`, search (Postgres full-text + `pg_trgm`) |
| Library | Playlists, likes, followed artists |
| Streaming | Signed URLs, preview URLs, ingest/upload |
| History | Recently played and play counts |
| Billing | Plans, subscriptions, purchases, payments, entitlements |

Per module: `Domain` · `Application` (vertical-slice use cases) · `Infrastructure` (Dapper + SQL, storage) · `Api` (controllers, error mapper).

## Consequences
- Modules talk through public interfaces or in-process events, never through another module's repositories, row models or SQL. Enforced with architecture tests (NetArchTest).
- Ingest/transcoding runs in a separate worker (`Vibra.Worker`).
- Implementation rules: `vibra-backend/CLAUDE.md`.

## Changes
- **2026-11-26:** the Search module was removed. With the search indexes on Catalog's tables it had nothing of its own, so Catalog serves search ([018](018-database-schema.md)). Genres were dropped from Catalog's scope (018).
