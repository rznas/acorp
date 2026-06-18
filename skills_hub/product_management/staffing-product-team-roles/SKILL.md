---
name: staffing-product-team-roles
description: Explains the roles on and around a modern product team from the book INSPIRED by Marty Cagan — what each role owns, how they collaborate with the product manager, their common failure modes, and when an organization actually needs them. Covers individual-contributor and supporting roles (product designer, product marketing manager, user researcher, data analyst, test automation engineer, delivery manager) and leadership roles (head of product/VP, GPM, principal PM, head of technology/CTO, holistic-view leaders). Use when staffing or structuring a product team, writing a role definition or job description, hiring a head of product or CTO, diagnosing why a team is struggling (missing design, missing principal PM, technical debt), clarifying who owns what, or deciding which roles a startup vs. growth-stage vs. enterprise company needs.
---

# Staffing Product Team Roles (INSPIRED)

This skill describes the people on and around a product team, what each owns, how each collaborates with the product manager (PM), the common failure modes, and when a company actually needs the role. It reflects the *modern* (empowered product team) model, not the old waterfall model where product marketing defined the product and PMs only shepherded delivery.

## How to use this skill

1. Identify the question: defining a role, hiring, diagnosing a struggling team, or sizing staffing for company stage.
2. For individual-contributor and supporting roles, see `individual-contributor-roles.md`.
3. For leadership roles, see `leadership-roles.md`.
4. Apply the staffing-by-stage guidance and failure-mode diagnostics below.

## Core principle

Modern product teams are cross-functional and durable. Designers, engineers, and (part-time) product marketing are full collaboration partners *from discovery through delivery* — they are not downstream service providers who receive specs. Roles are measured on the **success of the product**, not the output of their function.

## Roles at a glance

| Role | Dedicated to team? | Owns | Reports to (typical) |
|---|---|---|---|
| Product Manager | Yes | Value & viability of the product | Head of Product |
| Product Designer | Yes | Holistic user experience; discovery partner | Head of Design |
| Engineers / Tech Lead | Yes | Feasibility, build & run | Head of Technology |
| Product Marketing Manager | Part-time (spread across teams) | Positioning, messaging, go-to-market | VP Marketing |
| User Researcher | Shared | Qualitative learning (generative + evaluative) | Head of Design / Research |
| Data Analyst (BI) | Shared (sometimes dedicated) | Quantitative learning & analytics | Data / Product |
| Test Automation Engineer | Shared (sometimes dedicated) | Automated tests, release confidence | Engineering |
| Delivery Manager (Scrum Master) | Shared | Removing obstacles / impediments | Engineering / Product |
| Head of Product (VP) | — | Team, vision/strategy, execution, culture | CEO |
| Head of Technology (CTO) | — | Org, leadership, delivery, architecture, discovery, evangelism | CEO |

## Staffing by company stage

- **Startup (1–2 teams):** You will likely have *none* of the supporting roles (researcher, data analyst, test engineer, delivery manager, principal PM) and the PM/designer absorb them. This is fine and even has advantages. Holistic view fits in everyone's heads.
- **Growth stage (~5–10 teams):** Delivery managers become important (PMs are drowning in project management). GPMs help coordinate tightly-coupled team groups. Product marketing partners become essential.
- **Enterprise / large complex systems:** Holistic-view leaders (head of product or principal PM, head of design or principal designer, CTO/architects) become critical — their *absence* is obvious in the product.

## Failure-mode diagnostics

Use these symptoms to spot a missing or misused role:

- Product looks like it was built by several different agencies, inconsistent UX → **missing head of design or principal designer**.
- PMs don't understand the implications of their decisions; constantly ask devs to read the code to explain how the system works → **missing principal product manager**.
- Software is spaghetti; even small changes take forever → **unmanaged technical debt** (a CTO failure).
- PM is doing wireframes/visual design themselves, or engineers invent the design from user stories → **absence of product design** (see IC roles file).
- PM spends the whole day enabling the sales force with no time to figure out the product → **missing product marketing partner**.
- PM has no time for the core job (a product worth building) because of project-management churn → **missing delivery manager**.
- Engineers used only to write code, not in discovery → **CTO not enforcing the Discovery responsibility**; you get a fraction of their value.
- Technology org reports to a **CIO** rather than acting as a CTO org → warning flag for failed-product pathologies.

## Reference files

- `individual-contributor-roles.md` — Product Designer, Product Marketing Manager, and the supporting roles (User Researcher, Data Analyst, Test Automation Engineer, Delivery Manager).
- `leadership-roles.md` — Holistic-view leadership, Head of Product/VP (four competencies, GPM, principal PM), Head of Technology/CTO (six responsibilities).
