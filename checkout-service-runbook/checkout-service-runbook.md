# Runbook: checkout-service

**Owner:** Checkout Platform Team
**Last reviewed:** 2026-02-14
**Status:** ⚠️ Not updated since payment-gateway migration or last capacity review — treat DB/connection-pool sections as unverified.

## Service overview

checkout-service handles cart validation, inventory reservation, payment
processing, and order creation. Runs as 12 pods behind the main load
balancer. Depends on: cart-items DB (Postgres), inventory-service,
payment-gateway (3rd party).

## Alert: High latency / elevated error rate

1. Check Grafana dashboard `checkout-service-overview` for p95 latency and
   5xx rate trends.
2. Check recent deploys in the CI/CD dashboard — roll back if a deploy
   correlates with the onset of the issue.
3. Check pod health (`kubectl get pods -n checkout`) — restart pods if
   CPU/memory looks abnormal.
4. If inventory-service or payment-gateway status pages show incidents,
   escalate to the respective on-call.
5. Escalate to Checkout Platform Team lead if unresolved after 15 minutes.

## Alert: Pod crash loop

1. Check pod logs for `OutOfMemoryError` or unhandled exceptions.
2. Roll back to previous stable version if crash loop began after a deploy.
3. Scale up replica count temporarily if load-related.

## Known past incidents

- **2025-11-03:** Inventory-service timeout caused checkout failures.
  Fixed by increasing `inventory.timeout_ms` from 300 to 500 (later raised
  again to 800 in PR #4730).
- **2025-08-19:** Bad feature flag rollout caused a client-side JS error
  blocking checkout submission. Fixed by disabling the flag.

## Rollback procedure

| Step | Purpose | Command |
|---|---|---|
| 1 | Identify current and previous stable versions | `kubectl -n checkout rollout history deployment/checkout-service` |
| 2 | Roll back to the previous revision | `kubectl -n checkout rollout undo deployment/checkout-service` |
| 3 | Watch rollout status | `kubectl -n checkout rollout status deployment/checkout-service` |

## Escalation contacts

| When to escalate | Team | How to reach them | What to include |
|---|---|---|---|
| Issue traced to checkout-service itself (bad deploy, pod crash loop, config), or unresolved after 15 minutes | Checkout Platform Team lead | Page PagerDuty schedule `checkout-platform-primary` | Alert name, current impact (error rate/latency), what you've already tried |
| Elevated errors/timeouts calling payment-gateway, or payment-gateway status page shows an incident | Payments integration on-call | Post in `#eng-payments` **and** page PagerDuty schedule `payments-primary` | Error codes/rates seen from payment-gateway, time window, checkout-service version |
| DB connection pool saturation, slow queries, or other Postgres/infra-level symptoms | Database/infra on-call | Post in `#infra-oncall` **and** page PagerDuty schedule `db-infra-primary` | Pool utilization %, active/idle connection counts, relevant query/log excerpts |

Page via PagerDuty for anything actively impacting customers; use the Slack
channel alone only for non-urgent updates or once the relevant on-call has
already been paged.

---
> **Gap note (intentional for this exercise):** This runbook has no
> section for "DB connection pool saturation" and does not mention how to
> check or safely adjust `max_pool_size`, nor any guidance on retry-storm
> effects on downstream dependencies like payment-gateway. Part of
> Exercise 3/4 is identifying and filling this gap.
