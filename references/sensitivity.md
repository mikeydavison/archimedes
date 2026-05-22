# Sensitivity Analysis

A **sensitivity** is a decision in the design where a small change to one parameter, component, or assumption produces a disproportionately large change in a quality attribute (latency, availability, cost, security posture, etc.). These are the load-bearing knobs of the architecture. Finding them up front tells the architect where the design is brittle, where to invest in measurement, and where future change will be expensive.

Sensitivities are **single-attribute**: turning the knob moves one quality attribute hard. When turning a knob moves *two or more* attributes in opposing directions, it's a tradeoff — see `references/tradeoffs.md`.

## Rule of thumb

If you can finish the sentence *"the whole thing falls over if ___ changes by even a little"*, you've found a sensitivity. If you can't measure the parameter or you don't know the threshold at which it breaks, the sensitivity is unmanaged.

A typical design has **3–7 real sensitivities**. Naming more than that usually means the analysis hasn't separated load-bearing decisions from incidental ones.

---

## How to run the analysis (conversational)

Sensitivity analysis is a dialogue, not a checklist. Drive it stage by stage with the architect. Don't dump the catalog below on them — use it to generate the *next* question.

### 1. Anchor on the NFRs that matter
Start from the architecturally significant requirements (`references/asr.md`). Pick the 2–4 quality attributes the design must hit (e.g., p99 latency, availability, $/request, RPO). Sensitivities only matter relative to a target — without one there is nothing to be sensitive about.

### 2. Walk the design and ask "what is this number sensitive to?"
For each NFR target, trace the design and ask the architect:
- *Which single component, parameter, or assumption is this target most dependent on?*
- *If that thing got 10% worse, would we still meet the target? 2x worse?*
- *What's the smallest change that breaks it?*

Capture each candidate as: **{parameter} → {quality attribute} → {threshold where it breaks}**.

### 3. Probe each candidate
For every candidate sensitivity, confirm three things with the architect:

1. **The threshold is known.** "Cache hit rate drives p99" is a hypothesis. "If hit rate drops below 92% we miss p99=200ms" is a sensitivity.
2. **The parameter is monitored.** If the knob can move silently, the sensitivity is also a hidden risk.
3. **There's a response.** What does the team do when the parameter approaches the threshold — scale, fail over, degrade, page someone? "Nothing" is a valid answer but should be acknowledged.

### 4. Separate sensitivities from tradeoffs
For each confirmed sensitivity, ask: *does moving this knob affect any other NFR?* If yes, escalate it to a tradeoff and hand it to `references/tradeoffs.md`. If no, keep it as a pure sensitivity.

### 5. Rank and report
Rank by **blast radius × likelihood of the parameter actually drifting**. A sensitivity to a parameter that never changes (e.g., a hardware constant) is less interesting than one that drifts with user behavior or vendor pricing.

---

## Common sensitivity patterns to probe

Use these as conversation prompts, not as a checklist to fill.

| Pattern | Typical parameter | Attribute it moves |
|---|---|---|
| **Cache effectiveness** | Hit rate, TTL, working-set size | Latency, throughput, cost |
| **Connection / thread pool sizing** | Pool size vs. concurrent load | Latency, availability under spike |
| **Replication factor / quorum** | N, R, W settings | Durability, consistency, write latency |
| **Batch size / window** | Batch interval, max batch bytes | Latency vs. throughput, cost |
| **Timeout and retry budget** | Per-call timeout, max retries, backoff | Tail latency, availability, retry-storm risk |
| **Single dependency SLO** | Upstream's published SLO | Your own availability ceiling |
| **Vendor pricing tier** | $/unit at current vs. next tier | Unit cost, total cost at scale |
| **Hot-key / hot-partition skew** | Top-k key share of traffic | Throughput, latency, scaling cost |
| **Crypto / hash work per request** | Algorithm choice, key size | Latency, CPU cost |
| **Index / query plan stability** | Selectivity, statistics freshness | Query latency, DB load |
| **Token / rate-limit budget** | Quota from upstream API | Throughput ceiling, cost |
| **Auth-token lifetime** | Token TTL | Latency (refresh churn), blast radius of leak |
| **Clock skew tolerance** | Max acceptable skew | Correctness of ordering, lease validity |
| **Queue depth threshold** | Max queue length before backpressure | End-to-end latency, memory, drop policy |
| **Region / AZ count** | Number of failure domains | Availability, latency, cost |

---

## Review checks

1. **Named targets** — Does each claimed sensitivity tie to a quantified NFR? If the NFR is unquantified, fix that first via `references/asr.md`.
2. **Threshold stated** — Is the breaking point a number, not an adjective?
3. **Measurable parameter** — Can the team see the parameter's current value in production?
4. **Alert / response** — Is there a defined action when the parameter approaches the threshold?
5. **Source of drift** — Is it known what *causes* the parameter to move (load growth, user behavior, vendor change, data shape)?
6. **Not actually a tradeoff** — Confirm the knob really only moves one attribute. If it moves two, move it to tradeoffs.
7. **Coverage vs. inflation** — 3–7 real sensitivities is typical. A list of 20 usually means incidental decisions are mixed in; a list of 0 means the analysis wasn't done.

## Common failure modes

- **"Cache solves it"** with no stated hit-rate target or what happens on a cold cache
- **Timeouts and retries without a budget** — each layer retries 3x, so the bottom layer sees 27x amplification under failure
- **Availability target above the weakest dependency's SLO** — a hard sensitivity to that dependency that's never named
- **Hot-partition risk dismissed as "we'll shard later"** — sharding is a sensitivity to key distribution, not a fix for it
- **Vendor pricing assumed constant** — cost NFR is silently sensitive to a contract renewal
- **"It scales horizontally"** — to what limit, and which parameter (DB connections, broker partitions, license count) caps it
- **Single-AZ / single-region designs with multi-AZ availability targets** — sensitivity to one failure domain
- **Parameters tuned once at launch and never revisited** — silent drift into the brittle zone

## Deliverable

Fold findings into the main review under **Concerns** or **Blockers** in `assets/review-template.md`. For each sensitivity, write it as:

> **{Parameter}** drives **{quality attribute}** ({target}). The design holds while **{parameter} stays {within threshold}**; outside that, **{what breaks}**. Currently **{monitored / unmonitored}**, with response **{action / none}**.

Group sensitivities that share an upstream cause so the architect can address the root, not the symptoms.
