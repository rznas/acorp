# PostHog CDP catalog: sources & destinations

Full list of pre-built integrations, grouped for PM lookup. Each has a doc at `/docs/cdp/sources/<slug>` or `/docs/cdp/destinations/<slug>`. The catalog grows over time; the live list lives at `app.posthog.com/data-management/sources` and `.../destinations`.

> Self-hosted note: managed connectors and OAuth-based integrations run on PostHog infrastructure and may be unavailable or need extra config on self-hosted. The **Webhook** destination, **incoming-webhook**/**custom REST** sources, and **self-hosted object-storage** sources (your own S3/GCS/R2) are the most portable.

## Table of contents
- [Realtime destinations (40+)](#realtime-destinations)
- [Batch export destinations (warehouses)](#batch-export-destinations)
- [Sources — managed connectors (120+)](#sources--managed-connectors)
- [Self-hosted object-storage sources](#self-hosted-object-storage-sources)

---

## Realtime destinations
Stream each event out as it arrives. Use **Webhook** when no native template exists.

**Chat / alerting:** slack, microsoft-teams, discord, gleap

**Ad platforms / conversions:** google-ads, meta-ads, tiktok-ads, snapchat-ads, reddit-ads-conversion-api, reddit-ads-pixel

**CRM / customer engagement:** hubspot, salesforce, attio, intercom, customerio, braze, engage, june, avo, zendesk

**Email / SMS / messaging:** mailchimp, mailgun, mailjet, klaviyo, brevo, sendgrid, loops, knock, twilio, whatsapp, activecampaign

**Automation / no-code:** zapier, make, rudderstack

**Storage / streaming infra:** google-cloud-storage, google-pubsub, aws-kinesis, airtable

**Generic / other:** webhook (any HTTP endpoint), posthog (send to another PostHog project)

## Batch export destinations
Scheduled, reliable bulk export of events/persons/sessions to a data warehouse or lake:

- s3
- bigquery
- snowflake
- redshift
- postgres
- databricks
- azureblob (Azure Blob Storage)

Each: `/docs/cdp/batch-exports/<slug>`. See destinations-and-export.md for per-destination schema and permission notes.

## Sources — managed connectors
Provide credentials; PostHog runs extract-and-load into the data warehouse. Grouped by what a PM typically wants to join to product data.

**Payments / billing / revenue:** stripe, braintree, paddle, chargebee, recurly, recharge, gocardless, mollie, square, checkout-com, polar, revenuecat, chartmogul

**CRM / sales:** salesforce, hubspot, pipedrive, close, copper, attio, apollo, salesloft, clari, gong, freshsales, crunchbase, vitally

**Support / success:** zendesk, intercom, front, gorgias, freshdesk, kustomer, gladly, dixa, plain, aircall, delighted, productboard

**Marketing / email / ESP:** mailchimp, mailerlite, mailgun, mailjet, klaviyo, brevo, sendgrid, postmark, resend, drip, convertkit, omnisend, ortto, iterable, campaign-monitor, attentive, customer-io, braze, matomo, mixpanel, amplitude

**Ads:** google-ads, meta-ads, tiktok-ads, snapchat-ads, reddit-ads, pinterest-ads, linkedin-ads, bing-ads, amazon-ads, adroll, outbrain, taboola, appsflyer

**Databases / warehouses:** postgres, mysql, microsoft-sql-server, redshift, snowflake, bigquery, clickhouse, mongodb, elasticsearch, supabase, convex, pganalyze

**Object storage:** s3, gcs, azure-blob, r2 (Cloudflare), cloudflare

**E-commerce:** shopify, woocommerce, commercetools, lightspeed-retail, shipstation

**Product / engineering / devtools:** github, gitlab, linear, jira, sentry, datadog, pagerduty, incident-io, circleci, launchdarkly, optimizely, fullstory, pendo, shortcut

**HR / people / finance ops:** bamboohr, hibob, rippling, deel, personio, lattice, culture-amp, greenhouse, lever, workos, okta, brex, ramp, coupa, doit

**Productivity / docs / PM:** notion, asana, monday, clickup, trello, coda, confluence, smartsheet, google-sheets, airtable, guru, granola, buildbetter

**Forms / surveys / events / scheduling:** typeform, surveymonkey, eventbrite, calendly, zoom, webflow

**Identity / auth / monitoring:** clerk, okta, workos, servicenow, pingdom, google-search-console

**Custom:** custom-rest-source (any REST API), incoming-webhooks (push events in)

## Self-hosted object-storage sources
Link your own bucket; PostHog reads CSV/Parquet on every query. You control freshness/volume:

- s3 (AWS)
- gcs (Google Cloud Storage)
- azure-blob (Azure)
- r2 (Cloudflare)

These are the most self-host-friendly source option since they use your storage, not PostHog-managed connectors.
