---
name: testing-product-risks
description: Provides Marty Cagan's INSPIRED toolkit for validating product ideas in discovery against the four product risks before building. Use when running a usability test, validating value (demand, qualitative, or quantitative), running a fake-door or landing-page demand test, designing a discovery A/B test, asking engineers to assess feasibility, or doing a business viability / stakeholder walkthrough. Triggers include "usability test", "A/B test", "demand test", "validate value", "fake door", "feature parity", "feasibility estimate", "business viability", "stakeholder review", and "will customers actually use/buy this".
---

# Testing Product Risks

Product discovery exists to quickly separate good ideas from bad before committing engineering to build. Every idea carries four risks. This skill is the toolkit for testing each one.

## The four risks

| Risk | Question | Owner |
|------|----------|-------|
| Value | Will the user/customer choose to use or buy this? | Product manager |
| Usability | Can the user figure out how to use this? | Product designer |
| Feasibility | Can we build this? | Engineers |
| Business viability | Does this work for our business? | Product manager + stakeholders |

Value is usually the hardest and most important; if value isn't there, nothing else matters. Most teams test in this order: value (with usability, on the same users at the same time) → feasibility → business viability. Test viability last so you don't stir up the organization until you're confident, and so you can show stakeholders evidence of what worked and why.

Most everyday work is low-risk and the team is confident — proceed to delivery. Discovery testing is for when the answers are NOT clear.

## SELECTOR: risk → technique

