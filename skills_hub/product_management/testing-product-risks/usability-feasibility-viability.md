# Feasibility and Business Viability — Detail

This file expands the feasibility and business-viability sections of SKILL.md. Usability and the value risks are covered in SKILL.md and `value-testing.md`.

---

## Testing Feasibility — full detail

Engineers validating feasibility are answering several related questions:
- Do we know **how** to build this?
- Do we have the **skills** on the team?
- Do we have enough **time**?
- Do we need **architectural** changes?
- Do we have all the **components** on hand?
- Do we understand the **dependencies**?
- Will **performance** be acceptable?
- Will it **scale** to the levels we need?
- Do we have the **infrastructure** to test and run it?
- Can we afford the **cost** to provision it?

Most ideas get a quick "no problem" — the work isn't new and engineers have built similar things before. But some ideas are genuinely hard (e.g., evaluating machine-learning technology, build/buy decisions, judging whether a technology fits the job).

**How to get good feasibility answers:**
- **Never put engineers on the spot.** Demanding instant estimates (time, story points) in a weekly planning meeting almost certainly goes badly — you'll get a conservative answer partly designed to make you go away.
- **Engineers estimate AFTER investigating.** If engineers have followed along while the team tested ideas with customers, they've already been mulling the issues. For anything worthwhile, give them time to investigate and consider.
- **Ask the right question:** not "Can you do this?" but **"What's the best way to do this, and how long would it take?"**
- Engineers may ask to build a **feasibility prototype** to answer one or more questions. If the idea is worth the discovery time, encourage it.
- **Love the items that need investigation.** Reasons: (1) the best ideas are often approaches "only now possible," requiring new tech and time to learn it; (2) given a day or two, engineers often return with good answers *and* better ways to solve the problem; (3) these items motivate the team — a chance to learn and shine.

### Discovery for hardware products
Hardware is everywhere (phones, watches, robotics, cars, medical instruments, thermostats). Most discovery techniques still apply, and prototyping matters even more. Differences and cautions:
- Different engineering skill sets, need for industrial design, longer manufacturing.
- **The cost of a mistake (time and money) is far more severe** — software corrections are cheap; hardware corrections are not.
- More technical feasibility risks AND added business viability risks: a much more sophisticated analysis of parts, manufacturing costs, and forecasting.
- 3D printing has dramatically helped device prototyping.
- Bottom line: tackle value, usability, feasibility, and viability aggressively and **raise your confidence bar before committing to manufacturing.**

---

## Testing Business Viability — full detail

The solution must work for the business, not just for customers and engineers. You need a viable business model (cost to produce, market, and sell must be sufficiently below revenue), to operate within the law, to honor partnership commitments, and to fit the company's brand promise. Doing this well is what separates good PMs from great ones — it's the real meaning of being **"CEO of the product."** It protects the company's revenue, reputation, employees, and customers.

The worst outcome: the team commercializes and delivers a shippable product, then can't ship because it violates a constraint. When that happens, **it's on the product manager.** It is your job to understand each relevant constraint and act within it.

### Stakeholder preview checklist
For each impacted stakeholder, walk them through the proposal **before building anything**, show them prototypes, and work together to address their concerns.

| Stakeholder | Typical concerns / constraints |
|-------------|-------------------------------|
| **Marketing** | Enabling sales; brand and reputation; market competitiveness and differentiation; fit with go-to-market channels. Flag anything that risks the sales channel, major programs, or falls outside the brand promise. |
| **Sales** | Products must fit the sales channel's strengths and limits. Direct sales is expensive → needs a high value/price point. A product needing very different selling skills may be rejected by the sales force. If your proposal departs from what sales has proven it can sell, meet leadership first. |
| **Customer Success** | Align with the company's **high-touch vs. low-touch** model. If you'd change it, discuss options with leadership. (High-touch staff are also great for product insights and prototype testing.) |
| **Finance** | Can you afford to build, sell, and operate it? Business analytics/reporting, investor relations. If costs are involved, model them with finance to prove a viable approach to leadership. |
| **Legal** | Privacy, compliance, intellectual property, competitive issues — especially for market disruptors. Sit down early to surface issues. |
| **Business Development** | Existing partner contracts carry commitments and constraints that can cripple or boost your ability to compete. Understand their impact. |
| **Security** | Normally part of the product team, but called out because it's so critical. For anything remotely security-related, bring your tech lead and meet security leadership early. |
| **CEO/COO/GM** | Responsible for the business unit and aware of all these constraints. If the PM isn't aware of the issues or has no plan, the exec won't trust the PM or team. They quickly tell whether a PM has done their homework. |

Also watch for **stakeholders unique to your business** that aren't on this list but are critical.

Testing viability means ensuring the proposed solution works within each area's constraints, and that impacted stakeholders get to review the proposal and confirm their concerns are addressed.

### Three modes for showing the prototype
Always use a **high-fidelity prototype, never a PowerPoint.** Pick the right mode for the situation:

| Mode | Who drives | Purpose |
|------|-----------|---------|
| **User test** | The user | *Test* usability and value qualitatively on real users/customers. |
| **Product demo** | The product manager | *Sell* / show off the product's value to prospects or evangelize internally. Usually a scripted product-marketing asset; PM may give it to high-value customers or execs. |
| **Walkthrough** | The PM (let the stakeholder play if they want) | Give a stakeholder every chance to *spot a problem*. Not selling, not testing on them, and definitely not hiding anything. |

Common rookie mistakes: doing a walkthrough with a prospective customer when you should have prepared a product demo; or doing a product demo during a user test and then asking the user what they think. Be clear about which mode you're in, and be skilled in all three.
