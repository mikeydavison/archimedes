# Correctness

Every architectural decision is, explicitly or implicitly, a **claim**: that a pattern applies, that a tool behaves a certain way, that the math adds up, that a failure mode is handled, that a standard is met. Correctness asks one thing: *are those claims actually true?* This analysis finds the places where the design says **X** but **X** isn't so — material errors that would cause the design to fail in practice if shipped as written.

This is a distinct lens from the others. Completeness asks whether each ASR is addressed; tradeoffs asks whether the chosen side is aligned with priorities; optimality asks whether a better alternative exists. Correctness asks only: *is what the doc says actually right?*

## Rule of thumb

Treat every claim in the design as a hypothesis that has to survive a sharp practitioner asking *"are you sure?"* with one follow-up question. Vague claims, vendor-marketing claims, claims that depend on unstated qualifications, and "industry standard" hand-waving all need verification before they count. Stay focused on material errors — errors that change whether the design works — not nitpicky language.

---

## How to run the analysis (conversational)

### 1. Inventory the claims
Walk the design and list every non-trivial claim it makes. Anything the design depends on being true that a competent reviewer could attempt to disprove. Typical categories:
- **Tool capability** — "Kafka supports exactly-once delivery"
- **Behavior** — "Component A retries on transient failures"
- **Math / capacity** — "100 GB/day × 90 days = 9 TB" or "this can handle 10k RPS"
- **Pattern** — "this is the saga pattern" / "we use CRDTs for merge"
- **Failure mode** — "failure of B is absorbed by C"
- **Standard / compliance** — "this design complies with PCI-DSS"
- **Causal** — "caching gives us 10x throughput"
- **Performance** — "p99 latency stays under 200 ms"

If the architect made the claim implicitly (e.g., chose a tool whose marketing claims a behavior), the claim still counts.

### 2. Classify each claim
For each claim, decide:
- **Known true** — well-established, you can confirm from authoritative source or first principles
- **Verifiable but unchecked** — a number, a citation, or a vendor statement that the architect should have but hasn't shown
- **Plausibly wrong** — your domain knowledge raises a flag; needs investigation
- **Materially incorrect** — you know it's wrong; the design will fail as written

Drop trivia. Focus on claims whose failure would change whether the architecture works.

### 3. Probe the questionable ones with the architect
For each claim in the bottom three categories, ask one sharp follow-up:
- *Where is that documented? Can you cite the qualification?*
- *Let's walk the math — what's the input, what's the multiplier, what's the result?*
- *Under what conditions does this hold? What conditions break it?*
- *Is that the actual guarantee, or is it the guarantee with qualifications?*

The architect's answer either confirms the claim (move on), reveals it as half-true (record as such), or reveals it as wrong (a Blocker).

### 4. Surface hidden claims
The most dangerous errors are claims the doc doesn't realize it's making. Look for:
- **Passive voice** — *"data is validated"* (by whom, where?), *"requests are routed"*, *"sessions are managed"*
- **Hand-waving connectives** — *"simply"*, *"just"*, *"obviously"*, *"of course"*, *"trivially"*
- **Familiar-pattern shortcuts** — *"standard pubsub setup"*, *"typical CQRS"*, *"normal caching"* — what are the specifics here?
- **Implicit-via-tool claims** — *"we use Redis, so it's atomic"* — atomic *what*, under *what* settings?

Each of these usually hides a claim that hasn't been verified.

### 5. Rank and report
Order by **what breaks if the claim is wrong**. A wrong capacity calculation that blows the load target is a Blocker; a misnamed pattern that the team still implements correctly is a Concern at most. Half-true vendor claims often live in the middle.

---

## Common claim categories to probe

