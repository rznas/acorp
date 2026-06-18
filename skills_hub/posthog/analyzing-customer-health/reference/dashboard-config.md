# Dashboard event configuration

How to configure the customer analytics dashboard events so the insights are meaningful. Source: PostHog Customer Analytics docs (configure-your-dashboard, configure-dashboard-with-ai, start-here).

The dashboard works out of the box on `$pageview`, but to extract full value you configure events in **Customer analytics > configuration** (also at Settings > Customer Analytics). These five events also serve as a good starting checklist if you've just begun instrumenting.

## The five events

| Event | What it is | Used to calculate | Gotchas |
| --- | --- | --- | --- |
| **Activity event** | What a user must do to count as "active" | DAU/WAU/MAU, power users, engagement/lifecycle | Avoid plain `$pageview`; narrow by URL/property or use an action grouping product events. Can be a single event or a set |
| **Signup pageview event** | Pageviews of your signup page | Signup rate, average conversion time | Usually pageviews of a specific URL |
| **Signup event** | Fires when a user completes registration / creates an account | Growth, acquisition | Keep distinct from login/sign-in — signup happens once, sign-ins repeat |
| **Subscription event** | User enrolls into a plan | Subscription tracking (independent of payment) | May coincide with signup or with payment; tracked separately |
| **Payment event** | Successful payment | Paying customers, free-to-paid conversion | Track successful payments only |

PostHog's own example activity is an **action** encompassing events across products: `insight created`, `feature flag created`, `survey viewed`, etc.

## Setup tips

- Use **actions** to group multiple events under one name for easier analysis.
- Be as specific as possible — "sloppy definitions = sloppy metrics."
- If you already capture events but don't know which to use, use PostHog AI (see below).

## Configure with AI

On the configuration page each event has a **sparkle button** beside its name. Clicking it pre-fills PostHog AI with a prompt tailored to identify *that specific* event (each event has its own prompt). After research, PostHog AI returns a list of candidate events; copy the event name and search it in the event selector to assign it.

> Self-hosted note: PostHog AI is typically cloud-managed; the sparkle assist may be unavailable or need extra configuration on self-hosted. See the using-posthog-ai-assistant skill.

## Tracking the events (PM spec reference)

A PM specs these; engineering implements. Minimal shape:

```js
// Identify on account creation
posthog.identify('user_123', { email: 'user@example.com', created_at: new Date().toISOString() })

// Signup
posthog.capture('signed_up', {})

// Free-to-paid conversion
posthog.capture('user_subscribed', { plan: 'pro', amount: 2900 })
```

Then map each captured event to its slot in Settings > Customer Analytics.
