# 012 — Valkey cache (optional)

**Status:** Decided (use is optional)

## Context
Hot catalog and sessions may benefit from a cache.

## Choice
**Valkey** (Redis-protocol compatible), optional. Replaces "Redis" from the original draft.

## Consequences
Use a Redis-compatible client such as StackExchange.Redis; **ask before adding** the NuGet package.
