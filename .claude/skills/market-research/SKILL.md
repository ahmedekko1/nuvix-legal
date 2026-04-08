# Market Research Skill

Systematic competitive analysis, market sizing, and customer research workflows.

## Research Workflow

```
1. Define the question
2. Identify sources
3. Collect data
4. Synthesize findings
5. Produce deliverable
```

Never skip step 1. A vague question produces useless output.

## Competitive Analysis Framework

For each competitor, capture:

| Field | What to find |
|-------|-------------|
| **Product** | Core features, pricing tiers, differentiators |
| **Positioning** | Tagline, ICP, messaging angle |
| **Traction** | Funding, employee count, G2/Capterra rating, review volume |
| **Weaknesses** | Negative review themes, missing features, complaints |
| **Go-to-market** | Channels (SEO, paid, partnerships), content strategy |

### Sources

- **Product**: Company website, demo videos, feature pages
- **Pricing**: Pricing page (archive.org for history)
- **Reviews**: G2, Capterra, Trustpilot, Reddit, HackerNews
- **Traffic**: SimilarWeb, Semrush (estimated)
- **Funding**: Crunchbase, PitchBook
- **Employees**: LinkedIn company page
- **Tech stack**: BuiltWith, Wappalyzer

## Customer Research

### Jobs-to-be-Done Interview Template

```
1. "Tell me about the last time you tried to [accomplish job]. What were you doing?"
2. "What triggered you to start looking for a solution?"
3. "What did you try before finding [product]?"
4. "What almost stopped you from switching?"
5. "What would you tell a friend about why you use it?"
```

### Review Mining (No-Interview Research)

Pull themes from 1-star and 5-star reviews separately:

**5-star pattern → what customers love most**
**1-star pattern → biggest failure modes / unmet needs**

```
Sources:
- G2: Filter by rating, export reviews
- Capterra: Same
- Reddit: site:reddit.com "[product name]" OR "[competitor name]"
- App Store / Play Store: if mobile product
```

## Market Sizing

### TAM → SAM → SOM

```
TAM (Total Addressable Market)
└── Everyone who could theoretically buy this
    Method: Top-down (industry reports) or bottom-up (count of buyers × ARPU)

SAM (Serviceable Addressable Market)
└── TAM filtered by your geography, segment, and go-to-market reach

SOM (Serviceable Obtainable Market)
└── Realistic share you can win in 3-5 years
    Rule of thumb: 1-5% of SAM for early-stage
```

### Bottom-Up Sizing Example

```
ICP: US law firms with 10-50 employees
Count: ~35,000 firms (from American Bar Association data)
Penetration: 15% likely to consider AI receptionist
Buyers: 35,000 × 0.15 = 5,250
ARPU: $400/month = $4,800/year
SAM: 5,250 × $4,800 = $25.2M ARR
```

## Deliverable Formats

### Competitive Landscape Table

```markdown
| Company | Pricing | Key Feature | Weakness | Funding |
|---------|---------|-------------|----------|---------|
| Acme AI | $299/mo | CRM sync | No mobile | $5M Seed |
| RezBot  | $149/mo | 24/7 voice | No integrations | Bootstrapped |
| Us      | $199/mo | WAT-native | Early stage | — |
```

### Executive Summary (1 page)

```
## Market Research: [Topic]
**Date:** YYYY-MM-DD
**Question:** [The specific question this answers]

### Key Findings
1. ...
2. ...
3. ...

### Competitive Gaps (Opportunities)
- ...

### Risks / Threats
- ...

### Recommended Actions
1. ...
```

## Research Quality Checklist

- [ ] Primary question stated clearly before starting
- [ ] At least 3 independent sources per major claim
- [ ] Data points have dates (markets change)
- [ ] Negative findings included (confirmation bias check)
- [ ] Assumptions labeled as assumptions, not facts
- [ ] Deliverable answers the original question
