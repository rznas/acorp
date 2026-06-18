# Testing Value

Customers don't have to buy; users don't have to use a feature. They only do so if they perceive real value. **Feature parity is not enough** — matching a competitor's features (even at a lower price) won't make people switch. To pull someone away from their current solution (often a homegrown one) and through the pain of migrating, your product must be perceived as **substantially better**. Good teams spend most of their time creating value: if value is there, everything else is fixable; if it's not, usability/reliability/performance don't matter.

There are three things to test: **demand**, **qualitative value** (the response/reaction), and **quantitative value** (efficacy — how well it solves the problem).

---

## 1. Demand Testing

One of the biggest wastes — and a top cause of failed startups — is building a product (testing usability, reliability, performance, everything) only to find people won't buy it, or won't even sign up for the trial. Same failure happens at the feature level: features ship and never get used. It's easily avoided.

Demand testing answers: *if we built an amazing solution, would customers even care about this problem — enough to buy and switch?* Don't assume demand. (Often demand is already established because you're entering an existing market; then the real challenge is being demonstrably better.)

### Fake-door demand test (feature level)
Use when contemplating a costly new feature (a big customer asked, a competitor has it, the CEO wants it) and you don't want to build it only to find it unused.
- Put the button/menu item exactly where it belongs in the UX.
- When the user clicks, instead of the feature, show a page explaining you're studying adding this and inviting them to talk (collect email/phone).
- **Critical:** users must have NO visible indication it's a test until after they click.
- Yields: click-through rate to compare against expectations/other features, plus a list of willing interviewees.

### Landing-page demand test (product level)
Same idea for an entire new offering. Set up the product funnel's landing page describing the offering exactly as if launching for real. If the user clicks the call to action, instead of signing up they see a page saying you're studying this offering and would like to talk.

### Exposure / risk control
- Early startup: show to every user.
- Larger company: show to a small percentage of users or a specific geography. For things that risk brand or revenue, an **A/B test with 1% or less exposed** is usually fine. More conservative: **invite-only live-data test**, or use **customer discovery program customers under NDA**.

In practice demand is usually NOT the problem — people do sign up. The problem is they try it and don't get excited enough to switch. That's what qualitative and quantitative testing address.

---

## 2. Qualitative Value Testing

Quantitative testing tells you *what* is happening but not *why* or what to fix. Qualitative testing is about rapid learning and big insights — **not proving anything**. No single user gives the answer; each is a puzzle piece until you see where you went wrong.

This is **probably the single most important discovery activity**. Aim for **two or three qualitative value tests every single week**.

### Sequence: interview → usability → value
1. **Interview first.** A short user interview: confirm they have the problem you think, how they solve it today, what it would take to switch.
2. **Usability test next.** A value test is ALWAYS preceded by a usability test — the user must understand what the product is and how it works before you can have a useful value conversation. Run them back-to-back. Skipping usability turns the value test into a focus group of hypotheticals (useful for market insight, useless for discovering the product to build).
3. **Value test.** Use a **high-fidelity user prototype** (or live-data/hybrid). High-fi realism matters especially for value.

Attendees: at minimum the PM and product designer. Push to have an engineer watch — the magic often happens there.

### Willingness-to-pay signals (catch people being merely "nice")
People are nice and won't tell you what they really think, so all value tests check whether the person is *not just being polite*. Look for them to "pay" — even though you don't actually collect anything:
- **Credit card.** Would they pay right now? Look for them pulling out a card to buy (don't take the number).
- **Letter of intent.** For expensive B2B products beyond a credit card, ask for a non-binding "letter of intent to buy."
- **Reputation / referral / NPS.** Likelihood to recommend (0–10 scale), willingness to share on social media, willingness to enter their boss's or friends' emails for a recommendation.
- **Time.** Especially B2B: would they schedule significant time with you to work on this (even if you don't need it)?
- **Access.** Would they hand over login credentials for the product they'd switch from (you cite a "migration utility")? — tests true willingness to switch right now.

### Iterate the prototype
As soon as you suspect a problem or want a different approach, try it. If two users respond very differently, figure out why (different customer types? different skill/domain knowledge? happy vs. unhappy with current solution?). If you can't get people interested or can't make it usable enough to realize the value, you may **shelf the idea** — that's a win: it saves the cost of building something customers won't buy, plus the opportunity cost of what engineering could build instead.

> **As product manager, attend EVERY single qualitative value test. Do not delegate it and don't hire a firm.** Your contribution comes from experiencing users first-hand reacting to your team's ideas.

---

## 3. Quantitative Value Testing

Qualitative is fast learning and big insights; quantitative is about **collecting evidence**. Sometimes you get statistically significant results (high-traffic consumer services); other times you set the bar lower and use actual usage data as useful evidence among other factors. This is the purpose of the **live-data prototype**. Technique choice depends on traffic, time, and risk tolerance. (Startup: little traffic/time, fine with risk. Established: lots of traffic, limited patience, risk-averse.)

### Discovery A/B test
The gold standard — the user doesn't know which version she sees, so the data is very predictive. Note this differs from **optimization A/B testing**:
- Optimization A/B: surface-level low-risk tweaks (call to action, button color), usually a **50:50 split**.
- **Discovery A/B: current product to 99% of users, live-data prototype to 1% or less.** Monitor it more closely. The goal is learning, not optimizing a split.

### Invite-only testing
For risk-averse companies, or when you lack the traffic to get useful results from 1% (or even 10%) anytime soon. Identify users/customers, contact and invite them to try an explicitly experimental version (they opt in). Data is less predictive than a blind A/B test (opt-ins skew early-adopter), but you get real users doing real work with the prototype. When they don't bite, follow up with a **qualitative test** to learn why.

### Customer discovery program
A variation of invite-only using customer-discovery-program members who've already opted into testing new versions and with whom you have a close relationship. For B2B this is typically the **primary** technique: ship them frequent live-data-prototype updates and compare their usage to your broader customers'.

### The role of analytics
Any capable PM today must be comfortable with data. Five uses: understand user/customer behavior, measure product progress, prove whether ideas work, inform decisions ("data beats opinions"), and inspire product work (surprises in the data spark breakthroughs). **Non-negotiable: if you put a feature in, instrument at least basic usage analytics — otherwise you can't know if it's working.**

**Core analytics set for most tech products:**
- User behavior (click paths, engagement)
- Business (active users, conversion rate, lifetime value, retention)
- Financial (ASP, billings, time to close)
- Performance (load time, uptime)
- Operational costs (storage, hosting)
- Go-to-market costs (acquisition cost, cost of sales, programs)
- Sentiment (NPS, customer satisfaction, surveys)

Often called KPIs. Remember: data shows *what* is happening, not *why* — use qualitative techniques to explain quantitative results.
