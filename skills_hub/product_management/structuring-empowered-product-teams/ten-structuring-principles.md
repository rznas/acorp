# The Ten Principles of Structuring Product Teams

From *INSPIRED* Chapter 20. At scale, splitting your product across many teams (25, 50, 100+) is one of the hardest problems and a major factor in speed, empowerment, and meaningful accountability. There is **no one right answer** and **no recipe** — understand these principles, weigh the options for your circumstances, debate the alternatives, and decide. Every structure optimizes for some things at the expense of others.

## 1. Alignment with investment strategy

Many companies' teams are simply reflections of past investments — they have certain teams because they always have. You must invest in the future: phase out products that no longer carry their weight, reduce investment in cash cows to fund future growth. Use a model (e.g., three horizons, or portfolio management). Your team structure should reflect your investment strategy.

## 2. Minimize dependencies

Minimizing dependencies helps teams move faster and feel more autonomous. You can never fully eliminate them, but you can continuously reduce them. Dependencies change over time — **track them continuously** and keep asking how to reduce them.

## 3. Ownership and autonomy

We want missionaries, not mercenaries — which leads directly to ownership and autonomy. A team should feel empowered yet accountable for a **significant** part of the offering. This is harder than it sounds: large systems don't slice cleanly, and interdependencies always chip away at the sense of ownership. Work hard to maximize it.

## 4. Maximize leverage

As organizations grow, common needs and shared services become more important — for speed and reliability, so teams don't reinvent the wheel. But beware: shared services **create dependencies** and can impinge on autonomy.

## 5. Product vision and strategy

The product **vision** describes where the organization is trying to go; the product **strategy** describes the major milestones to get there. Many larger/older orgs have lost a relevant vision and strategy — this is key. Once you have them, structure teams to be well positioned to deliver on them.

## 6. Team size

A practical principle:
- **Minimum:** 2 engineers + 1 PM (+ 1 designer if user-facing). Below that is under critical mass.
- **Maximum:** ~10–12 engineers — beyond that, one PM and one designer can't keep them supplied with good work.
- Each product team must have **one, and only one, product manager.**

## 7. Alignment with architecture

For many organizations this is the **primary** structuring principle. Many start with the vision, devise an architectural approach to deliver it, then design teams around that architecture.

Why: **architecture drives technologies, which drive skill sets.** Full-stack teams that work any layer are ideal but often unrealistic — engineers specialize, and architecture doesn't change quickly.

Signs a company ignored architecture when assembling teams:
1. Teams feel like they're constantly **fighting the architecture.**
2. **Interdependencies** between teams seem disproportionate.
3. As a result, things move slowly and teams don't feel empowered.

**Common-services / core-services / platform teams:** at scale it's typical to have teams that provide common services to other product teams. They reflect the architecture and are **very high leverage**. They are, by design, dependencies of all other teams — they exist to *enable* the others — so they're hard to staff. Use strong, highly technical **platform product managers**.

## 8. Alignment with user or customer

Aligning teams to user/customer types has real benefits. In a two-sided marketplace, having some teams focus on buyers and others on sellers lets each go deep with *their* customers rather than learning *all* customers. Even so, such companies still have teams providing the common foundation/shared services (a reflection of architecture) — it's normal and fine to have **both** types of teams.

## 9. Alignment with business

Larger companies often have multiple lines of business on a common foundation. If the technology were truly independent across businesses, treat them as separate companies. Usually it isn't — multiple businesses share an integrated foundation. This resembles aligning by customer type, but the business-unit structure is an **artificial construct** (different units often sell to the same actual customers). So aligning with business units usually comes **after** the other factors in priority.

## 10. Structure is a moving target

The optimal structure changes as the organization's needs change. You won't reorganize every few months, but **review team structure roughly once a year**. There is never a perfect structure — it's always tradeoffs and choices.

---

# Autonomy @ Scale: the autonomy-vs-leverage tradeoff

Most of the benefits of the empowered, dedicated/durable, cross-functional, collaborative model come from increased **motivation and ownership** when teams control their own destiny. Yet teams often complain they don't feel autonomous. The complaints usually fall into two cases:

1. The team isn't **trusted** yet by management (no long rope).
2. The team wants to change something leaders assumed was part of the **foundation**.

Most teams agree some things are wide open and others are part of a **common foundation** all teams share. Example: if engineering standardized on GitHub for source control, that's foundation — the org-wide cost of one team using a different tool would outweigh the benefit. Many cases are far less clear (test automation approach, programming languages, UI frameworks, browser compatibility, expensive features like offline support, flavor of Agile, company-wide initiatives).

It boils down to a tradeoff: the team's **autonomy** vs. **leverage** of a high-leverage foundation. There is no single right answer — it differs per company, per team, and depends on culture. The author loves autonomous teams **and** is a big fan of investing in a high-leverage foundation.

## Factors to weigh

- **Team skill level.** *A team:* experienced, can be trusted to choose well. *B team:* right intentions, may lack experience, needs help. *C team:* junior, may not know what they don't know; needs significant coaching.
- **Importance of speed.** Leverage avoids reinventing the wheel. Sometimes duplication/slowness is an accepted cost of empowerment; sometimes the business depends on leverage.
- **Importance of integration.** Related-but-independent products → integration matters less. Highly integrated products → leverage is critical. Optimize for the particular solution vs. for the company as a whole.
- **Source of innovation.** If future innovation must come from the **foundation**, give teams freedom to revisit core components. If it's at the **solution** level, discourage revisiting the foundation and focus creativity on the application.
- **Company size and locations.** Scale and dispersed teams make leverage more important and harder. Approaches: *center of excellence* (leverage focused by location), stronger holistic roles, or added process.
- **Company culture.** Pushing toward leverage can feel like chipping away at autonomy — acceptable for B/C teams, more problematic for A teams.
- **Maturity of technology.** Don't standardize on a foundation prematurely. Forcing leverage before the foundation is ready builds a house of cards.
- **Importance to business.** With a solid foundation, *not* leveraging it carries risk. For business-critical products it becomes a question of which battles to pick.
- **Level of accountability.** Empowerment must come with accountability. You *want* teams to stress over these tradeoffs. If a strong A team fully understands the consequences and still wants to replace a foundation component, side with the team.

## The two critical pieces of context

Discuss these tradeoffs openly — most teams are reasonable, and a few key questions help them decide well. If teams consistently decide poorly, check their experience level, but most likely they're missing **business context**. Leadership must provide:

1. **The overall product vision.**
2. **The specific business objectives assigned to each team.**

Without these there's a vacuum and real ambiguity over what a team can and can't decide. Note: leadership provides vision and objectives, but says nothing about **how** to solve the problems — that's where the team's autonomy and flexibility live.
