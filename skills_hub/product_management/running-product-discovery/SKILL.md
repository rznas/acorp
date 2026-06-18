---
name: running-product-discovery
description: >-
  Guides product discovery the INSPIRED (Marty Cagan) way: validating an idea's
  value, usability, feasibility, and business-viability risks (plus the ethics
  question "should we build it?") with fast, cheap iterations BEFORE asking
  engineers to build production-quality software. Acts as the discovery umbrella
  and router. Use when someone wants to run product discovery, validate or
  de-risk a product idea or feature, figure out "should we build it", decide
  whether an idea is worth building, set up a discovery process, or choose which
  discovery technique fits their situation. Routes to sibling technique skills
  for framing/planning, ideation, prototyping, and testing.
---

# Running Product Discovery

This is the **discovery umbrella / router skill**. Use it to understand the
purpose of discovery, frame which risks matter, and then route to the right
sibling skill for the actual techniques.

## Purpose of discovery

Product discovery exists to make sure that when we ask engineers to build a
production-quality product, it won't be wasted effort. Building real product is
expensive (scalable, performant, tested, instrumented, localized, maintainable,
on-brand, releasable with confidence) — so before that, we collect *evidence*
that the idea is worth it.

Discovery addresses four core **risks**:

- **Value risk** — Will the customer buy this, or choose to use it? (Usually the
  hardest and where most discovery time goes. Without value, nothing else
  matters.)
- **Usability risk** — Can the user figure out how to use it?
- **Feasibility risk** — Can we build it, with the time, skills, and technology
  we have?
- **Business-viability risk** — Does this solution work for the rest of our
  business (finance, marketing, sales, legal, business development, brand,
  executives)?

It is not enough for these to be the product manager's *opinion*. We collect
evidence on real users and customers.

A fifth risk applies in some cases: **ethics — "should we build it?"** Just
because we *can* build something that meets a business objective doesn't mean we
*should*; watch for solutions that hit goals (engagement, growth, monetization)
but harm users or the environment. When a significant ethical risk is found,
look for an alternative solution. Raise ethics issues with management from a
strong understanding of how the business makes money — identify the issue and
bring potential solutions, don't police.

## The 10 principles of product discovery

These drive *how* we work, and let you absorb new techniques as they emerge.

1. **We can't count on customers (or executives or stakeholders) to tell us what
   to build.** Customers don't know what's possible; none of us know what we
   really want until we see it. Our job is to find a solution that solves the
   underlying problem.
2. **The most important thing is to establish compelling value.** We can survive
   usability or performance issues for a while; without core value we have
   nothing. Spend most discovery time here.
3. **Good user experience is usually harder than the engineering — and more
   critical to success.** Not every team uses its design skills the way it
   needs to.
4. **Functionality, design, and technology are inherently intertwined.**
   Technology enables functionality and design; design enables functionality.
   This is why PM, designer, and tech lead should sit adjacent and work
   together.
5. **Most ideas won't work out, and the ones that do need several iterations.**
   "The most important thing is to know what you can't know." Be open to solving
   the problem in different ways.
6. **Validate ideas on real users and customers** — before building, not after.
7. **Validate the fastest, cheapest way possible.** Discovery is about speed, so
   we can try many ideas and multiple approaches for the promising ones.
8. **Validate feasibility during discovery, not after.** If the first time
   engineers see an idea is at sprint planning, you've failed. Early engineering
   input also improves the solution.
9. **Validate business viability during discovery, not after.** Don't find out
   after building that the PM missed an essential aspect of the business.
10. **It's about shared learning.** Teams of missionaries (not mercenaries) have
    seen the customer's pain together and learned together which ideas failed
    and which worked.

## Output of discovery

The output of discovery is a **validated product backlog** — ideas with evidence
that they are worth the cost of being built into production-quality product.
Delivery turns that backlog into a releasable product.

## Discovery iterations

An iteration in discovery means **trying out at least one new idea or
approach**. Teams competent in modern discovery techniques run on the order of
**10–20 iterations per week**. Many iterations never leave you, your designer,
and your tech lead — the act of building a prototype often exposes the problem
itself. Rule of thumb: a discovery iteration should take **at least an order of
magnitude less time and effort than a delivery iteration.**

Key tension: we must **learn fast in discovery yet release with confidence in
delivery.** Most discovery techniques don't require engineers' build time, and
the goal is to get ideas in front of real customers *without* pushing fragile
experiments into production.

## The technique taxonomy

Discovery techniques fall into categories. There is no single perfect taxonomy —
some techniques serve multiple situations.

| Category | What it does |
| --- | --- |
| **Framing** | Clarify the underlying problem behind a proposed solution; tease out which risks are significant and where to focus; align with other teams. |
| **Planning** | Identify the bigger challenges and plan how you'll attack the discovery work. |
| **Ideation** | Generate a wealth of promising solutions aimed at the problems you're focused on now. |
| **Prototyping** | Build a prototype (the go-to discovery tool) — four main types, each suited to different risks. |
| **Testing** | Quickly try out an idea to separate good from bad — split into testing feasibility, usability, value, and business viability. |
| **Transformation** | Migrate the organization from how it works today to how it needs to work. |

## How to choose a technique — only validate what you need to

Many ideas carry little risk and are straightforward; some have a single risk
(e.g. a legal/privacy concern); occasionally a tough problem is risky across
most or all areas. **Don't validate everything by reflex.** First frame the
idea to identify which risk(s) are actually significant, then pick the technique
category that addresses those risks.

### Decision guide: risk → technique category → sibling skill

| If the significant risk is... | Use this category | Go to sibling skill |
| --- | --- | --- |
| Unclear problem / which risks even matter / coordinating with other teams | Framing & Planning | `../framing-and-planning-discovery/SKILL.md` |
| Need promising solution ideas for the focused problem | Ideation | `../generating-product-ideas/SKILL.md` |
| Need something tangible to put in front of users/engineers/business | Prototyping | `../prototyping-for-discovery/SKILL.md` |
| **Value** — will they buy / choose to use it? | Testing (value) | `../testing-product-risks/SKILL.md` |
| **Usability** — can they figure out how to use it? | Testing (usability) | `../testing-product-risks/SKILL.md` |
| **Feasibility** — can we build it? | Testing (feasibility) | `../testing-product-risks/SKILL.md` |
| **Business viability** — does it work for the business? | Testing (business viability) | `../testing-product-risks/SKILL.md` |

Note: framing/planning, ideation, and prototyping usually precede or feed
testing. A typical flow is **frame the risks → plan → ideate → prototype →
test only the significant risks**, iterating fast and cheap.

## Sibling technique skills (the toolkits)

This umbrella skill intentionally holds no detailed technique instructions — it
points to the sibling skills that do:

- **framing-and-planning-discovery** — framing the problem and risks, and
  planning the discovery effort (`../framing-and-planning-discovery/SKILL.md`).
- **generating-product-ideas** — ideation techniques for promising solutions
  (`../generating-product-ideas/SKILL.md`).
- **prototyping-for-discovery** — the four prototype types and when to use each
  (`../prototyping-for-discovery/SKILL.md`).
- **testing-product-risks** — testing feasibility, usability, value, and
  business viability (`../testing-product-risks/SKILL.md`).

Start here to frame which risks matter, then hand off to the matching sibling
skill to do the work.
