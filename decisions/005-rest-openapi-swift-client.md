# 005 — REST/JSON + OpenAPI + generated Swift client

**Status:** Decided. Sub-item *OpenAPI version compatibility*: **to verify**.

## Context
iOS and backend are separate repos; the contract must be shared and typed.

## Options considered
REST/JSON + OpenAPI (chosen). gRPC/GraphQL → see [R02](r02-grpc-graphql.md).

## Choice
- REST/JSON over HTTPS for all request/response traffic.
- OpenAPI as contract; Swift client generated with `swift-openapi-generator`.
- `openapi.json` produced by backend CI and committed to `vibra-contracts`.
- Audio: range requests (or HLS) directly from CDN/storage.

API conventions: `/v1` versioning · cursor pagination · Problem Details (RFC 7807) · ETags on catalog reads · `Idempotency-Key` on payment operations · stable camelCase `operationId`s (they become Swift method names).

## Consequences
- **To verify:** ASP.NET Core 11 emits OpenAPI 3.2 by default (breaking change). Check which version `swift-openapi-generator` supports and configure the .NET document accordingly (see `vibra-backend/scripts/export-openapi.sh`).
- Contract workflow: see [011](011-multi-repo-layout.md).

## Open items
- [open-questions.md](../open-questions.md) #4.
