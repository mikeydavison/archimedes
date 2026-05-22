# Tradeoff Analysis

A **tradeoff** is a design decision where moving a single knob improves one quality attribute and worsens another. Every non-trivial architecture has them. The job of review is not to eliminate tradeoffs — that's impossible — but to judge whether each one was **resolved in the right direction given the prioritized requirements**, and to confirm the architect has named what was given up.

Tradeoffs are **multi-attribute**. When a knob moves only one attribute, it's a sensitivity (`references/sensitivity.md`). When it moves two or more in opposing directions, it's a tradeoff and belongs here.

ASRs in this framework are **prioritized** (see `references/asr.md`). That priority ordering is what makes a tradeoff judgeable: the chosen side must protect the higher-priority attribute and concede on the lower-priority one. A tradeoff resolved against the priority order is **misaligned** and is a review finding regardless of how cleverly it was made.

## Rule of thumb

A tradeoff that isn't written down has been resolved by accident. A tradeoff that *is* written down but sacrifices a higher-priority NFR to favor a lower-priority one has been resolved in the wrong direction — and is usually a Blocker, not a Concern. If priorities aren't set at all, the design isn't reviewable for tradeoff appropriateness; flag that first and stop.

A typical design has **3–6 real tradeoffs** worth naming. Listing more usually means dressing up preferences as tradeoffs; listing zero means the analysis wasn't done.

---

## How to run the analysis (conversational)

Tradeoff analysis is a dialogue. Use the catalog below to prompt the next question — don't recite it.

### 1. Establish the priority order first
Tradeoff analysis is only meaningful against prioritized NFRs. Before anything else, confirm with the architect:
- *Which ASRs are top-priority? Which are explicitly secondary?*
- *Where two ASRs conflict, which one wins by policy?*

If priorities are unstated or all-equal, **stop** and report that as a Blocker — every tradeoff finding downstream will be unjudgeable until this is fixed. Drive the architect to at least a tiered ranking (must-have / should-have / nice-to-have) before continuing.

### 2. Start from the sensitivities and the NFR list
Every tradeoff sits on top of at least one sensitivity. Walk the sensitivities already identified (`references/sensitivity.md`) and ask: *does this knob also move a second quality attribute?* In parallel, scan the ASR list (`references/asr.md`) for pairs that classically conflict (consistency/availability, security/latency, cost/redundancy) and ask where in the design they meet.

### 3. Name both sides of the knob
For each candidate, force the architect to state it as a pair:
- *Turning this up improves **{attribute A}** and worsens **{attribute B}**.*
- *Where on that axis did you land, and why there?*
- *What value of A did you give up? What value of B did you accept?*

If the architect can only describe one side, the tradeoff isn't yet understood.

### 4. Judge alignment against the priority order
This is the heart of the analysis. For each tradeoff, classify it:

- **Aligned** — The chosen side protects the higher-priority ASR; the conceded side is genuinely lower priority. Record the rationale and move on.
- **Misaligned** — The chosen side favors a lower-priority ASR at the expense of a higher-priority one. This is a finding. Treat misalignment on a top-tier NFR as a **Blocker**; misalignment on a secondary NFR as a **Concern**.
- **Unjudgeable** — The two attributes are at the same priority level, or priorities weren't set. Push back: either the architect provides a tiebreaker (cost ceiling, business driver, regulatory constraint) or the priority list itself needs revision.
- **Within tolerance** — The conceded side moves, but stays inside its NFR target. Note it but don't escalate.

Misalignment dressed up as cleverness ("we get 30% better throughput, only 5ms latency cost") is still misalignment if latency was the higher-priority attribute. Don't let a favorable-sounding ratio mask a priority inversion.

### 5. Probe the choice
For each named tradeoff, confirm four things:

