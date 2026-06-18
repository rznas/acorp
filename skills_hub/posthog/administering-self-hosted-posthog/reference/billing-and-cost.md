# Billing & cost control (detailed)

## Table of contents
- [How PostHog bills](#how-posthog-bills)
- [Billing limits](#billing-limits)
- [Billing alerts](#billing-alerts)
- [Spike detection](#spike-detection)
- [Estimating usage](#estimating-usage)
- [Reducing cost](#reducing-cost)
- [Pre-paid plans](#pre-paid-plans)
- [Invoices, cards, recipients](#invoices-cards-recipients)

## How PostHog bills
Usage-based per product: events (product/web analytics), session recordings, feature-flag/experiment **requests**, and data pipelines (rows via batch exports + events via realtime destinations). Each product has a generous monthly free tier even on paid plans. Rationale: more usage = more value and more cost to process/store; MAU-based pricing would overcharge for low-value users.

## Billing limits
Organization > Billing > each product > "Set billing limit" > enter $ amount > Save. Repeat per product; remove via "Remove limit." **When the limit is hit, PostHog stops ingesting that product's data and the overage is lost forever** (not deferred). To capture more in the current period, raise the limit; for future periods, tune your implementation.

## Billing alerts
Automatic when limits are set: the org owner gets emails at **80% and 100%** of the billing limit, and also at **80% and 100%** of the free allotment.

## Spike detection
Automatic daily system, no setup. Detects unusual up/down usage swings (all products except data warehouse) — flags implementation bugs causing excess events, traffic surges, or outages (low usage). Uses statistical baselines from history, accounting for weekday/weekend variation. Emails are sent **only to customers without an assigned account owner**, consolidated into one daily email, and suppressed if one was sent in the last 7 days.

## Estimating usage
- **Option 1 (best)**: sign up free, run for ~1 week (covers weekdays + weekends), read projected volume/cost on the billing page.
- **Option 2**: estimate from MAU. Session replays ≈ MAU × sessions/MAU. Events ≈ MAU × monthly-events-per-MAU (varies by category; doc table examples: PostHog/B2B 87, financial reporting 44, cloud monitoring 22, document mgmt 54, speech-to-text API 583, crypto wallet 162, meditation app 118, fashion retail 31, event booking 8). Autocapture vs custom capture changes counts.
- Flag/experiment requests are charged per request (frontend vs backend differ).
- Then use the pricing calculator or the billing dashboard's projected figures.

## Reducing cost
- Product analytics: capture fewer unwanted events.
- Session replay: control which sessions you record.
- Feature flags: cut flag costs (e.g. local evaluation, bootstrapping).
- Set billing limits per product as a hard cap.
- Build a billable-usage dashboard from the PostHog template to monitor.

## Pre-paid plans
For longer-term commitments (typically 12-month term). Discounted PostHog credit usable across current and future products. Order form (PandaDoc e-sign) captures credit, term, discount %, payment terms (Net 30 default). Buy more credit at the same discount in the first 6 months. Roll over up to half of unused credit on a renewal of equal/higher spend. Still supports monthly spend controls. Via Sales.

## Invoices, cards, recipients
- Billing > "Manage card details and view past invoices" → Stripe (update card/info, download invoices/receipts).
- Billing emails go to **all owners + admins** and **cannot be re-pointed** — invite a finance alias as an admin to receive them.
- No trial needed (free tier is open-ended). Startup program: up to $50,000 credits; non-profits contact Sales.
