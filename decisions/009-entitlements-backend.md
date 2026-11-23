# 009 — Entitlements computed in the backend

**Status:** Decided

## Context
Two access models (subscription, purchase) need one authoritative rule for what a user can play.

## Choice
`IEntitlementService.CanPlay(userId, trackId)` returns true if **any** holds:
1. Active subscription (`status = Active` and `now < current_period_end`).
2. Purchase of the track.
3. Purchase of the album containing the track.

Otherwise: **403 + 30 s `previewUrl`** (freemium experience, useful paywall).

## Decided edge cases
- Expired subscription: catalog access lost; **purchases always kept**.
- Simulated refund: `Payment = Refunded` and the `Purchase` is removed (only case where a purchase is deleted).
- Album gains tracks later: a purchased album includes everything it has, today and tomorrow.
- Buying an album while owning single tracks: accepted with no discount, showing "You already have X of N" (prorated discount = optional extra).
- Double purchase: blocked by `UNIQUE(user_id, item_type, item_id)`.

## Data model (Billing)
```
Plan: id, code (monthly|annual), price_cents, currency, interval
Subscription: id, user_id, plan_id, status (Active|Canceled|PastDue|Expired),
              current_period_start/end, cancel_at_period_end
Purchase: id, user_id, item_type (Track|Album), item_id, price_cents,
          payment_id, purchased_at   UNIQUE(user_id, item_type, item_id)
Payment: id, user_id, amount_cents, status (Succeeded|Failed|Refunded),
         idempotency_key (UNIQUE), kind (Subscription|Purchase), created_at
```
Catalog `Track` and `Album` add `price_cents`, `currency`, `store_product_id`. Money = integer cents + ISO 4217; timestamps ISO 8601 UTC.

## Endpoints
```
GET    /v1/plans
POST   /v1/subscriptions                  { planCode, paymentMethod }   (Idempotency-Key)
DELETE /v1/subscriptions/current          → cancels at period end
POST   /v1/purchases                      { itemType, itemId, paymentMethod }   (Idempotency-Key)
GET    /v1/me/entitlements
GET    /v1/me/purchases
GET    /v1/tracks/{id}/stream             → 200 signed URL | 403 + previewUrl
POST   /v1/storekit/transactions          { productId, transactionId, expiresDate }   (demo mode)
```

## Consequences
Every rule and edge case needs a test; billing tests use a fake `TimeProvider`. Payment side: [010](010-simulated-payments-storekit2.md).
