# Root Causes and Company Stages — Detail

Supporting reference for `SKILL.md`. Use this for the full breakdown of the ten root causes of failed product efforts and the three company stages.

## The common (waterfall) process that causes failure

Most companies of every size still build products this way:

1. **Ideas** originate inside (executives, stakeholders, business owners) or outside (current/prospective customers).
2. **Roadmap** — leaders prioritize ideas so the most important work goes first and they can predict delivery dates.
3. **Quarterly/annual planning** — leaders negotiate the roadmap, requiring a **business case** per item: (1) how much money/value will it make? (2) how much money/time will it cost?
4. **Requirements** — once an idea reaches the top, a PM talks to stakeholders to flesh it out into requirements (user stories or a functional spec) to communicate to designers and engineers.
5. **UX design** — the design team provides interaction, visual, and (for devices) industrial design.
6. **Engineering** — requirements and design specs reach engineers; this is where Agile finally enters. Work is broken into sprints (1–3 to build the idea).
7. **QA testing** — verify it works and introduces no regressions.
8. **Deploy** — release to customers.

Almost everyone claims to be Agile, but this is fundamentally a **waterfall** process; engineers do as much Agile as the broader waterfall context allows.

## The ten root causes (full)

1. **Source of ideas.** Leads to sales-driven specials and stakeholder-driven products — not the source of the best product ideas. Teams become disempowered mercenaries who only implement.

2. **The business-case fatal flaw.** Business cases are fine for large investments, but at this stage the two key inputs are unknowable: revenue depends entirely on how good the solution turns out to be (many ideas make literally *nothing*, proven by A/B testing), and cost is extremely hard for engineering to estimate without a real solution (hence the t-shirt-sizing compromise). A core product lesson is **knowing what we can't know.** Companies still play the business-case game to get a prioritized roadmap.

3. **Roadmaps and the two inconvenient truths.** Roadmaps are usually prioritized lists of features and projects. But:
   - **First truth:** at least **half of our ideas won't work** — customers aren't as excited as we are, the product is too complicated to be worth it, or it's far more expensive to build than expected. (The best teams assume *three quarters* won't perform as hoped.)
   - **Second truth:** even ideas with potential typically need **several iterations** to deliver the necessary business value — *time to money*.

   There's no escaping these truths no matter how smart you are; the difference is how you deal with them.

4. **Role of product management.** In this model it's really **project management** — gathering and documenting requirements for engineers. This is 180 degrees from modern tech product management.

5. **Role of design.** Too late to get design's real value — the **"lipstick on the pig"** model. The damage is done; designers can only make the mess look as nice and consistent as possible.

6. **Engineering brought in too late.** The biggest missed opportunity. Using engineers only to code captures about half their value — **engineers are typically the single best source of innovation**, yet aren't invited to the party.

7. **Agile too late.** Principles and benefits of Agile arrive far too late. Teams get maybe **20%** of Agile's value — Agile for delivery only, while the rest of the organization is anything but Agile.

8. **Project-centric.** Companies fund, staff, push, and launch **projects**. But **projects are output and product is about outcome.** This predictably produces **orphaned projects** — something shipped that doesn't meet its objectives.

9. **Customer validation too late.** The biggest flaw of waterfall: all risk is at the end. A core Lean principle is reducing waste, and the biggest waste is designing, building, testing, and deploying something only to discover it wasn't needed. Many teams *believe* they're doing Lean while trying out ideas in one of the slowest, most expensive ways possible.

10. **Opportunity cost.** While wasting time and money on this process, the biggest loss is what the organization **could and should have done instead** — time and money you can't get back.

All ten are very serious; many companies suffer more than one, or all. The best teams operate nothing like this.

## Beyond Lean and Agile

There is no silver bullet for building products. Lean and Agile **values and principles are here to stay** — they represent meaningful progress — but the common *manifestations* fall short: teams spend months on a so-called MVP and don't know what they have until they've spent real time and money (not Lean), or they over-test everything and go nowhere fast; and most "Agile" is barely Agile. The best teams leverage the core principles while raising the bar (see the three overarching principles in `SKILL.md`).

## The three company stages

In the technology world there are three stages of companies. Each has characteristic challenges.

### Startup — Getting to product/market fit

A **startup** is a new product company that has **not yet achieved product/market fit** — still trying to come up with a product that can power a viable business.

- PM role usually covered by a co-founder; typically fewer than 25 engineers across one to five product teams.
- The reality is a **race to product/market fit before the money runs out.** Nothing else much matters until there's a strong product meeting an initial market's needs, so focus is necessarily on the product.
- Limited early funding to determine whether the company can discover and deliver the needed product; the closer to running out, the more frantic and desperate.
- Good startups are optimized to learn and move fast with little bureaucracy. Failure rates are high; the few that succeed are usually **really good at product discovery.**
- The experience is stressful, exhausting, and risky — but can be amazingly positive and financially rewarding.

### Growth-stage — Scaling to success

Startups skilled and lucky enough to reach product/market fit face the equally hard challenge of **growing and scaling** — a "good problem to have."

- Must replicate earlier success with new, adjacent products and services **while growing the core business as fast as possible.**
- Typically **25 to several hundred engineers** — more help, but signs of organizational stress everywhere.
- Symptoms: product teams don't understand the big picture or how their work contributes to larger goals, and struggle with what it means to be an empowered, autonomous team. Sales/marketing find the original go-to-market doesn't fit new products. Technology infrastructure is bursting at the seams — **technical debt** from every engineer.
- Tough on leaders too: the leadership style and mechanisms that worked as a young startup fail to scale; leaders must change roles and behaviors.
- Strong motivation: pursuit of a public offering or becoming a major business unit, plus real potential for positive world impact.

### Enterprise — Consistent product innovation

Companies that scale and want a lasting business face the toughest challenges still ahead: ensuring **consistent product innovation** — constantly creating *new* value for customers and business (not just **value capture** by tweaking and optimizing existing products), developing each product to its full potential.

- Many enterprise companies enter a **slow death spiral**, living off value and brand created years or decades earlier. The decline is gradual but the end state is all but certain.
- Numerous stakeholders work hard to **protect what exists**, often shutting down new ventures that could re-create the business (but risk the core) or piling up obstacles so few can drive a new direction.
- Symptoms: diminished morale, lack of innovation, much slower delivery to customers. Having largely achieved the original vision, people aren't sure what's next — complaints about lack of vision, lack of empowerment, decisions taking forever, and **design by committee.**
- Leadership is frustrated by the lack of innovation and often resorts to acquisitions or separate **"innovation centers,"** which rarely produce the desired innovation.
- Companies like Adobe, Amazon, Apple, Facebook, Google, and Netflix avoid this fate — others *could* too, but must make big changes.
