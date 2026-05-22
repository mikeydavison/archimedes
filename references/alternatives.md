# Alternatives Considered

A design without rejected alternatives is a design that wasn't chosen — it was defaulted into. Competent architects evaluate at least 2–3 plausible options and can say, in ASR terms, why each loser lost. The job of this review stage is to surface those alternatives and confirm the rejection rationale ties back to prioritized requirements, not to preference, familiarity, or sunk effort.

## Rule of thumb

For every load-bearing decision in the design, the architect should be able to name **at least one alternative they rejected** and **which ASR drove the rejection**. "We picked X" with no losers named is not a decision — it's a preference dressed as one.

If the artifacts already list alternatives with reasons tied to ASRs, this stage is short: validate the reasoning, note any obvious option that's missing, and move on. If they don't, run the full probe below.

---

## How to run the analysis (conversational)

### Short path — alternatives are documented

If the artifacts include a "Considered alternatives", "Options", or ADR-style section:

1. **Check coverage.** Are the named alternatives the obvious ones, or only strawmen? Cross-check against the catalog below.
2. **Check rejection rationale.** For each rejected option, is the reason stated as *"fails ASR X because Y"*, or as *"we didn't like it"*? Flag any rejection that doesn't cite a prioritized requirement.
3. **Check for the missing alternative.** Is there a plausible option not even mentioned? Name it and ask why it was excluded from consideration.
4. **Stop.** No need to elicit further if the architect has done the work.

### Long path — alternatives are absent or thin

For each load-bearing decision in the design, walk the architect through:

1. *What other options did you consider for this?*
2. *What was the simplest thing that could have worked here?*
3. *Did you consider doing nothing — or doing this in a later phase?*
4. *Did you consider buying instead of building (or vice versa)?*
5. *For each rejected option: which specific ASR did it fail, and by how much?*

Capture each as: **{rejected option} → failed {ASR} because {specific reason}**.

Then push one level deeper:

6. *If your top ASR were different — say cost instead of latency — would you have made the same choice?* This tests whether the rejection logic is robust or accidental.
7. *Is there a partial adoption — using the rejected option for one piece, the chosen option for another?* Often the best answer is a hybrid the architect hadn't considered.

---

## Alternatives worth probing for

Use these as prompts. Each is an option architects routinely fail to consider explicitly.

| Class | Example prompt |
|---|---|
| **Do nothing** | Is this problem worth solving now? What breaks if we wait a quarter? |
| **Do the simplest thing** | Cron + SQL, a single script, a manual process — why isn't that enough? |
| **Buy / SaaS / managed** | Is there a vendor offering that covers 80% of this? Why reject it? |
| **Build in-house** | If the chosen path is a vendor, was building considered? What ruled it out? |
| **Reuse existing infra** | Is there a system already in production that could absorb this? |
| **Different paradigm** | Sync vs async, batch vs streaming, monolith vs services, push vs pull |
| **Different storage** | SQL vs NoSQL vs object store vs in-memory; row vs column; one DB vs many |
| **Different consistency model** | Strong vs eventual; was the weaker model considered and rejected on which ASR? |
| **Different deployment model** | Single-region vs multi-region; single-tenant vs multi-tenant; edge vs central |
| **Different vendor** | If a specific cloud / DB / queue was chosen, were the alternatives evaluated on the same ASR set? |
| **Smaller scope** | Is there an MVP that delivers 80% of the value? Was it considered as the destination, not just the path? |
| **Phased rollout** | Could this be done in two smaller steps that each independently de-risk? |
| **Off-ramp / reversibility** | Was an alternative chosen partly because it's easier to back out of? |

---

## Review checks

1. **At least one alternative per load-bearing decision** — Can the architect name what they rejected?
2. **Rejection tied to a prioritized ASR** — Is the reason a specific requirement, not a preference?
3. **Rejection quantified where possible** — "Fails latency target by ~3x" beats "too slow"
4. **Obvious options not missing** — Cross-check against the catalog; flag any unmentioned plausible alternative
5. **No strawmen** — Are the alternatives real candidates, or set up to lose?
6. **Hybrid considered** — Was a mix of the chosen and rejected options on the table?
7. **Robust to ASR shift** — Would the choice still hold if priorities were slightly different, or is it sensitive to one specific ASR ranking?

## Common failure modes

- **Single-option design** — No alternatives named; the chosen approach is presented as inevitable
- **Strawman alternatives** — Options listed only to be dismissed in a sentence, with no real analysis
- **Rejection by preference** — "We don't like X" or "the team isn't familiar with X" stated as architectural reasoning
- **Rejection by familiarity** — Chosen option is the one the architect knows best; alternative wasn't seriously evaluated
- **Sunk-cost rejection** — "We already use Y elsewhere" treated as decisive when the ASRs would favor a different choice here
- **Vendor-first thinking** — Tool picked, then justified; alternatives evaluated against the winner's strengths rather than the ASRs
- **Missing the "do nothing" option** — Building assumed; the option of not building (or building later) never explicitly considered
- **Missing the "boring" option** — Off-the-shelf, single-binary, or cron-and-SQL versions skipped over for something more interesting
- **No hybrid explored** — Treated as A-or-B when A-for-X-and-B-for-Y was available
- **Rationale that doesn't survive a priority shift** — Choice would flip if the #1 and #2 ASRs swapped, and nobody has stress-tested that

## Deliverable

If the artifacts already document alternatives well, note that as a **Strength** and skip to any gaps.

For each gap or weak rationale, fold into `assets/review-template.md`:

> **{Decision}** — alternative **{rejected option}** was **{considered / not considered}**. Stated rejection reason: **{reason / none}**. ASR driving rejection: **{ASR / unclear}**. Reviewer note: **{whether the rationale holds, or what's missing}**.

If a plausible alternative was never mentioned, raise it as an **Open Question**:

> Did you consider **{alternative}**? It would seem to fit because **{reason}**, but may fail **{ASR}** — please confirm whether it was evaluated.

Don't push alternatives the architect has already validly rejected. The goal is to expose unexamined choices, not to relitigate examined ones.

Alternatives findings also feed the Stage 6 risk synthesis (`references/risks.md`). Missing alternatives and rejections by preference become risks (often framed as "design is sensitive to its ASR ranking and hasn't been stress-tested against alternatives"); thorough alternatives analysis with priority-grounded rejections becomes a non-risk worth naming.
