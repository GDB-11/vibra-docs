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
- Double purchase: blocked by partial unique indexes on (`UserId`, `TrackId`) and (`UserId`, `AlbumId`).

## Data model (Billing)
`Plan`, `Subscription`, `Payment` and `Purchase`: columns and constraints in [018](018-database-schema.md). Catalog `Track` and `Album` have `PriceCents`, `Currency` and `StoreProductId`. Money = integer cents + ISO 4217; timestamps ISO 8601 UTC.

**Changed 2026-11-26** (approved with 017/018):
- `Purchase.item_id` became nullable `TrackId` / `AlbumId`, so both can have FKs; this replaces `UNIQUE(user_id, item_type, item_id)`.
- `Plan.code` (`monthly`/`annual`) was dropped: `Interval` identifies the plan, and `PlanCode` is now the public UUID.
- `Currency` was added to `Payment` and `Purchase`.

In the endpoints below, `planCode` is that UUID (read from `GET /v1/plans`); `itemId` and `{id}` carry the item's public code, never the internal ID.

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
