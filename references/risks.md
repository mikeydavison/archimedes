# Risks, Non-Risks, and Risk Themes

A **risk** is an architectural decision (or omission) that may lead to undesirable consequences for one or more prioritized quality attributes. A **non-risk** is a decision that has been affirmatively validated as safe given the current assumptions. A **risk theme** is a cluster of individual risks that share a common underlying cause, named so stakeholders can see the systemic concern rather than only the local symptoms.

This stage is a *synthesis* over the Stage 5 analyses, not a parallel one. Sensitivities, misaligned tradeoffs, incorrect claims, unsatisfied ASRs, missing alternatives, and context gaps each produce findings — Risk Synthesis pulls them together, clusters them by root cause, and maps the resulting themes to the business drivers captured in `references/context.md`. The output is what executives, sponsors, and affected downstream teams need to see; the per-analysis detail remains in the Detailed Findings.

This is distinct from:
- `references/sensitivity.md` — single-attribute brittleness (a class of risk, but local)
- `references/tradeoffs.md` — direction of multi-attribute decisions (also a class of risk, but local)
- `references/correctness.md` — whether claims are true (incorrect claims become risks here)

Risk synthesis is the place where these merge.

## Rule of thumb

A typical review surfaces **3–7 risks and 2–5 themes**. Listing 20 risks usually means the synthesis didn't cluster; listing 0 means it didn't happen. Every risk must cite the analysis it came from; every theme must cite the business driver it threatens. Affirmative non-risks are equally important — naming what *isn't* a risk is what makes the review honest rather than uniformly critical.

---

## How to run the synthesis (conversational)

Risk synthesis is mostly internal work — by Stage 6 the architect has already answered the questions that produced the findings. Confirm with the architect at the end rather than running a separate elicitation.

### 1. Harvest from upstream analyses
Walk every finding from Stage 5 and Stage 4 (and any Context blockers from Stage 3). Each becomes either a risk candidate or a non-risk:

- **Sensitivity findings** — unmonitored thresholds, single dependencies on weak SLOs, brittle parameters → risks. Sensitivities with stated thresholds, monitoring, and defined responses → non-risks.
- **Tradeoff findings** — misaligned tradeoffs, conceded sides outside their budget → risks. Aligned tradeoffs with quantified concessions inside budget → non-risks.
- **Correctness findings** — materially incorrect claims, half-true vendor claims → risks. Verified claims that anchor risky parts of the design → non-risks worth naming.
- **Completeness findings** — partial / not-satisfied / implicit ASR coverage → risks. Fully satisfied + verifiable ASRs → non-risks.
- **Alternatives findings** — missing alternatives or rejection by preference → risks (often reframed as "design is sensitive to its ASR ranking and hasn't been stress-tested against alternatives").
- **Optimality findings** — improvement opportunities, not risks. **Excluded from this pass** — they stay under Suggestions.
- **Context findings** — missing constraints, assumptions, or business drivers → risks (often highest-impact, because they undermine every downstream judgment).
- **ASR findings** — unquantified or unprioritized ASRs → meta-risks ("the design isn't reviewable for tradeoff appropriateness"); usually Blockers.

If a finding translates to neither a risk nor a non-risk, drop it from this synthesis — it lives only in the Detailed Findings of its originating analysis.

### 2. Score each risk
Use the existing severity routing — Blocker / Concern / Open Question — weighted by:

- **Impact** — severity of the consequence to the prioritized ASR. A risk to a top-tier ASR is more severe than the same risk to a nice-to-have.
- **Likelihood** — how likely the triggering condition is. Name the trigger (parameter drift, vendor change, traffic growth, partition, dependency failure, contract renewal, team turnover), not just "could happen."

A high-impact risk with a low-likelihood trigger is usually a Concern. A medium-impact risk with a high-likelihood trigger is also usually a Concern. High on both is a Blocker. Low on both is barely worth naming.

