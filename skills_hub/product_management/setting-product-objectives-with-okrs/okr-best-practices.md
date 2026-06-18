# OKR Best Practices (detailed)

Detailed practice list and checklists for setting product objectives with OKRs,
drawn from INSPIRED (Marty Cagan), chapters 22-23 and 28-30. Use alongside
`SKILL.md`.

## The 12 critical points for product orgs

1. **Objectives qualitative, key results quantitative.** The objective states the
   problem/aspiration in words; key results state measurable targets.
2. **Key results measure business results, not output or tasks.** Measure the
   outcome (e.g., onboarding time, daily active users, customer acquisition cost),
   never the activity (e.g., "ship the wizard," "re-platform").
3. **Focus on org + product-team OKRs.** Don't let personal objectives or
   functional-team objectives dilute or confuse the focus. Team OKRs roll up to org OKRs.
4. **Find a cadence.** Typically annual for the organization's objectives,
   quarterly for each team's objectives.
5. **Keep counts small.** 1-3 objectives per team/org, each with 1-3 key results.
6. **Track active progress.** Each team checks progress against objectives,
   typically weekly.
7. **Cover what matters, not everything.** Objectives needn't enumerate every task,
   but must cover what the team *needs to accomplish*.
8. **Accountability.** Teams must feel accountable for their objectives. On
   substantial failure, hold a post-mortem/retrospective with peers or management.
9. **Agree on scoring org-wide.** Consistency matters more than the exact method so
   teams know when they can depend on one another (see scoring below).
10. **Flag high-integrity commitments distinctly.** Most KRs target ~0.7; a
    high-integrity commitment is binary — delivered as promised or not.
11. **Be transparent.** Across product and technology, make each team's objectives
    and current progress visible.
12. **Clear ownership.**
    - Senior management (CEO + exec team): organization's objectives and key results.
    - Heads of product and technology: product-team objectives, and ensuring they
      deliver the org's objectives.
    - Individual product teams: propose the key results for each assigned objective.
    - Expect a quarterly give-and-take to finalize team and org OKRs.

## Scoring scale (0 to 1.0)

| Score | Meaning |
|-------|---------|
| 0.0   | Essentially no progress |
| 0.3   | Bare minimum — what you knew you could achieve |
| 0.7   | More than minimum; really achieved what you'd hoped (normal-KR target) |
| 1.0   | Truly exceptional, beyond anyone's hopes |

High-integrity commitments are scored differently — treat them as binary.

## Worked example

- **Objective:** Dramatically reduce the time for a new customer to go live.
- **Key result:** Average new-customer onboarding time < 3 hours (down from 30 days).

The KR is a business outcome (time), not output. The team chooses how to get there.

## Product-team level vs functional level

**Rule: focus OKRs at the cross-functional product team level; cascade UP to the
company/business-unit level.**

A cross-functional product team = product manager + product designer + a few
engineers (sometimes data analyst, user researcher, test automation engineer),
owning a significant product/technology area.

Anti-pattern: each functional department (design, engineering, QA) writes its own
OKRs. Members of those departments *are* the product team members, so they receive
conflicting direction → confusion, frustration, poor business results.

Acceptable:
- Functional managers (heads of UX/engineering/QA/product) may hold org-level
  objectives — they don't sit on a product team, so they aren't conflicted.
- Individual contributors may have a few personal growth objectives if they don't
  interfere with their primary product-team duty.
- Large functional initiatives (responsive design, technical debt, test automation)
  are prioritized at leadership level and folded into the relevant team's objectives.

## Scale checklist (growth-stage / enterprise)

- [ ] Define clear **organization-level objectives** and decide which teams pursue each.
- [ ] Identify **platform / shared-services teams**; coordinate their objectives and
      dependencies (they serve customers indirectly).
- [ ] Run the **reconciliation** process — find gaps across proposed key results and
      adjust (enlist teams, re-prioritize).
- [ ] Make objectives and progress **transparent** (tooling + management connecting dots).
- [ ] Actively **manage commitments and dependencies** — more high-integrity
      commitments at scale; delivery managers track them.
- [ ] For multi-BU enterprises, maintain **corporate-level and business-unit-level
      OKRs**; teams roll up into those.

## High-integrity commitments checklist

- [ ] Minimize the number of date/deliverable commitments.
- [ ] Make commitments **after** product discovery, never before.
- [ ] Validate value + usability (customers), feasibility (engineers), viability
      (stakeholders) before committing.
- [ ] Involve delivery managers to set realistic dates (account for team availability
      and dependencies).
- [ ] Once committed, ensure the org can depend on delivery.
