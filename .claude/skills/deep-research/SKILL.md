# Deep Research Skill

Structured methodology for thorough, multi-source research on complex technical and business topics.

## When to Use Deep Research

Use this skill when:
- The question requires synthesizing information from multiple sources
- A quick web search would give shallow or conflicting answers
- The output will drive an important decision
- The topic is technical and requires reading documentation, papers, or code

## Research Protocol

### Phase 1: Frame the Question

Before searching, write out:
1. **Core question**: What exactly are we trying to know?
2. **Decision it serves**: How will this answer be used?
3. **Success criteria**: What does a complete answer look like?
4. **Scope boundaries**: What is explicitly out of scope?

### Phase 2: Source Hierarchy

Prioritize sources in this order:

| Tier | Source Type | Use For |
|------|-------------|---------|
| 1 | Official docs, specs, RFCs | Technical facts, API behavior |
| 2 | Primary research (papers, studies) | Data, benchmarks |
| 3 | Engineering blogs (Stripe, Vercel, etc.) | Implementation patterns |
| 4 | Reputable news (Reuters, FT, WSJ) | Business/market facts |
| 5 | Community (Stack Overflow, Reddit) | Real-world experience, edge cases |
| 6 | AI-generated content | Never cite as source |

### Phase 3: Structured Search

```
Technique: Layered search

Round 1 — Broad: "[topic] overview"
Round 2 — Specific: "[topic] [specific aspect]"
Round 3 — Contrarian: "[topic] problems" OR "[topic] vs alternative"
Round 4 — Recency: "[topic] 2024" or site:news.ycombinator.com [topic]
Round 5 — Expert: "[topic] site:github.com" or arxiv.org for papers
```

### Phase 4: Note-Taking During Research

For each source:
```
Source: [URL or citation]
Date: [publication date]
Key claim: [one sentence]
Evidence: [what supports it]
Conflicts with: [any contradiction to prior findings]
```

Flag conflicts explicitly — don't silently pick the convenient answer.

### Phase 5: Synthesis

Structure findings as:
1. **Consensus** — what all/most sources agree on
2. **Contested** — where sources disagree (explain why)
3. **Uncertain** — where data is thin or missing
4. **Your assessment** — explicitly labeled as analysis, not fact

## Output Templates

### Technical Research Report

```markdown
# Research: [Topic]
**Date:** YYYY-MM-DD  
**Researcher:** [name/role]  
**Question:** [exact question]

## Summary (TL;DR)
[3-5 sentences answering the core question]

## Findings

### [Finding 1]
[Detail with source citation]

### [Finding 2]
...

## Trade-offs / Considerations
| Factor | Option A | Option B |
|--------|----------|----------|
| ...    | ...      | ...      |

## Gaps / Open Questions
- ...

## Sources
1. [Title](URL) — [date]
2. ...
```

### Decision Brief

```markdown
# Decision Brief: [Topic]
**Recommendation:** [clear recommendation in one sentence]
**Confidence:** High / Medium / Low

## Why This Recommendation
1. ...
2. ...

## What We Considered
- Alternative 1: [rejected because...]
- Alternative 2: [rejected because...]

## Risks
- ...

## Next Steps
1. ...
```

## Research Anti-Patterns

**Avoid:**
- Searching to confirm a predetermined answer
- Stopping after the first credible source
- Treating LLM output as a source (it's synthesis, not evidence)
- Omitting contradicting evidence
- Using data without noting its date
- Conflating correlation with causation in studies

**Instead:**
- Steelman the alternatives
- Seek disconfirming evidence deliberately
- Check when data was collected
- Note sample sizes for studies
- Prefer primary over secondary sources

## Quality Checklist

- [ ] Core question stated before research began
- [ ] At least 3 independent sources for key claims
- [ ] Contradictions surfaced and addressed
- [ ] All data points dated
- [ ] Assumptions explicitly labeled
- [ ] Recommendation clearly tied to findings
- [ ] Confidence level stated
- [ ] Gaps/unknowns acknowledged