Don't invent a new severity vocabulary. Likelihood and impact feed the existing routing; they don't replace it.

### 3. Cluster into themes by common cause
Look across the risk list for shared roots:

- **Single dependency** — multiple risks conditioned on the same upstream service / vendor / library.
- **Single failure domain** — multiple risks landing in the same region, AZ, cluster, account.
- **Single assumption** — multiple risks resting on the same unstated belief (traffic forecast, pricing tier, regulatory interpretation, team capacity).
- **Single attribute under pressure** — multiple decisions all spending the same conceded attribute (e.g., several tradeoffs that each cost a little latency; cumulatively the latency budget is gone).
- **Single missing capability** — multiple risks tracing back to one absent control (no DLQ, no idempotency layer, no audit log, no schema registry).
- **Single business driver under threat** — risks across different analyses that all imperil one stated outcome.

A "theme" without a shared root isn't a theme — it's a list. Discard.

### 4. Map themes to business drivers
For each theme, identify which business driver from `references/context.md` (Goals of the change) it threatens. If a theme doesn't tie to a stated business driver, either:
- The business driver is missing from the context — push back upstream and surface it, or
- The theme is in service of a tacit goal the architect hasn't articulated.

Themes without business-driver mapping are weak — stakeholders have nothing to act on.

### 5. Identify non-risks explicitly
Walk the design once more and name decisions that have been *validated as safe* — not silent, not absent, *validated*. Common sources:
- Sensitivities with stated thresholds, monitoring, and defined responses.
- Tradeoffs aligned with the priority order, with quantified concessions inside budget.
- ASRs fully satisfied with explicit decisions and verification paths.
- Claims confirmed against authoritative sources.

Naming non-risks does two things: it reassures stakeholders that strong parts of the design have been checked, and it documents *why* a decision was confirmed safe — so it doesn't get relitigated. Not naming non-risks makes a review look more critical than it is.

### 6. Confirm with the architect
Before delivery, surface the themes:
- *Are these the right clusters? Did I miss a common cause?*
- *Are there business drivers I should be mapping to that aren't in the context doc?*
- *Are there decisions you want named as non-risks that I haven't flagged?*

Update before delivery. Don't defend themes the architect can show aren't actually shared-root.

---

## Common theme patterns

Use these as prompts during clustering, not as a catalog to fill.

| Theme pattern | Typical root cause |
|---|---|
| **Single-vendor concentration** | Multiple risks all condition on one cloud / DB / SaaS provider's availability, pricing, or roadmap |
| **Single-region exposure** | Multiple availability / latency / DR risks rolling into one geographic failure domain |
| **Hidden coupling to a weak SLO** | Self-SLO targets above an upstream's stated SLO; multiple decisions inherit the gap |
| **Cumulative budget exhaustion** | Several individually-aligned tradeoffs each spend a little of the same attribute (latency, cost, complexity); the bill adds up past target |
| **Unmonitored sensitivity cluster** | Multiple knobs are load-bearing but none are instrumented; silent drift threatens multiple ASRs |
| **Implicit assumption stack** | Several decisions all rest on one unstated belief (traffic forecast, vendor pricing, regulatory interpretation) |
| **Missing systemic control** | Several risks could be closed by one absent capability (DLQ, outbox, idempotency layer, audit log, schema registry) |
| **Skill / staffing fragility** | Multiple operational risks tied to one team, one on-call rotation, one specialist |
| **Reversibility cliff** | Several decisions individually defensible but cumulatively lock in a path that's expensive to back out of |
| **Compliance surface drift** | Multiple decisions claim compliance without per-control mapping; risk lands as one regulatory theme |

---

## Review checks