1. **The choice is explicit.** "We chose strong consistency" — not "the database handles it."
2. **The rejected side is quantified.** "We accept p99 write latency up to 80ms" beats "writes will be a bit slower."
3. **The conceded side stays inside its NFR budget.** Quantify the sacrifice and check it against the target for that attribute. A tradeoff that pushes a lower-priority NFR *past its target* is still a failure, even if the priority order was respected.
4. **The choice is reversible — or not.** Some tradeoffs (data model, wire format, vendor lock) are one-way; the architect should know which. Irreversibility raises the bar for tolerating misalignment.

### 6. Look for hidden tradeoffs
The dangerous ones are unstated. Probe:
- Decisions framed as "best practice" or "industry standard" — these often hide a tradeoff someone else made for a different context with different priorities.
- Adopted defaults (framework defaults, cloud defaults, library defaults) — defaults encode a tradeoff made by someone who didn't know your priority order; ask whether it's the right one here.
- "We get both X and Y" claims — usually one of them is qualified in fine print.

### 7. Rank and report
Order findings by **misalignment severity first**, then by irreversibility, then by size of the sacrifice. The architect's most important question is *"which of my tradeoffs are pointing the wrong way?"* — answer that before anything else. Aligned tradeoffs go into the review as confirmations under **Strengths**; misaligned ones go to **Blockers** or **Concerns**; unjudgeable ones go to **Open Questions**.

---

## Common tradeoff patterns to probe

Use these as conversation prompts. Each row names the knob and the two attributes it moves in opposing directions.

| Knob | Improves | Worsens |
|---|---|---|
| **Strong consistency** | Correctness, reasoning simplicity | Availability under partition, write latency |
| **Synchronous replication** | Durability, consistency | Write latency, availability under replica loss |
| **Caching** | Latency, throughput, cost | Freshness, cache-invalidation complexity |
| **Batching** | Throughput, cost per item | End-to-end latency |
| **Encryption / signing on the hot path** | Security, integrity | Latency, CPU cost |
| **Fine-grained authz checks** | Security, least privilege | Latency, code complexity |
| **More replicas / regions** | Availability, durability, read latency | Cost, write latency, operational complexity |
| **Aggressive retries** | Success rate under transient failure | Tail latency, retry-storm risk, cost |
| **Idempotency keys / dedup stores** | Correctness under retry | Storage cost, write latency |
| **Schema normalization** | Write simplicity, integrity | Read latency, query complexity |
| **Denormalization / materialized views** | Read latency | Write amplification, freshness, storage |
| **Async / event-driven** | Decoupling, availability, throughput | Debuggability, consistency reasoning, end-to-end latency |
| **Microservice decomposition** | Independent deploy, team scaling | Operational complexity, end-to-end latency, debugging |
| **Managed / vendor service** | Time-to-market, operability | Cost at scale, lock-in, portability |
| **Build in-house** | Fit, control, no lock-in | Time-to-market, ongoing maintenance load |
| **Multi-tenant shared infra** | Cost, utilization | Blast radius, noisy-neighbor risk, isolation |
| **Configurability / feature flags** | Flexibility, safe rollout | Combinatorial test surface, cognitive load |
| **Eventual consistency** | Availability, latency, scale | Reasoning complexity, user-visible anomalies |
| **Compression** | Storage cost, network cost | CPU, latency for small payloads |
| **Pre-computation / pre-aggregation** | Read latency | Write cost, staleness, storage |
| **Rich client / thick edge** | Latency, offline support | Update cadence, security surface, payload size |

---

## Review checks

