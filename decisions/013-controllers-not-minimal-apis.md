# 013 — MVC controllers, no Minimal APIs

**Status:** Decided

## Context
Need a consistent HTTP layer that composes with the Result pattern ([014](014-result-pattern-bindsharp.md)) and yields stable OpenAPI `operationId`s.

## Choice
MVC controllers, thin, via a shared `FunctionalController`. One `IErrorHttpMapper<TError>` per controller converts module errors to Problem Details. **Minimal APIs are not used.**

## Consequences
- Never expose exception messages, SQL or stack traces.
- Rules: `vibra-backend/CLAUDE.md` (Controllers). Original draft's "Minimal API endpoint groups" was corrected.
