# Architecture Review — {{doc title}}

**Reviewer:** Archimedes
**Stage:** {{napkin | draft RFC | ready-to-build}}
**Date:** {{YYYY-MM-DD}}

## Summary
{{2–4 sentences. What the design proposes, and the overall verdict: ship / revise / rethink.}}

## Strengths
- {{specific things the doc does well — be concrete; tag with originating analysis where relevant}}

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

## Next Steps
1. {{ordered list of what the architect should do next}}
2. ...
