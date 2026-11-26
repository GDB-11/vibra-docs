# 010 — Simulated payments: StoreKit 2 + local `.storekit` file

**Status:** Decided. The app runs **only in the Xcode Simulator** (2026-11-23), so free-account / real-device limits are out of scope.

## Context
Payments must be simulated, and StoreKit 2 is one of the 3 required native tools.

## Options considered
- **B (chosen):** StoreKit 2 + local `.storekit` behind `IPaymentGateway` (`StoreKitDemoGateway`).
- **A (rejected):** backend-side fake gateway → [R06](r06-backend-fake-payments.md).

## Choice and flow
1. `.storekit` defines an auto-renewable subscription (one group, monthly + annual) and **non-consumable** products for purchases.
2. App purchases with `Product.purchase()` and listens to `Transaction.updates`.
3. App sends the transaction to `POST /v1/storekit/transactions`.
4. Backend, **only with `Billing:DemoMode=true` (fail closed if missing), trusts the client** and creates the `Subscription` or `Purchase`.

## Consequences
- **Known limitation, document it:** production would verify the JWS with the App Store Server API.
- Every purchasable item is a distinct StoreKit product: define only a few albums/tracks (e.g. 5 and 10) and map via `StoreProductId`.
- Demo trick: Editor > Subscription Renewal Rate > Monthly Renewal Every 30 Seconds.

## Paid Apple account?
- Local StoreKit Testing in Xcode (Simulator): **no**.
- Sandbox, TestFlight, App Store: **yes** (US$99/year; sandbox needs the paid apps agreement).
- Real device with free account: **out of scope** (Simulator only). For reference, a "Personal Team" cannot sign paid capabilities and free-account apps stop opening after 7 days.
- If ever published: guideline 3.1.1 requires In-App Purchase for digital content.

## Open items
- None.

## Sources
- https://developer.apple.com/documentation/xcode/setting-up-storekit-testing-in-xcode.md
- https://developer.apple.com/documentation/storekit/testing-in-app-purchases-in-xcode.md
- https://developer.apple.com/videos/play/wwdc2023/10142
- https://expo.dev/blog/apple-developer-program-explained-free-vs-paid
- https://developer.apple.com/forums/thread/811929
- https://flaviocopes.com/apple-developer-free-vs-paid/