| If you need to know... | Risk | Technique | Where |
|------------------------|------|-----------|-------|
| Can users operate the product? | Usability | Usability test (5 users, high-fi prototype, observe-don't-help) | below |
| Do customers even care about this problem? | Value (demand) | Fake-door test, landing-page demand test, enterprise A/B at 1% | value-testing.md |
| Do customers/users love it? Why or why not? | Value (qualitative) | Interview → usability → value; willingness signals | value-testing.md |
| How well does the solution work, measured? | Value (quantitative) | Discovery A/B (99/1), invite-only, customer discovery program, analytics | value-testing.md |
| Can engineers actually build this, and how? | Feasibility | "Best way + how long" after time to investigate; feasibility prototype | below |
| Will this work for marketing/sales/legal/finance/CEO? | Business viability | Stakeholder walkthrough checklist; 3 showing modes | usability-feasibility-viability.md |

For the full value-testing toolkit (demand, qualitative, quantitative) read `value-testing.md`. For the detailed feasibility advice and the full business-viability stakeholder checklist read `usability-feasibility-viability.md`.

---

## Testing Usability

Usability testing is the most mature, straightforward discovery technique. The key shift: do it in discovery with prototypes, BEFORE building — not at the end, when fixing issues means waste. You don't need a trained researcher or a lab; you can do this yourself and still catch the serious friction points.

### Recruit users to test
- Use your customer discovery program (works well for B2B; supplement for consumer).
- Advertise on Craigslist or run a Google AdWords SEM campaign (great for catching users *in the moment* of needing a product like yours).
- Select from your user email list (product marketing can help narrow it).
- Solicit volunteers on your company website — then call and screen them against your target market.
- Go where users congregate: trade shows, shopping centers, sports bars. Bring thank-you gifts.
- Meet at a mutually convenient spot like Starbucks ("Starbucks testing"). Compensate them if they come to you.

### Prepare the test
- Use a **high-fidelity user prototype**. Low/medium-fi gives some usability feedback, but the value test that follows needs realism.
- Attendees: the product manager and product designer must be at every test. Bring one engineer too (rotate them) — the magic often happens when an engineer watches.
- Define the **set of primary tasks** to test in advance (the things users do most), not obscure edge cases.
- Run the test fast, before the PM/designer fall in love with their own ideas. Good PMs know they'll get it wrong first and that learning from tests is the fastest path to success.
- One person administers, another takes notes; debrief together afterward to confirm you saw the same things.
- A Starbucks table beats a formal lab in many ways — the user feels less like a lab rat. The customer's own office is also excellent (all the cues are there). Remote tools are a supplement, not a replacement, because they don't support the value test that follows.

### Run the test
1. **Interview first.** Before tasks, learn whether the user really has the problem you think, how they solve it today, and what it would take to switch.
2. **Frame it as "test not of you."** Tell them it's an early prototype, it's not real, they can't hurt your feelings. You're testing the prototype, not them — only the prototype can pass or fail.
3. **Observe the first impression / landing page.** Before they dive into tasks, see if they can tell from the landing page what the product does and why it's valuable. You lose this first-time context once tasks start. Landing pages are critical to bridging expectations and reality.
4. **Keep users in "use mode," not "critique mode."** What matters is whether they can do the tasks, not whether they think something is ugly. Never ask "what three things would you change?" Watch what they do, not what they say.
5. **Don't help. Stay quiet.** Suppress the urge to rescue them. Become a horrible conversationalist; get comfortable with silence. Note three outcomes: (1) sailed through unaided, (2) struggled but eventually got there, (3) gave up (truly would leave for a competitor).
6. **Don't lead the witness.** If they're scrolling and hunting, you may ask what they're looking for — that's valuable. Avoid running narration; it pushes people into critique mode.
7. **Parrot technique.** Reflect their actions/questions back instead of answering or judging:
   - Silence: "I see you're looking at the list on the right." → prompts them to explain.
   - Their question "Will clicking this make a new entry?" → "You're wondering if clicking this will make a new entry?" → they answer themselves.
   - Avoid value judgments: instead of "Great!" say "You created a new entry."
   - Parroting also buys your note-taker time.
8. **Watch body language and tone.** It's painfully obvious when they don't like it and clear when they genuinely do. If they like it they'll ask for an email at launch; if they love it they'll try to get it early.

### Summarize the learning
As soon as you spot an issue (nomenclature, flow, visual design, mental model), **fix it in the prototype immediately** — there's no rule the test must stay identical across subjects. This is qualitative learning, not proof. After each subject/set, the PM or designer writes a **short summary email** of key learnings to the team. No big reports — they take long to write, are seldom read, and are obsolete before delivery.

---

## Testing Feasibility

Once value and usability look good, review the approach with engineers. They're answering: do we know how to build it, do we have the skills/time, do we need architectural changes, do we have the components, do we understand the dependencies, will performance and scale be acceptable, do we have the test/run infrastructure, can we afford to provision it?

Most ideas get a quick "no problem." But some (e.g., evaluating machine-learning tech, build/buy decisions) are genuinely hard.

**Key rules:**
- **Don't put engineers on the spot.** Throwing ideas at them in a weekly planning meeting and demanding instant estimates produces conservative answers designed to make you go away.
- **Let them follow along** as the team tests ideas with customers, so they've been chewing on the issues already. Then give them time to investigate.
- **Ask the right question.** Not "Can you do this?" but **"What's the best way to do this, and how long would it take?"**
- Engineers may ask to build a **feasibility prototype**. If the idea is worth the discovery time, encourage it.
- **Love the "needs investigation" items.** The best ideas are often "only now possible" with new tech. Given a day or two, engineers come back with answers *and* better ways to solve the problem — and these items motivate the team to learn and shine.
- **Hardware caution.** With hardware, the cost of a mistake (time and money) is far more severe — you can't cheaply patch it. There are more feasibility risks and added viability risks (parts, manufacturing cost, forecasting). 3D printing helps prototyping. Tackle all four risks aggressively and **raise your confidence bar before committing to manufacturing.**

---

## Testing Business Viability (summary)

The solution must work for the business, not just for customers and engineers. The PM walks each impacted stakeholder through the proposal — using a **high-fidelity prototype, never a PowerPoint** — *before* building. Stakeholders: marketing, sales, customer success, finance, legal, business development, security, CEO/COO/GM. Plus any stakeholder unique to your business.

Use the **right showing mode**: a **user test** (let the user drive; test usability/value), a **product demo** (you drive; sell/show off value), or a **walkthrough** (you drive; help the stakeholder spot every possible concern — hide nothing).

Full per-stakeholder concerns checklist and the three modes are in `usability-feasibility-viability.md`.
