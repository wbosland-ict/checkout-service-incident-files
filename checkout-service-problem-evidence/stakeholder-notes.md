# Stakeholder notes (collected after the incident was closed)

## Teams — Product owner Checkout (M. Jansen), 2026-07-07 14:02 UTC

> Understood that v2.14.0 had to be rolled back. However, the "saved carts"
> feature (v2.15.0, planned for 2026-07-21) is built on top of that
> refactor. As long as we're stuck on v2.13.4 the team can't release
> anything from main. Can we get a proper fix planned? I'd like to know the
> earliest realistic date.

## Teams — Release manager, 2026-07-07 14:20 UTC

> Deploy freeze on checkout-service `main` until the problem behind
> I 2607 041 has an approved RFC. Hotfixes on the v2.13.x branch only.

## Teams — Database/infra on-call, 2026-07-07 15:05 UTC

> FYI the cart-items Postgres instance (db.m6g.xlarge) is configured with
> `max_connections = 200`. 12 checkout pods share the pool of 50 via
> PgBouncer, so there is headroom on the DB side, but raising the pool
> without fixing query efficiency would just move the bottleneck to DB
> CPU. Happy to review any pool change — please involve us in the RFC.

## Teams — Payments integration engineer, 2026-07-07 15:30 UTC

> payment-gateway confirmed the 300 req/min limit is per API key and can be
> raised to 600 req/min on request (2-week lead time, extra cost). They
> strongly recommend idempotency keys + exponential backoff on 429 instead
> of retrying immediately.
