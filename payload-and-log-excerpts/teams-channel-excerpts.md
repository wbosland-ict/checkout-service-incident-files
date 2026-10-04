# Microsoft Teams — channel excerpts

All times UTC, 2026-07-07.

## Team "IT Operations" > channel "SRE Alerts"

> **10:06 — Grafana Alerting (bot):** 🔴 [critical] checkout-service — High
> latency & elevated error rate. p95 2140ms, 5xx 6.8%.
>
> **10:07 — Grafana Alerting (bot):** 🔴 [critical] checkout-service-db — DB
> connection pool saturation. 50/50 active connections for 3 min.
>
> **10:09 — Grafana Alerting (bot):** 🟠 [warning] payment-gateway-client —
> Elevated 429 rate from payment-gateway (85/min).
>
> **10:11 — Grafana Alerting (bot):** 🔵 [info] order-notification-service —
> Email send queue depth 640 (> 500).

## Team "IT Operations" > channel "Service Desk"

> **10:08 — Service desk agent (L. de Vries):** Phones are lighting up —
> 11 calls in 6 minutes from customers saying checkout is stuck on
> "Processing..." or shows a generic error. Web and app both affected.
>
> **10:12 — Service desk agent (L. de Vries):** Also 4 webshop customers
> asking whether they've been charged twice. Registering a high priority
> incident now.
>
> **10:16 — Service desk agent (L. de Vries):** Incident **I 2607 041**
> registered and assigned to operator group *SRE – Checkout Platform*. Please
> pick it up.

## Team "Engineering" > channel "Payments"

> **10:10 — Payments integration engineer:** payment-gateway is seeing a
> wave of 429s from checkout-service, way more than normal traffic would
> explain. Their status page is green. Is something on our side retrying a
> lot?
