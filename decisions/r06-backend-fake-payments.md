# R06 — Backend-side fake payment gateway (option A)

**Status:** Rejected. Chosen instead: [010 StoreKit 2 + `.storekit`](010-simulated-payments-storekit2.md).

## What it would have been
`FakePaymentGateway` with test cards (`4242…` approves, `…0002` declines, `…9995` approves then fails on renewal), injectable `TimeProvider`, a renewal `BackgroundService`, and dev-only endpoints `/dev/clock/advance` and `/dev/subscriptions/{id}/fail-renewal`.

## Note
If dev-only endpoints are ever added, register them only in the Development environment.
