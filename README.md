# Archimedes

A standalone Claude Agent Skill that conducts conversation-based structured reviews of architecture documentation.

## What it does

Given an architecture document, design doc, RFC, ADR, or diagram, Archimedes walks the architect through a staged review covering:

- **Context** — audience, purpose, goals, constraints, assumptions, scope
- **Requirements** — completeness, quantification, and prioritization of architecturally significant requirements (ASRs)
- **Completeness** — for each ASR, does the design contain decisions that address it (and vice versa)
- **Correctness** — are the design's claims actually true (tool capabilities, math, patterns, failure modes, standards)
- **Sensitivities** — where the design is brittle to a small parameter change in a single attribute
- **Tradeoffs** — where a decision moves two attributes in opposing directions, and whether the chosen side is aligned with the prioritized ASRs
- **Alternatives** — what was rejected, and whether the rejection rationale ties back to a prioritized ASR
- **Optimality** — Pareto-improving or tradeoff-reframing alternatives the architect should consider adopting

Output is a structured review with strengths, blockers, concerns, open questions, and suggestions — plus per-analysis detailed findings — delivered conversationally so the architect can push back and iterate.

## How to use

Drop this directory anywhere Claude can find skills:

- **Claude Code**: place under `~/.claude/skills/archimedes/` (or a plugin's `skills/` directory)
- **Claude apps / API**: load via the Agent Skill mechanism for your runtime
- **Agent SDK**: register the skill folder path

Then ask Claude to review an architecture doc.

## Layout

```
archimedes/
├── SKILL.md                      # entrypoint — frontmatter + workflow
├── README.md                     # this file
├── references/                   # loaded on-demand during review
│   ├── context.md                # audience, purpose, constraints, scope (Stage 3)
│   ├── asr.md                    # ASR catalog and prioritization (Stage 4)
│   ├── completeness.md           # ASR-to-decision traceability (Stage 5)
│   ├── correctness.md            # material errors in design claims (Stage 5)
│   ├── sensitivity.md            # single-attribute brittleness (Stage 5)
│   ├── tradeoffs.md              # multi-attribute decisions and priority alignment (Stage 5)
│   ├── alternatives.md           # rejected options and rationale (Stage 5)
│   └── optimality.md             # Pareto-improving and tradeoff-reframing alternatives (Stage 5)
└── assets/
    └── review-template.md        # output format
```

## Customizing

- Edit `references/*.md` to reflect your org's standards, tech stack, or domain. Each reference is intentionally scoped — e.g., `correctness.md` focuses on material errors in claims, `optimality.md` only on Pareto-improving or tradeoff-reframing alternatives. Preserve the scope when editing.
- Edit `assets/review-template.md` to match your existing review format. Per-analysis line formats are defined in each reference's Deliverable section; keep them in sync if you rename template sections.
- Edit the `description:` field in `SKILL.md` to adjust when Claude invokes the skill.
