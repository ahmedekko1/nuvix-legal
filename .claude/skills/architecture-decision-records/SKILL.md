# Architecture Decision Records (ADR) Skill

Capture, format, and maintain architecture decision records whenever significant technical decisions are made.

## Trigger Signals

Activate this skill when you detect:
- Phrases like "why did we choose", "we decided to use", "we're going with X instead of Y"
- Technology selections (framework, database, cloud provider, library)
- Design pattern choices (monolith vs microservices, REST vs GraphQL, etc.)
- Significant trade-off discussions
- Reversals of prior decisions

## ADR Format

Store ADRs in `docs/adr/` numbered sequentially: `docs/adr/0001-record-architecture-decisions.md`

```markdown
# ADR-NNNN: [Short Title in Present Tense]

**Date:** YYYY-MM-DD
**Status:** [Proposed | Accepted | Deprecated | Superseded by ADR-XXXX]
**Deciders:** [names or roles]

## Context

What is the issue or situation that motivates this decision? Describe the forces at play — technical, business, social. Be factual, not evaluative.

## Decision

What is the change we're making? State it in the active voice: "We will use X."

## Rationale

Why this option over the alternatives? List key factors:
- Factor 1
- Factor 2

## Alternatives Considered

| Option | Pros | Cons |
|--------|------|------|
| Option A | ... | ... |
| Option B | ... | ... |

## Consequences

**Positive:**
- ...

**Negative / Trade-offs:**
- ...

**Neutral:**
- ...

## References
- Link to relevant docs, issues, or prior ADRs
```

## Lifecycle States

- **Proposed** — under discussion, not yet adopted
- **Accepted** — decision is active and in effect
- **Deprecated** — no longer relevant but not replaced
- **Superseded** — replaced by a newer ADR (link to it)

## Index File

Maintain `docs/adr/README.md` as an index:

```markdown
# Architecture Decision Records

| ADR | Title | Status | Date |
|-----|-------|--------|------|
| [0001](./0001-record-architecture-decisions.md) | Record architecture decisions | Accepted | 2024-01-01 |
```

## Workflow

1. Detect a decision being made in conversation or code
2. Check `docs/adr/` for the next available number
3. Create the ADR file using the template above
4. Update `docs/adr/README.md` index
5. If this supersedes an older ADR, update the old one's status

## Rules

- Titles are short noun phrases in present tense
- Context section is neutral — no advocacy, just facts
- Consequences must include negatives — hiding trade-offs defeats the purpose
- Never delete an ADR — only deprecate or supersede
- One decision per ADR — split compound decisions
