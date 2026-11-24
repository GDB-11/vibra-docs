# 005 — REST/JSON + OpenAPI + generated Swift client

**Status:** Decided. Sub-item *OpenAPI version compatibility*: **OpenAPI 3.1 chosen**; end-to-end generator test with the real export **to verify** (Sprint 1 spike).

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
- **OpenAPI version: 3.1.** ASP.NET Core 11 emits OpenAPI 3.2 by default (`OpenApiVersion` moved from 3.1 to 3.2; breaking change). `swift-openapi-generator` supports 3.0 and 3.1; its README lists 3.2 as *preliminary* only. The .NET document is therefore pinned with `options.OpenApiVersion = OpenApiSpecVersion.OpenApi3_1` in `AddOpenApi`, and `vibra-backend/scripts/export-openapi.sh` should fail if the exported `openapi` field is not `3.1.x`.
- Revisit when the generator declares 3.2 stable. Do not bump the version without confirming generator support (rule in `vibra-contracts/CLAUDE.md`).
- **To verify:** run the generator on the real exported document (Sprint 1 spike). Also unconfirmed: exact Microsoft.OpenApi 3.x version used by ASP.NET Core 11 (sources disagree) and what the generator's preliminary 3.2 support covers.
- Sources: [MS Learn breaking change](https://learn.microsoft.com/aspnet/core/breaking-changes/11/microsoft-openapi-3x) · [swift-openapi-generator README](https://github.com/apple/swift-openapi-generator).
- Contract workflow: see [011](011-multi-repo-layout.md).

## Open items
- [open-questions.md](../open-questions.md) #4.