1. **Priorities exist** — Are the ASRs explicitly prioritized? If not, every check below is unanswerable.
2. **Both attributes named** — Does the doc state which two (or more) attributes the decision moves, and in which direction?
3. **Side chosen explicitly** — Is the chosen side stated as a decision, not as a side-effect of tool choice?
4. **Aligned with priority order** — Does the chosen side protect the higher-priority ASR? Flag any inversion.
5. **Conceded side within budget** — Is the sacrificed attribute still inside its stated NFR target after the concession?
6. **Sacrifice quantified** — Is the cost on the rejected side a number or budget, not an adjective?
7. **Tied to a requirement** — Does a *higher-priority* NFR or business constraint justify which side was chosen? "Preference" or "elegance" is not a valid driver against a prioritized ASR.
8. **Reversibility called out** — Does the doc say whether revisiting the tradeoff later is cheap, expensive, or one-way? Misaligned + irreversible is the worst combination.
9. **No false "both"** — Where the doc claims to get both sides ("fast *and* consistent", "flexible *and* simple"), is there a clear mechanism, or is it wishful?
10. **Stakeholder visible** — For tradeoffs that affect users, ops, or downstream teams, has the affected party seen and accepted the chosen side?

## Common failure modes

- **Priority inversion** — A lower-priority attribute (often cost, performance, or developer convenience) wins over a higher-priority one (often security, durability, or compliance) with no acknowledgment that the priority order was violated
- **Unprioritized NFR list** — All ASRs treated as equally important, making every tradeoff appear locally reasonable and globally unjudgeable
- **Favorable-ratio framing hiding inversion** — "Only 5% latency cost for 30% throughput gain" — fine if throughput is higher priority, a Blocker if latency is
- **Conceded side blown past target** — Priority-aligned tradeoff that nonetheless pushes the lower-priority NFR outside its stated budget (still a failure)
- **Tool choice hiding the decision** — "We use Kafka" silently picks throughput-and-durability over end-to-end latency; the tradeoff is never written down, so alignment can't be assessed
- **Defaults treated as decisions** — Framework / cloud defaults inherited without checking whether the implicit priority order matches this system's
- **One-sided framing** — "Cassandra gives us scale" with no mention of what consistency model was accepted
- **"Best of both worlds"** claims without the mechanism that makes it true
- **Tradeoffs resolved at the wrong layer** — Latency budget eaten by a low-level choice that the high-level SLA can't recover from
- **CAP / PACELC ignored** — Availability *and* strong consistency *and* multi-region writes claimed simultaneously
- **Security treated as free** — Authn/z, encryption, audit added without naming the latency, complexity, or UX cost (often a hidden inversion when latency is the higher-priority ASR)
- **Cost treated as free** — Redundancy, replication, observability stacked without a stated $/unit impact
- **Lock-in framed as zero-cost** — Managed service adopted without naming what portability was traded away
- **Reversibility assumed** — Data model and wire-format decisions described as "we can change it later" when in practice they can't; misaligned + irreversible deserves the strongest pushback

## Deliverable

Route findings into `assets/review-template.md` by alignment classification:

- **Aligned** tradeoffs → **Strengths** (a confirmed correct call worth naming)
- **Misaligned** tradeoffs on a top-tier ASR → **Blockers**
- **Misaligned** tradeoffs on a secondary ASR, or aligned tradeoffs where the conceded side breaches its budget → **Concerns**
- **Unjudgeable** tradeoffs (no priority set, or attributes at equal priority with no tiebreaker) → **Open Questions**

For each tradeoff, write it as:

> **{Decision / knob}** improves **{attribute A}** ({value gained}) at the cost of **{attribute B}** ({value given up}). Priority order: **A > B** / **B > A** / **equal / unset**. Verdict: **aligned / misaligned / within tolerance / unjudgeable**. Chosen because **{higher-priority requirement / driver}**. Reversibility: **{cheap / expensive / one-way}**.

Group related tradeoffs (e.g., several decisions that all spend latency to buy availability) so the architect can see the cumulative bill on the sacrificed attribute. A series of individually-aligned tradeoffs can still add up to push the conceded attribute past its NFR target — call that out explicitly.

Tradeoff findings also feed the Stage 6 risk synthesis (`references/risks.md`). Misaligned tradeoffs and conceded sides outside their budget become risks; aligned tradeoffs with quantified concessions inside budget become non-risks. Cumulative-bill blowouts on a shared conceded attribute are a classic theme pattern.
