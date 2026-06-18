---
name: prototyping-for-discovery
description: >-
  Toolkit for choosing and building the right discovery prototype to tackle a
  product risk fast and cheap, based on Marty Cagan's INSPIRED. Explains the
  five principles of prototypes and the four prototype types (feasibility, user,
  live-data, hybrid/Wizard of Oz), with a selector that maps each risk to the
  right type and fidelity. Use when someone wants to prototype an idea, test a
  mockup or wireframe, validate a workflow or user experience, prove out a
  feature with real data, de-risk a hard technical or algorithm problem, or set
  up a Wizard of Oz, concierge, feasibility, user, or live-data prototype.
  Triggers include "prototype", "mockup", "wireframe", "wizard of oz",
  "feasibility prototype", "test a mockup", "low/high fidelity", "fake it",
  "build things that don't scale".
---

# Prototyping for Discovery

A prototype lets you learn at a fraction of the cost of building a real product. The skill is picking the *right kind* of prototype for the risk you're tackling, at the *right level of fidelity*. There is no single "prototype" — there are four distinct types, each suited to different things, and using the wrong one is a common mistake.

## Five principles of prototypes

Apply these regardless of which type you build:

1. **Order-of-magnitude cheaper.** Every prototype should take *at least* 10x less time and effort than the eventual product. If it doesn't, it's not a prototype.
2. **Forces deeper thinking.** The act of creating a prototype forces you to think through the problem far more deeply than talking or writing about it. This is *why* it exposes major issues that would otherwise stay hidden until much later.
3. **Enables collaboration.** A prototype is a shared artifact. Product team and business partners can all experience it and build shared understanding.
4. **Choose appropriate fidelity.** Fidelity = how realistic it looks. There is no one correct fidelity. Lower fidelity is faster and cheaper, so **only go high-fidelity when the question actually requires it.** Match fidelity to purpose, not habit.
5. **Prototype as spec.** Beyond de-risking, a prototype often communicates to engineers and the org what needs to be built. It's frequently enough on its own; for complex products or distributed teams, supplement it with use cases, business rules, and acceptance criteria.

**What prototypes are NOT for:** proving your product will *sell* or that customers truly value it. People say they love a slick mockup, then do something different. Demand (value) validation needs other techniques — never conclude "it's validated" because 10–15 people liked a high-fidelity prototype.

## Selector: risk to prototype type

| Risk / question you're tackling | Prototype type | Who builds it | Fidelity | Effort |
|---|---|---|---|---|
| Is this technically possible? Algorithm, performance, scalability, new tech / 3rd-party / legacy / cross-team dependency | **Feasibility** | Engineers | n/a (often no UI) | 1–2 days (longer for major new tech, e.g. ML) |
| Will the experience, workflow, navigation, or information architecture work? How will people use this? | **User** | Designers | Low to high (choose deliberately) | Fast; minutes to days |
| Does this idea actually *work* in practice? Need real evidence/analytics for 1–2 use cases (relevance, funnels, game dynamics, social features) | **Live-data** | Engineers | Functional but unpolished | 5–10% of full build; days to a week |
| Want to learn fast about a not-yet-automatable experience, qualitatively | **Hybrid / Wizard of Oz** | Designers + a human "wizard" | High-fidelity front end, manual back end | Very fast |

Quick rule: technical risk to feasibility; experience/usability risk to user; need real-data evidence to live-data; need to fake automation to learn to Wizard of Oz.

## The four prototype types

### Feasibility prototype
Engineers write **just enough throwaway code** to mitigate a *technical* risk before committing to delivery. Typically quick-and-dirty: no UI, no error handling, no productization — just enough to answer "can we do this / will performance hold?"

- Use for: algorithm, performance, scalability, fault-tolerance concerns; unfamiliar technology, third-party component, or legacy system; dependencies on other teams' changes.
- Engineers estimate the time; the PM decides whether the idea is worth that time (or whether a lower-risk approach exists).
- This is **discovery work, not delivery work.** Skipping it is the usual root cause of teams grossly underestimating delivery effort.

### User prototype
A **simulation** — a façade with nothing real behind it (you can "buy" the bike, but nothing happens). Best tool for testing the **user experience, workflow, and information architecture**, and a top communication tool.

- **Low fidelity** = interactive wireframes. Fast; tests information and workflow only — says nothing about visual design or real data.
- **High fidelity** = looks and feels real with realistic (but fake/static) data. Use only when visual design or realism matters for the question.
- Built by designers with prototyping tools (or hand-coded, if fast and treated as disposable).
- **NOT for** proving relevance of real results, or for market/value validation. The search always returns the same fake bikes — fine for testing the shopping flow, useless for testing result relevance.

### Live-data prototype
A **very limited real implementation** (≈5–10% of productization effort) that runs well enough to handle 1–2 specific use cases against **real data sources and real, limited traffic**, instrumented with analytics. Its purpose is to **collect actual evidence** comparing the new approach to today's product or to expectations.

- Use for: search-result relevance, product funnels, game dynamics, many social features — anything where you need data, not opinions.
- Skips: full use-case coverage, automated tests, full analytics, i18n/l10n, performance/scalability, SEO.
- **Engineers build it** (it's code, not a designer mockup).
- **NOT shippable.** If results are good and you productize, engineers must do the real delivery work. The PM does **not** get to declare it "good enough," and must make sure executives/stakeholders understand its limits.

### Hybrid / Wizard of Oz
Combines aspects of the others. The classic **Wizard of Oz**: a high-fidelity front-end experience with **a real human behind the scenes** doing manually what would eventually be automated.

- Embodies the **"build things that don't scale"** philosophy — be clever to learn fast.
- Absolutely not scalable; never send significant traffic to it.
- Example: an "automated" 24/7 chat helper where a team member actually composes replies, gradually mixing in system-generated responses (and even a live-data prototype of the algorithm). A **concierge test** (manually delivering the service to learn how it should work) is a related technique.
- Learning is mainly **qualitative** — which is often where the biggest insights come from.

## Choosing fidelity

- Default to the **lowest fidelity that answers your question** — it's faster and cheaper.
- Raise fidelity only when the question depends on realism: visual design impact, "does this feel real," or stakeholder/user reactions that require believability.
- Feasibility prototypes usually need no fidelity (often no UI at all); Wizard of Oz needs a *high*-fidelity front end so users believe it's real.
