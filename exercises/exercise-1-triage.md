# Exercise 1: Triage with AI

**Time box:** ~25 minutes
**Goal:** Use AI to rapidly turn noisy, multi-source signals into a
prioritized understanding of "what's broken and how bad is it."

## Setup

You've just been paged. Read `scenario/00-incident-brief.md` and
`scenario/01-architecture.md` if you haven't already. You have access to:

- `data/alerts/pagerduty-alerts.json` — 4 alerts fired in the last ~5 minutes
- `data/logs/checkout-service.log` and `data/logs/payment-gateway.log`

## Task

1. Feed the alert payload and/or log excerpts to your AI assistant and ask
   it to:
   - Summarize what's happening in plain language.
   - Rank the 4 alerts by likely priority/relevance to the customer-facing
     impact (checkout success rate).
   - Flag anything that looks like a **distraction or downstream symptom**
     rather than a primary issue.
2. Ask the AI to identify what additional evidence you should pull next
   (e.g., "check recent deploys," "check DB metrics") based on patterns in
   the logs.
3. Write a one-paragraph **initial incident summary** (as you would post in
   `#incidents`) — have the AI draft it, then edit it yourself.

## Questions to answer

- Which alert(s) matter most right now, and why?
- Which alert looks like a secondary/downstream effect rather than a root
  cause? What evidence made you (or the AI) conclude that?
- What is the current measured customer impact (be specific — cite numbers)?
- What's your next investigative step?

## Try these prompt angles

- "Here are 4 alerts that fired within 5 minutes of each other for a
  checkout service outage. Rank them by likely priority and explain which
  might be downstream noise. [paste alerts JSON]"
- "Summarize this log file for a non-technical stakeholder update in 3
  sentences. [paste log excerpt]"
- "Given this timeline of metrics, what's the earliest timestamp where
  things started to degrade? [paste CSV]"

## Watch out for

- AI confidently ranking alerts without asking for the missing context
  (e.g., not noticing the `order-notification-service` alert is almost
  certainly downstream noise until you point it at the "note" field).
- AI inventing plausible-sounding root causes at this stage — triage is
  about **impact and priority**, not root cause yet. Save deep root-cause
  reasoning for Exercise 2.
- Pasting full raw logs into a public/shared AI tool — in a real incident,
  scrub or check your org's data-handling policy for what's OK to paste
  into an external AI tool (customer PII, secrets, tokens, etc.).
