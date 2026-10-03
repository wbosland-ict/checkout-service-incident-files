# Payment reconciliation — 2026-07-07 10:00–10:40 UTC

**Prepared by:** Finance Operations, 2026-07-08 09:30 UTC

| Metric | Value |
|---|---|
| Checkout attempts in window | 18,420 |
| Failed / abandoned checkouts | 5,760 |
| Successful checkouts | 12,660 |
| Duplicate charges detected (same customer, same cart, < 60 s apart) | **7** |
| Duplicate charges refunded | 7 (refunded 2026-07-08 by Finance; customers emailed) |
| Total value of duplicates | € 612.40 |
| Average order value (July) | € 74.80 |

**Finance note:** All duplicates occurred where the first charge request
timed out on the checkout-service side (no response received within
3000 ms) but was actually processed by payment-gateway, after which
checkout-service retried the charge. payment-gateway supports an
`Idempotency-Key` header that would have prevented this; checkout-service
does not send it.
