# Architecture Review — {{doc title}}

**Reviewer:** Archimedes
**Stage:** {{napkin | draft RFC | ready-to-build}}
**Date:** {{YYYY-MM-DD}}

## Summary
{{2–4 sentences. What the design proposes, and the overall verdict: ship / revise / rethink.}}

## Risk Themes
*Per `references/risks.md`. Each theme groups related risks by common root cause and ties to a business driver. Read this section to see the systemic concerns; drill into Detailed Findings → Risks & Themes for the underlying risks.*

- **T-1: {{short name}}** ({{Blocker | Concern | Open Question}}) — {{one-line root cause}}. Threatens: {{business driver}}. Rolls up: {{R-1, R-3, R-7}}.

*If risks don't cluster into a systemic theme, state that explicitly: "Risks are individually contained; no systemic theme identified."*

## Strengths
- {{specific things the doc does well — be concrete; tag with originating analysis where relevant}}

### Non-Risks
*Decisions affirmatively validated as safe given current assumptions. Per `references/risks.md`.*

- **NR-1:** {{decision}} validated against {{ASR}} — {{evidence: threshold + monitoring + response, verification path, or authoritative source}}. Stays a non-risk while {{load-bearing assumption}} holds.

## Blockers
*Must be resolved before building.*

- **{{title}}** ({{analysis}}) — {{one-line summary; see Detailed Findings for the structured entry}}

## Concerns
*Worth resolving, not blocking.*

- **{{title}}** ({{analysis}}) — {{one-line summary; see Detailed Findings for the structured entry}}

## Open Questions
*The doc doesn't answer these; the reviewer needs answers to fully judge the design.*

- **{{question}}** ({{analysis}}) — needed because {{reason}}

## Suggestions
*Alternatives or refinements, with tradeoffs.*

- **{{title}}** ({{analysis}}) — {{what to change; tradeoff: what you give up}}

---

## Detailed Findings

Each section below uses the per-finding line format defined in its reference. Omit any section with no findings to report; for analyses where everything is in order, a one-line confirmation is enough.

### Context
*Per `references/context.md` Deliverable.*
{{Note whether context is solid, has gaps, or is too thin to judge the design. List per-element gaps using the prescribed line format.}}

### Requirements (ASRs)
*Per `references/asr.md` Deliverable.*
{{List the ASRs being assessed against, with priority tier. Note any gaps in coverage, quantification, or prioritization using the prescribed line format.}}

### Completeness
*Per `references/completeness.md` Deliverable.*
{{Open with the summary line — counts by satisfaction level. Then list per-ASR findings and inverse (decision-without-ASR) findings using the prescribed line formats.}}

### Correctness
*Per `references/correctness.md` Deliverable.*
{{Per-claim findings, ordered by what breaks if the claim is wrong.}}

### Sensitivities
*Per `references/sensitivity.md` Deliverable.*
{{Per-sensitivity findings. Group sensitivities that share an upstream cause.}}

### Tradeoffs
*Per `references/tradeoffs.md` Deliverable.*
{{Per-tradeoff findings, ordered by misalignment severity. Group related tradeoffs that spend the same attribute.}}

### Alternatives
*Per `references/alternatives.md` Deliverable.*
{{Per-finding entries. If the doc covers alternatives well, note as a Strength and keep this section short.}}

### Optimality
*Per `references/optimality.md` Deliverable.*
{{Per-finding entries. If no findings, say so explicitly — "design is well-aligned with its prioritized ASRs" is a valid outcome.}}

### Risks & Themes
*Per `references/risks.md` Deliverable. Synthesis of findings across the analyses above.*
{{Open with a one-line summary — count of risks, non-risks, and themes (e.g., "5 risks, 4 non-risks, 2 themes"). Then list per-risk entries in priority order, then per-theme entries referencing the risks by ID. Non-risks already appear under Strengths → Non-Risks above; don't repeat them here.}}

## Next Steps
1. {{ordered list of what the architect should do next}}
2. ...
