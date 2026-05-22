# Architecturally Significant Requirements (ASRs)

ASRs are the requirements that materially shape the architecture — primarily **non-functional requirements (NFRs)** plus a small set of functional requirements with structural impact. Guidance on use of ASRs is found at https://iasa-global.github.io/btabok/requirements.html.  Use this catalog during review to check which ASRs the design names targets for, which it ignores, and which are stated but unmeasurable.

## Rule of thumb

An NFR without a number is not an NFR. "Highly available" is a wish; "99.95% monthly uptime, RTO ≤ 2 min, RPO ≤ 30 s" is reviewable. ASRs should also describe how the system must behave in response to a stimulus, not only what state it holds. Flag any NFR stated without a target. Don't get hung up on nitpicky language — focus on whether the requirement describes how the system must respond.

Most designs only need **5–8 NFRs** treated rigorously — the ones that constrain the shape of the system. Pushing for all of them is noise.

---

## How to run the analysis (conversational)

### 1. Inventory the ASRs the doc names
Walk the document and list every requirement that materially shapes the architecture. Use the tier tables below to spot omissions — top-tier categories not named, second-tier ones missing without justification, often-missed categories silently absent.

### 2. Verify quantification and stimulus-response form
For each named ASR, confirm it has a number (per Rule of thumb) and describes the system's response to a stimulus. Adjective-only ASRs ("scalable", "fast", "secure") are gaps — flag them.

### 3. Establish priority order
With the architect, rank the ASRs at least into tiers (must-have / should-have / nice-to-have). Tradeoffs and completeness depend on this — without priorities, downstream analyses can't judge alignment and will stop.

### 4. Run the review checks
Coverage, quantification, tradeoff honesty, verification path, inheritance, scaling regime — see the checks below.

---

## Top tier — present in nearly every design

| Category | What it covers | Typical metrics |
|---|---|---|
| **Performance** | Speed under expected load | Latency p50/p95/p99, throughput (RPS, MB/s), response time |
| **Scalability** | Behavior as load / data grows | Scale dimension (horiz/vert), headroom multiple, scaling cost curve |
| **Availability** | Fraction of time usable | Uptime % (99.9, 99.95, 99.99), error budget, planned-downtime policy |
| **Reliability** | Correctness under failure | MTBF, MTTR, fault tolerance, retry/fallback behavior |
| **Security** | Confidentiality, integrity, authn/z | Threat model, encryption (rest + transit), audit, secret handling |
| **Maintainability** | Cost of changing the system | Modularity, test coverage, deploy frequency, lead time for changes |
| **Observability** | Ability to know what's happening | Log/metric/trace coverage, alert SLO coverage, MTTD |

## Second tier — usually applicable

| Category | What it covers | Typical metrics |
|---|---|---|
| **Recoverability / DR** | Survival of catastrophic failure | RPO, RTO, backup cadence, restore drill frequency |
| **Durability** | Data survival once committed | Annual loss probability (e.g., 11 nines), replication factor |
| **Capacity** | Hard ceilings | Storage limits, connection/quota limits, file/row count caps |
| **Consistency** | Read/write guarantees | Strong vs. eventual, isolation level, read-your-write rules |
| **Compliance** | Regulatory / contractual | Applicable regimes (GDPR, HIPAA, SOC 2, PCI-DSS, FedRAMP), data residency |
| **Compatibility** | Working with other systems | API/protocol versioning policy, backwards/forwards compatibility window |
| **Portability** | Movement across environments | Cloud / OS / runtime independence, lock-in inventory |
| **Usability** | Ease of use for humans | Task completion rate, time-on-task, WCAG conformance level |
| **Cost / Economic** | TCO and unit economics | $/request, $/GB stored, $/active user, scaling cost curve |

## Often-missed but increasingly load-bearing

| Category | What it covers |
|---|---|
| **Privacy** | Data minimization, retention, consent, subject-access rights (distinct from security) |
| **Sustainability** | Carbon intensity, power efficiency, hardware lifecycle |
| **Safety** | Hazard analysis for cyber-physical, medical, automotive, industrial systems |
| **Localization / i18n** | Multi-language, locale-aware formatting, RTL support, character encoding |
| **Accessibility** | WCAG 2.x / Section 508 conformance, assistive-tech compatibility |
| **Deployability** | Time to deploy a change, rollback time, blue/green or canary support |

---

## Review checks

When reading a design doc, run through these:

1. **Coverage** — Are the architecturally significant NFRs named? Which top-tier ones are silent?
2. **Quantification** — Does each named NFR have a number, or only an adjective?
3. **Tradeoff honesty** — Where two NFRs conflict (e.g., consistency vs. availability, security vs. usability), does the doc state which it favors and why? (Full analysis in `references/tradeoffs.md`.)
4. **Verification path** — How will the team measure compliance with each NFR? Load test? Chaos test? Synthetic probe? Audit?
5. **Inheritance** — Are inherited NFRs from upstream/downstream systems stated? (e.g., your 99.99% target is impossible if a hard dependency is 99.9%.)
6. **Scaling regime** — Are NFR targets stated at current scale only, or at 10x / projected peak?

## Common failure modes

- "High performance" / "highly scalable" / "secure by design" — adjectives, not requirements
- NFRs listed but never referenced in the design rationale
- Mutually impossible targets (e.g., strong consistency + 99.999% availability across regions)
- NFR targets that exceed dependency SLOs
- Compliance named but no mapping from regime → design control
- Cost listed as an NFR with no model behind the number

## Deliverable

ASR findings shape every downstream analysis. Route into `assets/review-template.md`:

- **Missing top-tier ASR** → **Blockers**
- **Stated but unquantified ASR** → **Concerns** (or **Blockers** if top-tier)
- **Unprioritized ASR list** → **Blockers** — downstream analyses (completeness, tradeoffs) depend on priorities and will stop
- **Missing verification path** → **Open Questions**
- **Comprehensive, quantified, prioritized ASR set** → **Strengths**

For each gap, write it as:

> **{ASR or category}** — **{absent / unquantified / unprioritized / unverifiable}**. Why it matters: **{which downstream analysis depends on it}**. Suggested resolution: **{quantify, rank, add the requirement, or specify verification method}**.

ASR findings also feed the Stage 6 risk synthesis (`references/risks.md`). Unquantified or unprioritized ASRs surface there as meta-risks — the design isn't reviewable for tradeoff appropriateness until they're resolved.