1. **Sourcing** — Does every risk cite the upstream analysis (or analyses) that produced it? Risks without provenance are speculation, not findings.
2. **Severity rationale** — Does each risk's severity state both impact and likelihood, with the trigger named?
3. **Theme cohesion** — Does every theme have a stated common cause, not just a label? "Performance risks" is a label; "cumulative latency from four small tradeoffs" is a cause.
4. **Theme-to-driver mapping** — Does every theme tie to at least one business driver from `references/context.md`?
5. **Non-risks explicit** — Are decisions validated as safe named, not just implied by their absence from the risk list?
6. **No double-counting** — A finding that surfaces in multiple analyses (e.g., a sensitivity that's also a tradeoff misalignment) appears as one risk, sourced from both.
7. **Disciplined volume** — 3–7 risks and 2–5 themes for a typical design. Inflation reads as criticism; deflation reads as superficial.
8. **Optimality excluded** — Improvement suggestions don't get re-labeled as risks to raise their visibility; they stay under Suggestions.
9. **Priority-weighted** — Risks against top-tier ASRs ranked above same-shape risks against secondary ASRs.
10. **Treatment named** — Each risk has a suggested treatment (mitigate / accept / transfer / defer), not just identification.

## Common failure modes

- **Themes that are just lists** — "Performance" and "Security" as themes, with no shared root cause inside each. Themes must name a *cause*.
- **Risks without provenance** — A risk appears in synthesis that wasn't a finding in any upstream analysis. Either go back and add it to the right analysis, or drop it.
- **Double-counting** — Same finding produces a sensitivity risk *and* a tradeoff risk *and* a completeness risk, inflating the count. Merge into one risk sourced from multiple analyses.
- **Theme has no business driver** — A theme that doesn't threaten anything in the context's Goals; stakeholders have nothing to act on.
- **Severity inflation** — Every risk graded Blocker; the prioritization signal is lost.
- **Severity deflation** — Genuine top-tier ASR threats demoted to Concerns because the architect was defensive; severity must follow the analysis, not the conversation.
- **No non-risks named** — Review reads as uniformly critical; validated decisions go unrecognized and get relitigated next iteration.
- **Treatment missing** — Risks identified but no suggested response; architect has to invent mitigations from scratch.
- **Optimality smuggled in** — Improvement opportunities labeled as risks to make them harder to ignore. Don't do this — optimality stays under Suggestions.
- **Themes invented without clustering** — A small number of unrelated risks get a thematic label slapped on them; theme cohesion fails.

## Deliverable

Risk synthesis produces three outputs in `assets/review-template.md`:

1. **Risk Themes** section (near the top, after Summary) — one line per theme with the business-driver tag, severity, and IDs of the risks rolling up.
2. **Non-Risks** (tagged subsection of Strengths) — affirmative validations.
3. **Risks & Themes** (end of Detailed Findings) — full structured entries.

### Per-risk format

> **Risk R-N:** {decision or omission} may lead to **{QA consequence}** for **{ASR}**. Likelihood: **{high / med / low}** — trigger: **{specific condition}**. Impact: **{severity tied to ASR priority}**. Sourced from: **{analysis or analyses}**. Suggested treatment: **{mitigate / accept / transfer / defer}** — **{specific action}**.

### Per-non-risk format

> **Non-risk NR-N:** **{decision}** has been validated to satisfy **{ASR}** because **{evidence — threshold + monitoring + response, or verification path, or authoritative source}**. Stays a non-risk while **{load-bearing assumption}** holds.

### Per-theme format

> **Theme T-N: {short name}** — **{one-line description of the root cause}**. Rolls up: **{R-1, R-3, R-7}**. Threatens business driver: **{goal from context.md}**. Severity: **{Blocker / Concern / Open Question}**. Suggested treatment: **{theme-level mitigation or acceptance posture}**.

Number risks, non-risks, and themes sequentially in the order they appear in the Detailed Findings. Reference risks from themes by ID so the executive-level Risk Themes section can be read on its own and drilled into when needed.

If the synthesis surfaces no real themes — risks exist but don't cluster — say so explicitly. *"Risks are individually contained; no systemic theme identified."* is a valid outcome on small or narrowly-scoped designs.
