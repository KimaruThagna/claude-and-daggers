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
3. Upload your input file (earnings deck, report, dataset, etc.) **or** provide your tickers/request inline
4. Say: *"Use the skill file to analyse this"*

You can also override or extend a skill inline:

> *"Use `equity_lens.md` but skip the Reverse DCF — this is a private company"*

> *"Use `equity_comparator.md` and add a third stock for comparison"*

---

## Skills

| File | Name | Description |
|---|---|---|
| [`equity_lens.md`](./skills/equity_lens.md) | Equity Lens | Transforms any earnings report or investor deck into a fully interactive, institutional-grade investment analysis dashboard |
| [`equity_comparator.md`](./skills/equity_comparator.md) | Equity Comparator | Takes 2+ stock tickers and generates a side-by-side institutional research report — financials, catalysts, risks, reverse DCF, and a portfolio allocation verdict |

*More skills added as they are developed.*

---

## Screenshots

![TMDX Earnings Q42025](https://github.com/KimaruThagna/claude-and-daggers/blob/main/screenshots/tmdx-q4-2025-overview.png)
![COHR VS LITE](https://github.com/KimaruThagna/claude-and-daggers/blob/main/screenshots/Screenshot%202026-03-09%20165253.png)
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

- Extracted brand colour: `#facc15` (Axon yellow)
- Reverse DCF solved at `$423.44` close price, `82.37M` diluted shares, `$554M` normalised FCF base
- Implied FCF growth rate: ~29% over 10 years at 10% WACC / 4% terminal growth
- Analyst consensus: `$802.50` median target, 18 Buy / 2 Hold / 0 Sell

---

## Skill: Equity Comparator

**File**: [`skills/equity_comparator.md`](./skills/equity_comparator.md)

**One line**: Give it 2+ tickers, get back a side-by-side institutional research report with allocation verdict, rendered as a premium interactive React dashboard.

### What It Builds

A single-page, Apple-ethos dark-themed React dashboard comparing two or more stocks across every dimension a portfolio manager cares about — sourced live from the web, rendered in one shot.

### Tabs

| Tab | What It Contains |
|---|---|
| **Overview** | Side-by-side company snapshots, sector/macro themes driving both names, KPI cards, 1-paragraph thesis per stock |
| **Financials** | Revenue growth bar chart (3yr + forward), gross margin trend, EPS trajectory, balance sheet snapshot per company |
| **Catalysts** | 5–6 near-term catalysts per stock, shared macro tailwinds, sector heat indicators |
| **Risks** | Risk matrix table: factor · severity (1–5) · likelihood (1–5) · mitigation — color-coded by severity |
| **Reverse DCF** | 10-year reverse DCF per stock solving for implied FCF CAGR at current price · 3-scenario bar chart · priced-to-perfection indicator |
| **Verdict** | Bullish / Neutral / Bearish stance badges · donut allocation chart · allocation reasoning · weighted decision matrix · first-person analyst conviction note |

### Data Sourcing

The skill always web-searches before generating the report:

1. **Current price, market cap, shares outstanding** — live from search
2. **TTM financials** — latest reported revenue, gross margin, EPS, FCF, debt
3. **Forward estimates** — analyst consensus for current and next fiscal year
4. **Recent news** — catalysts, earnings surprises, strategic announcements

### Design System

- Background `#080910` · accent palette: Blue `#4A90E2` · Gold `#F5A623` · Green `#34D399`
- Font: `-apple-system, 'SF Pro Display', 'Helvetica Neue'`
- Sticky tab nav · glassmorphism KPI cards · dark-gridline Recharts · themed tooltips
- Fully self-contained — no external API calls, all data hardcoded at generation time

### Example Output

Built from **COHR vs LITE** (March 2026):
![COHR VS LITE](https://github.com/KimaruThagna/claude-and-daggers/blob/main/screenshots/Screenshot%202026-03-09%20165253.png)

- Both stocks had just received simultaneous $2B NVIDIA investments
- Reverse DCF: COHR implied ~27% FCF CAGR (stretched), LITE implied ~37% (priced-to-perfection)
- Allocation verdict: COHR 55% / LITE 35% / Cash 10%
- Stances: COHR 🟢 Bullish · LITE 🟡 Bullish with valuation caution

---

## Contributing

Skills are added when a repeatable, high-quality analytical or technical workflow has been refined to the point where a single file reliably reproduces the output.

If you build a skill worth sharing, the format above is the standard.

---

## License

MIT — use freely, attribution appreciated.
