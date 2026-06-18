---
name: applying-the-product-operating-model
description: Applies Marty Cagan's INSPIRED foundations for building technology-powered products. Covers the three overarching principles (tackle risks up front, design collaboratively, solve problems/outcomes not features), the four product risks (value, usability, feasibility, business viability) plus ethics, continuous discovery and delivery, MVP-as-prototype, the holistic definition of product, the ten root causes of failed product efforts, the three company stages (startup, growth, enterprise) and product/market fit. Use when assessing how a product organization works, diagnosing why product efforts fail, distinguishing discovery from delivery, clarifying MVP confusion, planning around a company's stage, or grounding product decisions in the modern product operating model rather than waterfall/roadmap-driven processes.
---

# Applying the Product Operating Model

This skill captures the foundational mindset from *INSPIRED* (Marty Cagan) for building **technology-powered products**. Use it to evaluate how a team works, diagnose failure, and frame product decisions correctly before getting into specific techniques.

## Scope: technology-powered products

This model applies specifically to products **powered by technology** — e-commerce sites and marketplaces (Netflix, Airbnb, Etsy), social media (Facebook, LinkedIn), business services (Salesforce, Workday), consumer devices (Apple, Sonos, Tesla), and mobile apps (Uber, Audible, Instagram). These can blend online and offline experiences (finding a ride, getting a home loan, sending a package). The guidance does not guarantee fit for non-tech products like consumer packaged goods.

## The three overarching principles

Strong teams have moved beyond how most teams practice Lean and Agile, keeping their *core principles* but raising the bar. Three principles run through everything:

1. **Risks are tackled up front, not at the end.** Address the four product risks *before* deciding to build anything — the opposite of waterfall, where all risk lands at the end.
2. **Products are defined and designed collaboratively, not sequentially.** Product, design, and engineering work side by side in give-and-take, rather than PM writes requirements → designer designs → engineering implements, each living with the prior person's constraints.
3. **It's about solving problems (outcomes), not implementing features (output).** A shipped solution is not enough; it must actually solve the underlying problem and produce business results.

Keep these three principles front and center in every product decision.

## The four product risks (+ ethics)

Every product idea carries four risks that must be addressed before building:

- **Value risk** — will customers buy it or choose to use it?
- **Usability risk** — can users figure out how to use it?
- **Feasibility risk** — can our engineers build what we need with the time, skills, and technology we have?
- **Business viability risk** — does the solution also work for the business (sales, marketing, finance, legal, etc.)?

A fifth dimension, **ethics**, asks whether we *should* build it — is this good for our customers and for the world? Tackle these risks during discovery, not after launch.

## Continuous discovery and delivery

There are two essential, ongoing, parallel activities on every cross-functional product team:

- **Product discovery** — figuring out *what product to build*. Intense collaboration between product management, UX design, and engineering, tackling the four risks before a single line of production code is written. The output is a **validated product backlog**. Discovery answers four questions: (1) Will the user buy/use this? (2) Can the user figure out how to use it? (3) Can our engineers build it? (4) Can our stakeholders support it?
- **Product delivery** — building and shipping **production-quality products** customers can run their business on and the company can sell and support. Requires the necessary scale, performance, reliability, fault tolerance, security, privacy, internationalization, and localization.

Both run in parallel and continuously. PMs and designers focus on discovery but help daily with delivery (clarifying intended behavior); engineers focus on delivery but help daily with discovery — and **engineers are typically the single best source of innovation**, so their early participation matters greatly.

### Prototypes and experiments

Discovery runs a series of fast, cheap experiments using **prototypes**, not products. There are several prototype types for different risks, but all require at least an order of magnitude less time and effort than building a product. Strong teams test **10–20 or more product ideas per week**. A prototype is for learning fast and cheap — it is never something the company would sell or stand behind.

## MVP is a prototype, not a product

The MVP concept causes enormous confusion. The critical point: **an MVP should be a prototype, not an actual product.** Building a real, product-quality deliverable just to learn — even one with minimal functionality — wastes months when the same learning was available in days or hours. That waste is the antithesis of Lean. It also confuses and embarrasses sales and marketing when they're asked to sell something half-baked.

When you hear "MVP," steer the team toward a **prototype** for learning during discovery, reserving **product** for what delivery actually ships.

## The holistic definition of product

"Product" means far more than features. A holistic product includes:

- the **functionality** (features),
- the **technology** that enables it,
- the **user experience design** that presents it,
- how we **monetize** it,
- how we **acquire users and customers**, and
- the **offline experiences** essential to delivering value.

Example: for an e-commerce business, "product" includes fulfillment and returns experiences — everything *except* the merchandise being sold. For a media company, everything except the content. Never reduce "product" to just implementing features.

## Key concept chain

We use **prototypes** to run rapid experiments in **discovery**, then in **delivery** we build and release **products** to achieve **product/market fit** — the smallest possible actual product that meets the needs of a specific market of customers (Marc Andreessen's concept) — on the way to delivering the company's **product vision** (the 2–10 year objective, how product delivers on the company's mission).

## The ten root causes of failed product efforts

Most companies still run a fundamentally **waterfall** process — ideas → business case → roadmap → requirements → design → engineering (where Agile finally appears, for delivery only) → QA → deploy — and then complain about lack of innovation and slow time to market. This produces ten serious failure causes (any one can derail a team). See `root-causes-and-stages.md` for the full breakdown; the headline issues:

1. Ideas come from sales/stakeholders, not the best sources; teams are mercenaries, not empowered.
2. Business cases at this stage are fiction — we *can't* know revenue or cost yet.
3. Roadmaps ignore the **two inconvenient truths**: at least half our ideas won't work, and the ones that do need several iterations to deliver value (*time to money*).
4. "Product management" is really project management — gathering and documenting requirements.
5. Design enters too late — "lipstick on the pig."
6. Engineering enters too late, wasting its innovation potential.
7. Agile's real benefits arrive too late (~20% of its value).
8. Process is project-centric: **projects are output, product is outcome** → orphaned projects.
9. Customer validation happens too late — the fatal waterfall flaw and the biggest form of waste.
10. **Opportunity cost** — the biggest loss is what the org could have done instead.

## Company stages

Three stages, each with distinct challenges (full detail in `root-causes-and-stages.md`):

- **Startup** — a new product company that has **not yet achieved product/market fit**. Racing to find a viable product before money runs out; success usually comes from being great at product discovery.
- **Growth-stage** — has product/market fit and must scale: replicate success with adjacent products, grow the core, while organizational stress (technical debt, lost big-picture, struggling empowerment) appears everywhere.
- **Enterprise** — must sustain **consistent product innovation** (create new value, not just capture/optimize existing value) or enter a slow death spiral driven by stakeholders protecting what exists.

## How to apply this skill

When evaluating or advising a product effort:

1. Identify the **company stage** and its characteristic challenges.
2. Check whether the team is running the **waterfall** anti-pattern; map symptoms to the ten root causes.
3. Confirm the four **product risks** are being tackled **up front** in discovery, not at the end.
4. Verify discovery uses **prototypes/experiments**, and that any "MVP" is a prototype, not a shipped product.
5. Confirm product is treated **holistically** and the team optimizes for **outcomes**, not feature output.
6. Anchor recommendations in the three overarching principles.
