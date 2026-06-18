---
name: structuring-empowered-product-teams
description: Designs and structures empowered, cross-functional product teams the way modern product organizations do, based on Marty Cagan's INSPIRED. Covers what makes a strong product team (dedicated/durable, missionaries not mercenaries, composition and size, empowerment and accountability, flat reporting, collaboration, co-location, scope/charter, autonomy) and the ten principles for splitting a product across many teams at scale, including the autonomy-vs-leverage tradeoff. Use when forming, sizing, scoping, or reorganizing product teams; defining team charters; deciding reporting structure; minimizing cross-team dependencies; introducing common-services/platform teams; or driving an org transformation toward empowered teams.
---

# Structuring Empowered Product Teams

Guidance for forming strong, empowered product teams and for splitting a product across many teams at scale. From Marty Cagan, *INSPIRED* (Chapters 9, 20, 21).

## When to use this skill

- Forming a new product team or splitting work across several teams.
- Sizing a team, defining its charter/scope, or setting reporting structure.
- Diagnosing why teams feel slow, blocked, or disempowered.
- Deciding how much autonomy a team gets vs. how much they leverage a shared foundation.
- Driving an organizational change toward the empowered product-team model.

## What a strong product team is

A product team is a group of people bringing together different specialized skills who feel **real ownership** for a product, or a substantial piece of a larger product. Such teams are called *dedicated*, *durable*, or *squads* (cross-functional). They are **not** assembled for a single project or feature.

If your company is not yet organized around dedicated product teams, fixing that is the single most important change. Everything else depends on it. You can start with one pilot team rather than reorganizing everything at once.

### Missionaries, not mercenaries

> "We need teams of missionaries, not teams of mercenaries." — John Doerr

Mercenaries build whatever they're told. Missionaries are true believers in the vision, committed to solving customer problems. A dedicated team should act and feel like a startup within the larger company — that's the intention.

### Team composition

A typical product team is:
- **1 product manager**
- **1 product designer** (omit only when there is no user-facing experience, e.g., programmatic APIs)
- **2 to ~10–12 engineers**

Optional additional members: product marketing manager, test automation engineer(s), user researcher, data analyst, and in larger orgs a delivery manager.

### Team size

- **Critical mass (minimum):** 1 PM + 1 designer (if user-facing) + 2 engineers. Fewer is below critical mass.
- **Practical upper bound:** ~8–12 engineers (the "two-pizza rule"). It's hard for one PM and one designer to keep more than ~10–12 engineers supplied with good work.
- Teams need not all be the same size. Balance of skills — to build the right things, and build them right — matters more than absolute size.
- Each team must have **one, and only one, product manager.**

### Empowerment and accountability

Teams exist to solve hard problems for the business. They are given **clear objectives**, are **empowered** to figure out the best way to meet them, and are **accountable for the results** (the outcome — not merely shipping).

### Reporting structure: flat, PM is not the boss

A product team is **not** about reporting relationships — it is intentionally flat. Members are individual contributors; there are no people managers on the team. People continue to report to their **functional manager** (engineers to an engineering manager, designer to head of design, PM to head of product).

**The product manager is not the boss of anyone on the team.**

### Collaboration

The relationship is **true collaboration** — product, design, and engineering working out solutions together — not a hierarchy. Collaboration is built on relationships, which is why durability and co-location matter.

### Co-location

Try hard to **co-locate** the team: close enough to easily see one another's screens. All else being equal, a co-located team substantially outperforms a dispersed one. Sitting together, eating lunch together, and building personal relationships creates a special dynamic. This is also a reason to prefer employees over contractors/agencies (more stable, easier to co-locate). Multiple company locations are fine — aim for co-located teams within each.

### Scope and charter

Two dimensions define a team's scope:
- **Type of work:** the team owns *all* the work for their product — projects, features, bug fixes, performance, optimizations, content changes — everything.
- **Scope of work:** a complete product (smaller companies), or more commonly a meaningful piece of a larger customer experience (e.g., fraud detection at eBay, newsfeed at Facebook).

Ways to slice the product: by user/customer type, device, workflow/customer journey, or — very often — by **architecture** (architecture drives the technology stack, which requires different engineering expertise). There is **never a perfect way** to carve up the pie; optimizing for one thing costs another. Alignment between **head of product and head of engineering** is critical — they jointly define team size and scope.

### Durability

Keep teams **together and stable**. People need time to gain enough domain expertise to innovate and to develop missionary-like ownership. A product team is not created to deliver a single short project and then disband.

### Autonomy

Give teams significant **autonomy**: not freedom to work on whatever looks fun, but freedom to solve their assigned problems **the best way they see fit**. Also **minimize dependencies** between teams (minimize, not eliminate — at scale you can't eliminate them, but track and continuously reduce).

### Why the model works

1. Collaboration is built on relationships — co-located, durable teams nurture them.
2. Innovation requires expertise — durability lets people go deep.
3. The full team understands the business objectives and context, and feels ownership for the **outcome** — they aren't off the hook just because something launched.

## Structuring teams at scale (the ten principles)

At scale (25, 50, 100+ teams), how you split the product across teams becomes a major factor in speed, empowerment, and accountability. **There is no recipe** — understand the principles, weigh the options for your circumstances, and decide. The ten principles:

1. Alignment with investment strategy
2. Minimize dependencies
3. Ownership and autonomy
4. Maximize leverage
5. Product vision and strategy
6. Team size
7. Alignment with architecture
8. Alignment with user or customer
9. Alignment with business
10. Structure is a moving target

See `ten-structuring-principles.md` for the full explanation of each principle, the common-services/platform team pattern, and the **autonomy-vs-leverage tradeoff** (with the factors to weigh and the two critical pieces of context every team needs).

## Transformation example: Lea Hickman at Adobe

In larger companies, the team model alone is not enough — it takes strong **product leadership** providing a compelling vision and strategy. In 2011, Lea Hickman led product for Adobe's Creative Suite (~$2B annual license revenue, ~half of Adobe's ~$4B). She drove the transformation from the desktop, annual-upgrade, license model to the subscription, multi-device **Creative Cloud** — across a 15-app integrated suite, touching finance, legal, sales, engineering, and a million+ resistant customers.

How she did it:
- Articulated a **compelling vision** of the new whole as greater than the sum of its parts, so interrelated pieces could move in parallel.
- Built **compelling prototypes** (with CTO Kevin Lynch) to rally executives and teams.
- Ran a sustained campaign of **continuous communication** — "no such thing as over-communication" — with leaders and stakeholders.

Result: Creative Cloud reached $1B+ recurring revenue faster than anyone, 9M+ subscribers, and Adobe more than tripled its market cap (~$60B). This is the model for a product leader driving massive, meaningful change in a large, successful enterprise — vision, strategy, and clear continuous communication.

## Key takeaways

- Dedicated, durable, cross-functional, co-located, empowered, accountable — these are the pillars.
- Missionaries, not mercenaries: ownership of outcomes, not just delivery.
- Flat structure; the PM is not the boss; people report to functional managers.
- 1 PM + 1 designer + 2–12 engineers; one PM per team.
- At scale, splitting the product is about tradeoffs guided by ten principles — there's no perfect or permanent structure.
- Large-company transformation needs strong product leadership: vision, prototypes, relentless communication.
