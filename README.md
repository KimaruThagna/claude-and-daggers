# claude-skills

> A curated collection of structured instruction sets that unlock consistent, high-quality outputs from Claude across complex analytical and technical tasks.

---

## What Is This?

Each file in this repository is a **skill** — a saved, reusable instruction set written for Claude.

Rather than re-explaining how to do something complex every time, you upload the relevant skill file alongside your input (a document, a dataset, a request), tell Claude to use it, and get a consistently excellent output without repeating yourself.

Think of skills as the difference between:

> *"Analyse this earnings deck — make it interactive, dark theme, tabs for financials, risks, a reverse DCF..."*

and just:

> *"Use `equity_lens.md` to analyse this deck."*

Same result. Every time. Without the prompt engineering overhead.

---

## How To Use a Skill

1. Start a new conversation with Claude (claude.ai or API)
2. Upload the skill `.md` file
3. Upload your input file (earnings deck, report, dataset, etc.)
4. Say: *"Use the skill file to analyse this"*

You can also override or extend a skill inline:

> *"Use `equity_lens.md` but skip the Reverse DCF — this is a private company"*

> *"Use `equity_lens.md` and add a competitors tab vs the two closest peers"*

---

## Skills

| File | Name | Description |
|---|---|---|
| [`equity_lens.md`](./skills/equity_lens.md) | Equity Lens | Transforms any earnings report or investor deck into a fully interactive, institutional-grade investment analysis dashboard |

*More skills added as they are developed.*

---

## Skills Format

Every skill file follows the same structure:

```
# skill_name.md
## Skill: [Human-readable name]

Overview        — what this skill produces
Artifact Spec   — tech stack, components, theme extraction
Tab Structure   — each section/tab with full spec
Data Sourcing   — tiered strategy: document → web search → enrichment
Commentary      — prose quality standards and analytical rules
Pitfalls        — known errors and how to avoid them
```

---

## Skill: Equity Lens

**File**: [`skills/equity_lens.md`](./skills/equity_lens.md)

**One line**: Drop in any investor deck or earnings report, get back an institutional-grade interactive investment analysis dashboard.

### What It Builds

A fully interactive, dark-themed React dashboard rendered directly in Claude — no setup, no deployment, no dependencies beyond the conversation.

### Tabs

| Tab | What It Contains |
|---|---|
| **Overview** | KPI grid (revenue, ARR, NRR, bookings, EBITDA margin, TAM), business description, segment revenue breakdown |
| **Financials** | 5-year revenue chart, EBITDA & margin combo chart, Non-GAAP EPS trend, ARR growth, key ratio tiles |
| **Verdict** | Investment stance (Bullish / Neutral / Bearish), two-paragraph conviction note, 8-dimension scorecard |
| **Catalysts** | 5–7 expandable catalyst cards, each with specific data-anchored thesis |
| **Risks** | 6–8 expandable risk cards with HIGH / MED / LOW severity ratings |
| **Outlook** | FY guidance KPIs, bookings trend, NRR trend, 4 strategic ambition tiles |
| **Reverse DCF** | Interactive model solving for the FCF growth rate implied by the current share price, 3-scenario grid, year-by-year cash flow table |

### Data Sourcing

The skill uses a **3-tier sourcing strategy**:

1. **Document** — extracts all financial history, segment data, guidance, and KPIs directly from the uploaded deck
2. **Web search (always)** — retrieves current share price, market cap, analyst consensus price target, rating breakdown, 52-week range, post-earnings reaction, and valuation multiples
3. **Web search (contextual)** — enriches commentary with competitor landscape, regulatory developments, peer multiples, and recent news when relevant

### Dynamic Theming

The colour theme is extracted from the presentation itself — brand accent colour, background tone (dark vs light deck), and contrast ratios are all derived from the uploaded document. No hardcoded palette.

### Example Output

Built from the **Axon Enterprise Q4 2025 Investor Deck** (February 2026):

- Extracted brand colour: `#facc15` (Axon yellow) from badge and highlight usage throughout the deck
- Dark theme applied (black background deck)
- Reverse DCF solved at `$423.44` close price, `82.37M` diluted shares, `$554M` normalised FCF base
- Implied FCF growth rate: ~29% over 10 years at 10% WACC / 4% terminal growth
- Analyst consensus: `$802.50` median target, 18 Buy / 2 Hold / 0 Sell (24 analysts)

---

## Contributing

Skills are added when a repeatable, high-quality analytical or technical workflow has been refined to the point where a single file reliably reproduces the output.

If you build a skill worth sharing, the format above is the standard.

---

## License

MIT — use freely, attribution appreciated.
