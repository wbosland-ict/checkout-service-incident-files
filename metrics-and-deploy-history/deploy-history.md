# Deploy History — checkout-service

| Version | Deployed at (UTC) | Author | Changelog |
|---|---|---|---|
| v2.14.0 | 2026-07-07 09:58 | ci-bot (PR #4821, approved by J. Alvarez) | Refactored cart items retrieval to use lazy-loaded associations (ORM), removed manual `JOIN FETCH` query. Cleans up ~80 lines of manual mapping code. |
| v2.13.4 | 2026-07-03 14:12 | ci-bot (PR #4790) | Bump logging library to 3.2.1 (security patch). No behavior change. |
| v2.13.3 | 2026-06-29 11:05 | ci-bot (PR #4772) | Add feature flag for "buy now pay later" option at checkout (disabled by default). |
| v2.13.2 | 2026-06-24 08:40 | ci-bot (PR #4755) | Fix currency rounding bug for JPY orders. |
| v2.13.1 | 2026-06-18 16:22 | ci-bot (PR #4730) | Increase request timeout for inventory-service calls from 500ms to 800ms. |

## Other relevant changes

- **payment-gateway** (3rd party): no changes on their side; their status
  page shows no incidents. Their published rate limit (300 req/min per
  client) has been unchanged for 6+ months.
- **cart-items DB**: no schema or config changes in the last 30 days.
  `max_pool_size` has been `50` since the last capacity review
  (2026-04-02), before recent traffic growth.