| Category | Typical false-claim pattern |
|---|---|
| **Messaging guarantees** | "Exactly-once" — almost always at-least-once + idempotency, or at-most-once |
| **Transactional / atomic** | "Atomic" applied across distinct systems (DB + queue, two services); usually needs outbox / saga |
| **Idempotency** | "X is idempotent" — true for the happy path, broken at the boundary (retries with different payloads, partial state) |
| **Consistency** | "Eventually consistent" used as a magic word with no convergence bound or anomaly window stated |
| **Tool capacity** | "Postgres handles this" — at what scale, with what indexes, with what locking? |
| **Vendor SLOs** | Cited SLO higher than vendor's actual contract; or SLO cited without composition math against dependencies |
| **Performance / throughput** | RPS claim doesn't multiply against component limits; latency budget doesn't account for all hops |
| **Standards / compliance** | "Complies with X" without mapping regime → design control |
| **Failure handling** | "It will retry" — which errors retry vs. fail-fast? bounded backoff? DLQ? |
| **Pattern application** | Pattern named correctly but implementation deviates in load-bearing ways (saga without compensation, CRDT without merge function) |
| **Math / capacity** | Storage / cost / connection math that doesn't actually multiply out |
| **Causal claims** | "X gives us Y improvement" — was X actually the cause, or did Y come from something else? |
| **"Industry standard"** | Used as a substitute for justification; standards differ by industry and era |

---

## Review checks

1. **Claims inventoried** — Have you listed the non-trivial claims, including implicit ones from tool choices?
2. **Material focus** — Are you probing things that would change whether the design works, not stylistic preferences?
3. **Vendor claims verified** — Capability claims checked against current docs, with qualifications surfaced?
4. **Math multiplies** — Capacity, throughput, cost, and latency calculations actually compute to the stated result?
5. **Patterns correctly applied** — Named patterns implemented with their load-bearing parts (saga compensations, CRDT merge, outbox dispatch)?
6. **Standards mapped** — Compliance claims backed by regime → specific design controls?
7. **Failure modes specified** — Retry / fallback / DLQ behavior is concrete, not abstract?
8. **Passive voice investigated** — "Is validated" / "is routed" / "is handled" claims traced to who and where?
9. **Hand-waving challenged** — "Simply" / "obviously" / "just" prompts replaced with the actual mechanism?
10. **Architect defends claim with follow-up** — Each questioned claim has been confirmed with one sharp probe?

## Common failure modes

- **"Exactly once"** — Reflexively claimed; almost never true without the qualification (at-least-once + dedup, or at-most-once with loss)
- **"Atomic" / "transactional"** across systems — Dual-writes to DB + queue treated as atomic with no outbox or saga
- **Half-true idempotency** — Handler is idempotent for the happy path but not for retries with mutated payloads, partial commits, or out-of-order arrival
- **Capacity that doesn't multiply** — Claimed RPS exceeds connection-pool × per-connection ceiling, or storage growth exceeds quota inside the planning horizon
- **Pattern shell without substance** — "Saga" with no compensation, "CRDT" with no merge, "event sourcing" with no replayability
- **Vendor capability misquoted** — Feature stated without qualifications that are in the docs
- **SLO above dependency** — Own availability target above a hard dependency's published SLO with no acknowledgment
- **Standards claimed without mapping** — "We comply with GDPR" with no per-article design control
- **Passive voice hiding owner** — *"Inputs are validated"* with no component named to do the validating
- **"Industry standard"** — Used as proof; standards vary widely and what's standard for industry A is malpractice in industry B
- **Stale citations** — Linking to docs / blog posts that describe past behavior; current behavior may differ

## Deliverable

Route findings into `assets/review-template.md` by severity:

- **Materially incorrect** → **Blockers**
- **Half-true / important qualification missing** → **Concerns**
- **Unverifiable / unchecked** → **Open Questions** (ask the architect to verify)
- **Known true and important to the design** → consider noting under **Strengths** (correct claims that anchor risky parts of the design)

For each finding, write it as:

> **{Claim}** ({where claimed in doc}) — **{materially incorrect / half-true / unverifiable / known true}**. Evidence: **{citation, math, or check performed}**. Impact if wrong: **{what breaks in the design}**. Suggested resolution: **{verify with source / restate with qualifications / redesign / remove the claim}**.

Group related claims (e.g., several around the same messaging guarantee) so the architect can fix the root, not the individual symptoms.
