# 014 — Result pattern (BindSharp) everywhere

**Status:** Decided

## Context
Expected failures (not found, conflict, validation, payment declined) should be explicit in types, not exceptions.

## Choice
BindSharp `Result<T, TError>` in Infrastructure, Application services and controllers. No exceptions for expected failures. RFC 7807 Problem Details at the HTTP edge.

## Consequences
- NuGet policy: BindSharp, Dapper, Npgsql pre-approved; ask before adding others.
- `FunctionalController` and related helpers do not exist yet in `vibra-backend` (TODO there).
- Rules: `vibra-backend/CLAUDE.md` (Result pattern).
