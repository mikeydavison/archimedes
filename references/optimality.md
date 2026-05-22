# Optimality

Optimality finds opportunities to improve the design **without making it worse on any prioritized ASR**. A suggestion qualifies only when it is either:

1. **Pareto-improving** — meets all prioritized ASRs at least as well as the chosen design, does better on at least one, and introduces no new tradeoff cost.
2. **Tradeoff-reframing** — meets the prioritized ASRs by replacing one tradeoff with a different one whose shape better matches the priority order (e.g., trades a concession on a low-priority ASR for protection of a high-priority one).

Critiques that introduce new tradeoffs without a priority-based justification are not optimality findings — they are preferences. Reserve this analysis for suggestions that genuinely strengthen the design against its own ASRs.

This is distinct from `references/alternatives.md`: alternatives audits the architect's own rejection rationale (*did you consider X, and is your reason for rejecting it valid?*). Optimality proactively suggests *adopting* a better option the architect may not have considered.

## Rule of thumb

Before raising an optimality finding, state three things explicitly: **(1) what the alternative is**, **(2) which ASR it improves and by how much**, **(3) which ASR it concedes and by how much — if any**. If a finding can't answer all three, it isn't ready. "Simpler" / "more elegant" / "what I would have done" are not optimality findings.

A typical review surfaces **0–3 real optimality findings**. Most designs that survived context, requirements, completeness, sensitivity, tradeoffs, and alternatives have already found their shape. Listing more usually means smuggling in preferences.

---

## How to run the analysis (conversational)

### 1. Anchor on the priority order
Pull the prioritized ASR list from `references/asr.md` and the tradeoff findings from `references/tradeoffs.md`. Optimality is only meaningful relative to those priorities. If priorities aren't set, this analysis cannot run — fix that first.

### 2. Walk each load-bearing decision
Go decision by decision through the design. For each, ask:
- *Is there an alternative that meets the priorities at least as well, and does better on something?*
- *Is there an alternative whose tradeoffs are shaped more like the priority order than the current choice?*

Don't probe trivial decisions — focus on the ones that shape the architecture.

### 3. Classify each candidate
For every alternative you surface, place it in one of three buckets:

- **Pareto improvement** — strictly no worse on every ASR; better on at least one.
- **Tradeoff reframe** — different tradeoff shape; better matches the priority order.
- **Preference / drop** — the alternative introduces a new tradeoff cost that isn't justified by priorities. Discard.

Most candidates fall into the third bucket. That's normal — be disciplined about dropping them.

### 4. Validate Pareto claims
Pareto is easy to claim and hard to prove. For every candidate marked Pareto, walk the full ASR list and confirm the alternative is not worse on any of them. Common hidden costs to check:
- Operational complexity (new tool to learn, monitor, page on)
- Lock-in or portability
- Team skill / cognitive load
- Migration / cutover effort
- Failure modes the original avoided

If any cost surfaces, downgrade the finding to a reframe — and then check that the reframe actually serves the priorities.

### 5. Validate reframe claims
For each tradeoff-reframe candidate, confirm:
- The conceded ASR is genuinely lower-priority than the protected ASR (per the priority order).
- The improvement on the higher-priority ASR is meaningful, not marginal.
- The conceded ASR still meets its NFR target after the reframe.

A reframe that improves a top-priority attribute by a hair while pushing a secondary attribute past its budget is not a win.

### 6. Respect the stage and the effort
Even a valid optimality finding may not be worth surfacing if the cost to adopt vastly outweighs the gain — particularly on late-stage docs where rework is expensive. Calibrate by the artifact's stage (see SKILL.md "Respect the stage"). Note the rework cost in the finding so the architect can decide.

### 7. Rank and report
Order findings by **size of improvement against the highest-priority ASR**, then by **adoption cost**. Pareto improvements that are cheap to adopt go to the top. Reframes that require significant rework go lower, even if the improvement is real.

---

## Common Pareto-improvement patterns

| Pattern | Why it's often Pareto |
|---|---|
| **Off-the-shelf replaces bespoke** | Same capability, less code to maintain, faster delivery |
| **Removing an intermediate layer** | Same behavior, fewer hops, less to operate |
| **Dropping a speculative abstraction** | Same behavior today, less cognitive load and refactor cost |
| **Using a tool the team already operates** | Same capability, no new on-call burden |
| **Removing a cache when underlying call is fast enough** | Same latency outcome, fewer invalidation problems |
| **Consolidating two near-duplicate components** | Same behavior, half the operational surface |
| **Removing a feature flag whose decision is settled** | Same behavior, less combinatorial test surface |
| **Replacing custom retries with a library** | Same retry behavior, less bug surface |

