# 007 — Auth: ASP.NET Core Identity + short JWT + rotating refresh tokens

**Status:** Decided

## Context
Users need accounts; the demo must not depend on a paid Apple Developer account.

## Options considered
- Email/password with Identity + JWT (chosen)
- Sign in with Apple → [R04](r04-sign-in-with-apple.md) (needs paid account)

## Choice
ASP.NET Core Identity, short-lived JWT, rotating refresh tokens. Email/password in the demo.

## Consequences
- iOS: tokens only in the Keychain; refresh inside a single actor so concurrent 401s trigger one refresh.
- Secrets (JWT keys) never committed: user-secrets locally, env vars in deployment.
