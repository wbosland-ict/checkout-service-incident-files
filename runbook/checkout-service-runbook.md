# Runbook: checkout-service

**Owner:** Checkout Platform Team
**Last reviewed:** 2026-02-14
**Next review due:** 2026-05-14

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

A rollback to the previous stable version to restore service is part of
handling a high priority incident and does not need a separate change
request. Announce it in the Teams *Incidents* channel before you start and
log it as an action in the TopDesk incident. If the rollback is a
workaround (the underlying defect is still there), register a problem in
TopDesk and link it to the incident before closing the incident.

| Step | Purpose | Command |
|---|---|---|
| 1 | Identify current and previous stable versions | `kubectl -n checkout rollout history deployment/checkout-service` |
| 2 | Roll back to the previous revision | `kubectl -n checkout rollout undo deployment/checkout-service` |
| 3 | Watch rollout status | `kubectl -n checkout rollout status deployment/checkout-service` |

## Escalation contacts

| When to escalate | Team | How to reach them | What to include |
|---|---|---|---|
| Issue traced to checkout-service itself (bad deploy, pod crash loop, config), or unresolved after 15 minutes | Checkout Platform Team lead | Teams call to the on-call engineer on the *Checkout Platform* on-call rota **and** add operator group *Checkout Platform* to the TopDesk incident | TopDesk incident number, alert name, current impact (error rate/latency), what you've already tried |
| Elevated errors/timeouts calling payment-gateway, or payment-gateway status page shows an incident | Payments integration on-call | Post in Teams *Engineering > Payments* **and** Teams call to the *Payments* on-call | TopDesk incident number, error codes/rates seen from payment-gateway, time window, checkout-service version |
| DB connection pool saturation, slow queries, or other Postgres/infra-level symptoms | Database/infra on-call | Post in Teams *IT Operations > Infra* **and** Teams call to the *DB/Infra* on-call | TopDesk incident number, pool utilization %, active/idle connection counts, relevant query/log excerpts |

For high priority incidents (customers actively impacted), always **call**
the on-call engineer via Teams; a channel post alone is only for non-urgent
updates or once the on-call has been reached. Log every escalation and every
action taken (including rollbacks) in the TopDesk incident.
