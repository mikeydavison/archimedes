# Completeness Assessment

Completeness in this skill means **ASR coverage**: for every architecturally significant requirement, does the design contain explicit decisions that address it — and does it avoid decisions that address nothing? This is a traceability check between requirements (`references/asr.md`) and the design. It complements `references/correctness.md` (are the design's claims actually true?) and `references/optimality.md` (are there better alternatives that don't worsen the design?).

This reference is intentionally *not* a generic document-structure checklist. Whether the doc has the right sections is a context concern (`references/context.md`); whether the doc has the right *answers* is what completeness measures.

## Rule of thumb

Walk the ASR list, then walk the design. Every ASR should map to a stated decision; every meaningful decision should map back to an ASR. Either direction failing is a finding. If an ASR is "addressed" without a number or without a verification path, treat it as partial — not complete.

---

## How to run the analysis (conversational)

### 1. Establish the ASR list
Pull the prioritized ASR list from the Requirements stage (driven by `references/asr.md`). If priorities aren't set, fix that first — completeness without priority weighting will treat a missed top-tier NFR the same as a missed nice-to-have. Confirm with the architect that the list is the one to assess against.

### 2. For each ASR, find the decision(s) that address it
Walk every ASR and locate the architectural decision(s) in the doc that address it. Acceptable evidence:
- **Explicit textual mapping** — *"to meet the 99.95% availability target, we deploy across three AZs with automated failover"*
- **Diagrammatic indication** — redundant components shown for an availability ASR; encryption boundaries shown for a security ASR
- **Measurable design parameter** — replication factor for durability, pool size for concurrency, retention window for compliance

If you can only find an *implicit* link ("we use Postgres, so we get ACID"), record it as implicit — not explicit — even when it happens to be true.

### 3. Classify each ASR
Use these levels with explicit criteria:

| Level | Criteria |
|---|---|
| **Fully satisfied** | Explicit decision(s) address the ASR; quantification matches the target; verification path is stated |
| **Partially satisfied** | Decision addresses part of the ASR, *or* addresses it qualitatively without numbers, *or* lacks a verification path |
| **Not satisfied** | ASR is named but no decision in the design addresses it |
| **Implicit** | Design likely satisfies the ASR but doesn't architect for it explicitly; satisfaction is incidental to a tool choice |
| **Deferred** | ASR explicitly punted to a later phase; acceptable when acknowledged with phasing, a finding when hidden |

For NFR-type ASRs, "addressed" without a number and without a verification path is partial at best. This closes the loop on the "Verification path" review check in `asr.md`.

### 4. Trace ASRs as paths, not just at components
Some ASRs (latency, availability, cost) are end-to-end properties. Verify the ASR is satisfied along the whole request path, not just at one component. A design where every hop meets its slice of a 200ms latency budget but the chained total is 400ms is a cumulative failure — common and easy to miss.

### 5. Run the inverse check: decisions without ASRs
Walk the design's notable decisions and ask *which ASR justifies this?* For each decision with no clear ASR backing:
- It may be **gold-plating** — extra capability the requirements don't ask for. Flag and ask whether to remove.
- It may be **evidence of a missing ASR** — the architect knew a requirement implicitly but never stated it. Surface it and add to the ASR list.
- It may be **vendor / tool inertia** — the decision exists because the team uses this stack everywhere, not because the requirements demand it.

Decisions without ASRs are completeness gaps in the other direction; treat them the same as ASRs without decisions.

### 6. Weight by priority and report
Severity comes from the ASR's priority, not from the satisfaction gap alone. A top-priority ASR partially satisfied is usually a Blocker; the same gap on a nice-to-have is a Concern. Order findings so the architect sees high-priority gaps first.

---

## Review checks

1. **ASR list is the source of truth** — Working from a prioritized list, not from whatever the doc happens to mention?
2. **Every ASR mapped** — Each ASR has at least one decision (or an explicit "not satisfied / deferred" verdict) — no silent omissions?
3. **Mapping evidence is explicit** — Textual or diagrammatic linkage, not inferred satisfaction?
4. **Quantification matches the target** — Decisions for numeric ASRs include numbers that meet or exceed the target?
5. **Verification path stated** — For each satisfied ASR, the doc says how compliance will be measured post-build (load test, chaos test, audit, synthetic probe)?
6. **End-to-end traced** — System-level ASRs validated along the full path, not just at individual components?
7. **Inverse check done** — Every meaningful decision tied back to an ASR; gold-plating and hidden requirements surfaced?
8. **Priority weighting applied** — Findings ranked by ASR priority, not flattened to a single list?
9. **Deferrals acknowledged** — Anything punted to a later phase is documented as deferred, not silently absent?
10. **Cumulative budgets respected** — Per-component satisfaction doesn't blow the system-level target when chained?

## Common failure modes

- **"All ASRs satisfied" with no traceability** — One-line assertion at the end of the doc instead of per-ASR evidence
- **Coverage without quantification** — "We address performance via caching" with no numbers tying caching to the latency target
- **Verification missing** — Design claims to satisfy an ASR but doesn't say how the team will know once it's built
- **Implicit satisfaction treated as explicit** — "Postgres gives us ACID" used to claim a transactional-integrity ASR is fully satisfied without an architected decision behind it
- **Wrong-layer satisfaction** — A decision deep in one component is claimed to satisfy a system-level ASR it doesn't actually cover
- **Cumulative blowout** — Each hop meets its slice of the latency / cost / error budget but end-to-end exceeds it
- **Silent deferral** — ASR not addressed because it's "phase 2", but no phasing plan documents that
- **Gold-plating** — Architectural decisions that satisfy no stated ASR; usually a missing requirement or an unjustified choice
- **Vendor inertia** — Decision exists because the team uses this stack everywhere, not because the ASRs demand it
- **Stale ASRs** — Catalog includes requirements the system no longer needs to satisfy; "satisfaction" is being claimed against drifted requirements
- **Priority-blind reporting** — All gaps reported with equal weight; architect can't tell which to fix first

## Deliverable

Route findings into `assets/review-template.md` by satisfaction level and ASR priority:

- **Fully satisfied + verifiable** → **Strengths**
- **Partially satisfied** → **Concerns**, or **Blockers** if the ASR is top-priority
- **Not satisfied** → **Blockers** if top-priority, **Concerns** otherwise
- **Implicit** → **Open Questions** (push the architect to make the decision explicit)
- **Deferred (acknowledged)** → **Open Questions** with the phasing reference, to validate timing
- **Decisions without ASRs** → **Concerns** (gold-plating or hidden requirement)

Open the completeness section with a one-line summary so the architect sees the picture at a glance:

> *Completeness: 7 fully satisfied, 3 partial, 2 not satisfied, 1 implicit, 1 deferred. 2 decisions in the design map to no stated ASR.*

For each ASR finding, write it as:

> **{ASR}** ({priority}) — **{Fully / Partially / Not / Implicit / Deferred} satisfied**. Addressed by: **{decision(s) and citation}**. Quantification: **{matches target / qualitative / absent}**. Verification path: **{stated / absent}**. Gap: **{what's missing, if anything}**.

For each inverse finding (decision without an ASR), write it as:

> **{Decision}** — no clear ASR backing. Likely **{gold-plating / hidden requirement / vendor inertia}**. Suggested resolution: **{remove / add the missing ASR / justify against an existing ASR}**.

Completeness findings also feed the Stage 6 risk synthesis (`references/risks.md`). Partial, not-satisfied, and implicit ASR coverage become risks; fully satisfied + verifiable ASRs become non-risks worth naming.
