# Context

The architecture's context is the foundation for every other judgment in this review. Without knowing who the doc is for, why it exists, what the change must achieve, and what bounds the solution, the rest of the review can only make local judgments about the design — not whether it's the right design *for this situation*. Establish context first; only then move deeper.

## Rule of thumb

After reading the doc, you should be able to answer in plain sentences: *who is this for, why now, what must it achieve, and what is it constrained by?* If any of those four are missing, resolve them with the architect before reviewing the design itself.

---

## How to run the analysis (conversational)

Walk the architect through the elements below. Use the artifact for what's already documented; probe directly for what isn't. Don't ask for things the doc clearly covers — confirm and move on.

### 1. Audience and stakeholders
*Who is this document for? Who is affected by the decision?* Separate the two:
- **Audience** — who reads the doc (execs, engineers, security, ops, partners). Drives tone and depth.
- **Stakeholders** — who is affected by the decision (funders, operators, end users, regulators, downstream teams). Drives whose interests must be respected in the design.

Industry and organization context belong here — a doc for a regulated healthcare client carries different defaults than one for an internal data team.

### 2. Purpose and stage of artifact
*Why does this document exist, and at what stage?* Is it a formal specification, an RFC to gather feedback, a lightweight design to motivate a sales conversation, an ADR capturing a single decision? The expected depth and rigor of the design — and of this review — depend on the answer. A napkin sketch and a ready-to-build spec are not held to the same standard (see also `SKILL.md` Principles: "Respect the stage").

### 3. Goals of the change
*What must this change achieve, in business terms?* The doc should state functional scope (what the design does or changes) and objectives expressed as outcomes — ideally OKRs / KPIs with quantified targets. Goals may be technical at the surface but should connect to business value: revenue protected, cost avoided, risk reduced, capability enabled. Flag any goal stated only in technical terms with no business linkage, and any objective without a number.

### 4. System context
*What is the existing landscape, and how does this change fit?* Greenfield is rare. Confirm:
- What systems are being **created, changed, or replaced**, and what coexists unchanged.
- Key **upstream producers** the system depends on.
- Key **downstream consumers** the system serves.
- **Integration patterns** (sync API, async event, batch, file drop).
- For replacements: the **migration / cutover** story and the **rollback** path.

### 5. Constraints
*What bounds the solution space?* These are non-negotiable inputs, not preferences:
- **Regulatory** — compliance regimes that apply (GDPR, HIPAA, SOC 2, PCI-DSS, FedRAMP, data residency).
- **Organizational** — mandated tech stack, approved vendor list, security policies, naming conventions.
- **Budget** — capex / opex ceilings, vendor spend caps.
- **Timeline** — hard deadlines (regulatory cutover, contractual commitment, market window).
- **Team / skills** — what the operating team can realistically build and run.

Constraints often explain *why an obvious option was rejected* — they pair tightly with `references/alternatives.md`.

### 6. Assumptions
*What is the design taking for granted that it doesn't itself control?* Examples: an upstream service's SLO, a vendor's pricing tier, a regulatory interpretation, a user-growth forecast, the availability of a specific team or skill. Each unstated assumption is a hidden risk; surface them so they can be tested.

### 7. Scope boundaries
*What is explicitly in scope, and what is explicitly out?* Out-of-scope items prevent the reviewer from demanding work the doc was never meant to cover. If the architect can't name what's out of scope, the scope is probably broader (and softer) than they realize.

### 8. Timeline and phasing
*When must this happen, and in what order?* A hard 6-week deadline is itself an architectural force. Phased rollouts (MVP → expansion) change which decisions are reversible and which can be deferred. If the doc has no timeline, the design can't be judged for feasibility.

### 9. Decision authority and open questions
*Who can approve this, and what's still unresolved?* Identify:
- Who **must approve** before building (architect-of-record, security, legal).
- Who must be **consulted** or **informed**.
- What the architect already knows is **unresolved** — surface these before the review adds to the list.

---

## Review checks

1. **All four foundations answerable** — Can you state, in plain sentences, who / why / what / constrained-by after reading the doc?
2. **Audience and stakeholders separated** — Both named, not conflated.
3. **Stage and purpose declared** — Is the artifact's maturity and intent stated? Without it, the rest of the review can't be calibrated.
4. **Goals quantified and business-tied** — Are objectives expressed as outcomes with numbers, and tied to business value?
5. **System context concrete** — Are the specific upstream / downstream systems named, not generic categories?
6. **Constraints listed explicitly** — Are regulatory, organizational, budget, timeline, and team constraints called out, not implied?
7. **Assumptions surfaced** — Are the things the design depends on but doesn't control named?
8. **Scope bounded both ways** — Are in-scope *and* out-of-scope items both named?
9. **Timeline stated** — Is there a deadline or phasing plan? "Soon" is not a timeline.
10. **Open questions acknowledged** — Has the architect named what's unresolved, or is everything presented as settled?

## Common failure modes

- **Implicit audience** — Doc reads like it's for "engineers in general" with no specific stakeholder named; tone and depth end up mismatched to who actually reviews it
- **Purpose unstated** — Reader can't tell if this is exploratory, a decision record, or a build spec; review demands get calibrated wrongly
- **Goals stated only technically** — "Migrate to microservices" or "adopt event-driven" with no business outcome attached
- **Goals without numbers** — "Improve performance" or "reduce cost" treated as goals; without targets, the design has nothing to satisfy
- **Greenfield framing of a brownfield problem** — Existing systems and migration burden invisible in the doc
- **Constraints mistaken for preferences** — Regulatory or organizational mandates not called out, so they look optional
- **Constraints used to justify weak design** — The opposite: every problem blamed on "constraints" without specifying which one
- **Assumptions implicit** — Design depends on a vendor's pricing or an upstream SLO that's never stated, so it can't be tested
- **No out-of-scope list** — Scope appears to expand throughout the review as new gaps surface
- **No timeline** — Or worse, a deadline that's incompatible with the design's complexity, but not acknowledged
- **Everything presented as settled** — Architect lists no open questions, so the review surfaces issues that were actually known but suppressed

## Deliverable

Context findings shape the rest of the review. Before producing the main review:

- **Context is solid** — Note as a Strength in `assets/review-template.md` and proceed.
- **Context has gaps but the design is still judgeable** — List gaps under **Concerns** or **Open Questions** and proceed with appropriate caveats.
- **Context is too thin to judge the design** — Stop. Report as a **Blocker** and request the missing context before continuing the review. Do not produce design-level findings against a foundation that isn't there.

For each gap, write it as:

> **{Context element}** — currently **{absent / thin / unclear}**. Needed because **{what downstream review judgment depends on it}**. Suggested resolution: **{specific question to answer or section to add}**.
