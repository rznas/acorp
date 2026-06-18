# Adoption Maturity Quick Guide

How to assess and advance Power BI / Fabric adoption. Grounded in the Fabric adoption-roadmap maturity-levels and governance docs and the center-of-excellence-establish doc. Cross-references Kimball `program-management` for the program/sponsorship framing.

## Contents
- Three inter-related adoption types
- Organizational adoption maturity (levels 100–500)
- User adoption stages (1–4)
- Solution adoption phases (1–4)
- Center of Excellence (COE) model
- Governance maturity self-check
- How to advance (key actions)

---

## Three inter-related adoption types

Assess each **per goal**, not as a single tenant-wide score. The three reinforce each other:

| Type | Measures | Influences |
|------|----------|-----------|
| **Organizational adoption** | Effectiveness of governance and data-management practices | Shapes user adoption via mentoring, training, support, community |
| **User adoption** | Whether consumers/creators actively and *effectively* use the tools | Drives organizational decisions and solution success |
| **Solution adoption** | Business value/impact of a deployed solution (reports, app, semantic model, lakehouse) | A well-built solution guides better user behavior |

Usage statistics alone do **not** prove adoption — adoption is about using the tool *effectively*.

## Organizational adoption maturity (Capability Maturity Model levels)

Target at least **level 300–400**. Different business units mature at different rates.

| Level | Name | Characteristics |
|-------|------|-----------------|
| **100** | Initial | Pockets of success; quick wins; organic growth with no coordinated strategy; undocumented, tribal-knowledge practices; risk from lack of awareness of data usage. |
| **200** | Repeatable | Some content now critical; siloed, reactive attempts to document practices; over-reliance on individuals' good habits; uncontrolled organic growth; an internal community channel exists; recognition that a COE adds value. |
| **300** | Defined | Standardized processes consistently applied across boundaries; **COE established** with clear goals; community of practice gains traction; champions emerge; initial training/docs/templates; initial governance model; active executive sponsor; roles/responsibilities understood. |
| **400** | Capable | Analytics delivers significant value; approved tools used for critical content; accepted governance model with cooperation from key BUs; training/docs readily used; standardized oversight/monitoring; COE represents all key BUs; active champions network. |
| **500** | Efficient | Fabric widely accepted; analytics skills highly valued; self-sustaining community (not over-reliant on individuals); COE reviews KPIs regularly; continuous improvement and automation. |

## User adoption stages

Individuals progress at their own pace; measured in aggregate. The slowest leap is **momentum → proficiency**.

1. **Awareness** — heard of / initially exposed; may have access but isn't actively using.
2. **Understanding** — sees the benefits; starts using the tools.
3. **Momentum** — actively gains skills (training, self-learning, experimentation); basic competency for their role.
4. **Proficiency** — uses analytics regularly and as intended; aligns behavior with governance; becomes an advocate; continually improves skills.

By momentum/proficiency the organization must be **ready to support** users (community of practice, user support).

## Solution adoption phases

Scope = one set of requirements (a report set, app, lakehouse, semantic model). Higher phases raise operational expectations.

1. **Exploration** — experimentation / proof of concept; small test group; may live in Desktop, Excel, or a single workspace.
2. **Functional** — meets basic requirements; deployed to the service; supporting components (e.g., a gateway for scheduled refresh) in place; target users aware; possibly a limited preview.
3. **Valuable** — users find tangible benefit; promoted to a **managed, secured, audited production workspace**; validated for data quality/accessibility/performance; **endorsed** when appropriate; usage metrics monitored; feedback loops; documentation; clear ownership/SMEs; branding/theming aligned to governance.
4. **Essential** — routinely used and considered essential for decisions; lives in a **production workspace separated from dev/test** with controlled change/release management; success and support expectations clear (SLAs); aligned with governance; **most content certified**; formal UAT for changes (especially IT-managed).

## Center of Excellence (COE) model

A COE is a team that establishes and maintains the BI platform and a single source of truth with consistent company-wide metrics — **not a help desk**.

- **Core + satellite**: large orgs run a **core COE** plus department-level **satellite COEs**. Satellites "pay to play" (fund the core) and extend core conformed dimensions/facts/business logic with local, department-specific models and KPIs — never disconnected or isolated. (This conformed-core / local-extension pattern mirrors Kimball conformed dimensions; see `program-management`.)
- **Shared capabilities** (fund-the-platform): core platform engineering, infrastructure/hosting, support & operations, release management.
- **Dedicated deliveries**: a delivery team per stakeholder group (data engineer, analytics engineer, technical PM).
- **Governance & compliance**: PM leads provide cross-program oversight; a steering committee meets regularly to review progress and approve major initiatives.
- **Grow the community**: regular **office hours**, a Teams channel for Q&A, informal user groups, and formal training (for example, Dashboard in a Day).

## Governance maturity self-check

| Level | State of governance |
|-------|---------------------|
| 100 Initial | No governance planning; reliant on individual judgment; tribal knowledge. |
| 200 Repeatable | Some BUs standardize/document; an initial governance approach exists; incremental progress. |
| 300 Defined | Complete strategy with focus/objectives/priorities, broadly communicated; policies for top priorities consistently followed; roles/responsibilities documented. |
| 400 Capable | Priorities aligned to business objectives and reassessed regularly; processes to customize policies or handle exceptions; activity-log/API data actively analyzed; proactive action taken. |
| 500 Efficient | Regular KPI/OKR reviews of governance goals; agility and continual improvement are top COE priorities; activity-log/API data informs adoption and governance. |

Introduce governance via **method 3 (iterative governance with staged rollouts)** for most organizations — it balances agility and control. (Method 1 = roll out first then govern; method 2 = full governance planning first.)

## How to advance (key actions)

- **Align goals**: document and communicate data-culture goals; ensure new policies align.
- **Understand current state**: workshops to assess adoption, data culture, and BI practices; document strengths to scale out and gaps to fix.
- **Prioritize one thing**: pick a high-impact, feasible pain point; choose a low-friction first policy users will support.
- **Use checks and balances**: operational (business units) → tactical (COE/supporting teams; audit & compliance) → strategic (executive sponsor & steering committee).
- **Define roles**: CDO/CAO, governance board, governance team, change-management board, executive sponsor, COE, champions, Fabric admins, data stewards — name backups for key roles.
- **Measure**: set measurable KPIs/OKRs; re-review policies on a schedule; track weekly/monthly progress.
- **Audit with data**: use the activity log + tenant inventory to verify whether users follow governance (see [auditing-monitoring-sources.md](auditing-monitoring-sources.md)).
- **Avoid complacency**: documentation alone creates a false sense of control — pair it with active COE community engagement and monitoring.