Verify each against the full ASR list before claiming Pareto — these are *typically* Pareto but not always.

## Common tradeoff-reframing patterns

| Pattern | When the reframe wins |
|---|---|
| **Strong → eventual consistency** | When business tolerates anomalies and availability / latency are higher priority |
| **Sync → async** | When throughput and decoupling are higher priority than end-to-end latency |
| **Async → sync** | When debuggability and ordering are higher priority than peak throughput |
| **In-house → managed service** | When time-to-market or operability is higher priority than cost-at-scale |
| **Managed → in-house** | When cost-at-scale or portability is higher priority than time-to-market |
| **Centralized → distributed** | When blast radius and tenant isolation are higher priority than operational simplicity |
| **Distributed → centralized** | When operational simplicity and consistency are higher priority than blast radius |
| **Batch → streaming** | When latency rises in priority above per-event cost |
| **Streaming → batch** | When per-event cost or processing simplicity rises above latency |
| **Single-region → multi-region** | When availability / latency-to-user is higher priority than cost and operational complexity |
| **Multi-region → single-region** | When cost and operational simplicity are higher priority than tail availability |

Each row is reversible — direction depends on which side of the tradeoff the prioritized ASRs sit on. Use the catalog as prompts, not as recommendations.

---

## Review checks

1. **Priorities anchor the analysis** — Each finding cites which ASR it improves and (if applicable) which it concedes
2. **Three-question test passed** — Alternative named, improvement quantified, concession quantified?
3. **Pareto claims walked** — For Pareto findings, confirmed not-worse on every ASR including operational, team, and lock-in costs?
4. **Reframe claims aligned** — For reframe findings, the protected ASR is higher-priority than the conceded one?
5. **Conceded ASR still inside budget** — Reframes don't push a secondary ASR past its NFR target?
6. **Preferences dropped** — No findings based on aesthetics, familiarity, or "what I would have done"?
7. **Adoption cost stated** — Each finding notes the rework effort, so the architect can weigh adoption?
8. **Stage-appropriate** — Optimization findings calibrated to the artifact's stage; no demand for 10x rework on a napkin sketch?
9. **Disjoint from alternatives** — Not relitigating options the architect already validly rejected per `references/alternatives.md`?
10. **Volume disciplined** — Findings count proportional to the design's optimization room (usually small)?

## Common failure modes

- **Disguised preferences** — "I'd use X instead" presented as optimization with no priority-based justification
- **Hidden tradeoffs in Pareto claims** — Suggested alternative is faster *and* cheaper but adds a new vendor, on-call surface, or skill requirement that wasn't accounted for
- **"Simpler"** that isn't — Subjective simplicity claimed without measuring against all ASRs (operability, evolvability, team fit)
- **Reframes that don't reframe** — Suggestion swaps one tradeoff for an identical one in different clothes
- **Improvements past the point of need** — Pushing for optimization on an ASR that's already comfortably inside target
- **Stage-disproportionate suggestions** — Demanding architectural rework on a doc that's still exploring
- **Volume inflation** — Many findings, each individually thin, that collectively read as criticism rather than improvement
- **Relitigating rejected alternatives** — Surfacing the same options the architect already considered and rejected with valid reasoning
- **Vendor switches without parity check** — "Use vendor Y instead" without confirming Y meets the ASRs that drove the choice of vendor X
- **Removing load-bearing complexity** — "Simplify by removing this" where the thing being removed is doing real work for a real ASR

## Deliverable

Optimality findings are suggestions, not blockers. Route them into `assets/review-template.md` under **Suggestions**, with the rare structural ones promoted to **Concerns** when the improvement is large and the design is still revisable.

For each finding, write it as:

> **{Alternative}** for **{current decision}** — type: **{Pareto / reframe}**. Improves: **{ASR + how much}**. Concedes: **{ASR + how much, or "none"}**. Net for prioritized ASRs: **{strictly better / better priority alignment}**. Adoption cost: **{cheap / moderate / significant rework}**. Suggested action: **{adopt / spike / defer to next iteration}**.

If the analysis surfaces no real findings, say so explicitly. *"No optimality findings — the design is already well-aligned with its prioritized ASRs"* is a valid and valuable outcome that confirms the architect's work.
