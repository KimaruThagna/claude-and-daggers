# equity_lens.md
## Skill: Institutional-Grade Earnings & Investor Deck Analyzer

---

## Overview

This skill produces a fully interactive, dark-themed React artifact that transforms any earnings report, investor presentation, or financial filing into a comprehensive investment analysis dashboard. The output mirrors the quality of a sell-side equity research note combined with a buy-side investment memo — complete with live interactive charts, a reverse DCF model, scorecard, verdict, and severity-rated risks.

The artifact is built as a single self-contained React component using `recharts` for all visualisations. It is structured around **7 tabs**, each serving a distinct analytical purpose.

---

## Artifact Specification

### Tech Stack
- **Framework**: React (default export, no required props)
- **Charts**: `recharts` — `BarChart`, `LineChart`, `Bar`, `Line`, `XAxis`, `YAxis`, `Tooltip`, `ResponsiveContainer`, `CartesianGrid`, `ReferenceLine`
- **State**: `useState` hooks only
- **Styling**: Inline styles only — no Tailwind, no CSS files
- **String interpolation**: Always use string concatenation (`"$" + v + "M"`) — never template literals (backticks cause parser errors)
- **Arrow functions in JSX callbacks**: Use explicit `function()` syntax inside `.map()` to avoid parser issues

### Colour Theme — Dynamic Brand Extraction

The colour theme is **derived from the presentation's visual identity**, not hardcoded. Before writing any code, analyse the uploaded deck for:

1. **Primary brand colour** — the dominant non-neutral colour used for headlines, buttons, highlights, and badges (e.g. Axon's yellow `#facc15`, Salesforce's blue `#00A1E0`, Palantir's dark teal)
2. **Secondary brand colour** — used for accents, icons, or supporting UI elements
3. **Background colour** — is the deck dark (black/charcoal) or light (white/grey)?
4. **Text colour** — primary heading colour used throughout slides

#### Extraction Process

**Step 1 — Identify deck tone**
- If the deck background is dark (black, navy, charcoal, dark grey) → use a **dark theme** base
- If the deck background is light (white, light grey, cream) → use a **light theme** base (inverted card/bg values, dark text)

**Step 2 — Extract the brand accent colour**
Look for the most frequently used non-neutral highlight colour across:
- CTA buttons and badges
- Slide title underlines or dividers
- Chart bar fills or icon tints
- Header backgrounds or banner fills

This becomes `T.brand` — the primary accent used for: ticker badge background, active tab indicator, KPI value highlights, verdict badge, catalyst badge numbers, and chart bars.

**Step 3 — Derive the full palette from the brand colour**

Once `brandHex` is identified, derive the rest of the theme algorithmically:

```js
// Dark theme (deck has dark background)
const T = {
  bg:     "#0d0f14",          // near-black page background — keep fixed
  card:   "#13161e",          // slightly lighter card background — keep fixed
  border: "#1e2330",          // subtle border — keep fixed

  brand:  brandHex,           // extracted from deck (e.g. "#facc15" for Axon)
  brandDim: brandHex + "22",  // 13% opacity version for badge backgrounds

  // Semantic colours — fixed regardless of brand
  green:  "#34d399",          // always used for positive/bullish signals
  amber:  "#fbbf24",          // always used for caution/neutral signals
  red:    "#f87171",          // always used for negative/bearish/high-risk

  // Text hierarchy — fixed
  white:  "#f1f5f9",          // primary text
  sub:    "#9ca3af",          // body / description text
  muted:  "#6b7280",          // labels, captions, secondary UI

  // Functional aliases driven by brand
  blue:   brandHex,           // replaces hardcoded blue — used for chart fills, KPI values, links
  yellow: brandHex,           // replaces hardcoded yellow — used for active tabs, brand badge
};
```

```js
// Light theme (deck has light background)
const T = {
  bg:     "#f8f9fa",          // off-white page background
  card:   "#ffffff",          // white card background
  border: "#e5e7eb",          // light grey border

  brand:  brandHex,           // extracted from deck
  brandDim: brandHex + "22",

  green:  "#059669",          // darker green for light bg legibility
  amber:  "#d97706",          // darker amber for light bg legibility
  red:    "#dc2626",          // darker red for light bg legibility

  white:  "#111827",          // inverted — dark text on light bg
  sub:    "#374151",          // dark grey body text
  muted:  "#6b7280",          // medium grey labels

  blue:   brandHex,
  yellow: brandHex,
};
```

#### Brand Colour Examples by Company

| Company | Deck Style | Brand Hex | Theme |
|---|---|---|---|
| Axon Enterprise | Dark (black bg, yellow accents) | `#facc15` | Dark |
| Salesforce | Light (white bg, blue accents) | `#00A1E0` | Light |
| Palantir | Dark (black bg, teal/blue accents) | `#6fc7b8` | Dark |
| Datadog | Dark (dark navy, purple accents) | `#774aa4` | Dark |
| Snowflake | Light (white bg, cyan/blue accents) | `#29B5E8` | Light |
| CrowdStrike | Dark (dark bg, red accents) | `#e8002d` | Dark |
| HubSpot | Light (white bg, orange accents) | `#ff7a59` | Light |
| Nvidia | Dark (black bg, green accents) | `#76b900` | Dark |
| Microsoft | Light (white bg, blue accents) | `#0078d4` | Light |
| Stripe | Light (white/purple bg, indigo accents) | `#635bff` | Light |

#### Where Brand Colour Is Applied

The `T.brand` / `T.yellow` / `T.blue` colour (all aliased to `brandHex`) is used in:

- **Header**: ticker badge background (`background: T.brand`, `color: contrastText`)
- **Tab bar**: active tab bottom border (`borderBottom: "2px solid " + T.brand`)
- **KPI cards**: primary value colour for key metrics
- **Charts**: bar fills and line strokes for primary data series
- **Verdict banner**: border colour when bullish matches brand (otherwise green)
- **Catalyst badges**: numbered pill background (`background: T.brandDim`, `color: T.brand`)
- **Reverse DCF**: slider `accentColor`, terminal value row highlight, implied price row
- **Scorecard**: score colour on strong (9–10) tiles

#### Contrast Text on Brand Colour Badge

For the ticker badge (solid brand colour background), choose the text colour based on luminance:

```js
// Simple luminance check — use black text on light brand colours, white on dark
function contrastText(hex) {
  const r = parseInt(hex.slice(1,3), 16);
  const g = parseInt(hex.slice(3,5), 16);
  const b = parseInt(hex.slice(5,7), 16);
  const luminance = (0.299 * r + 0.587 * g + 0.114 * b) / 255;
  return luminance > 0.55 ? "#000000" : "#ffffff";
}
```

- Axon yellow `#facc15` → luminance ~0.77 → black text ✓
- CrowdStrike red `#e8002d` → luminance ~0.19 → white text ✓
- Salesforce blue `#00A1E0` → luminance ~0.45 → white text ✓

#### Semantic Colours Remain Fixed

`T.green`, `T.amber`, and `T.red` are **never replaced by the brand colour**. They serve as universal semantic signals:
- Green = positive, bullish, strong
- Amber = caution, neutral, mixed
- Red = negative, bearish, high risk, warning

These must remain perceptually distinct from the brand colour to preserve meaning. If the brand colour is green (e.g. Nvidia), `T.green` for bullish signals should shift to a visually distinct shade (e.g. `#6ee7b7` lighter mint) to avoid confusion.

### Reusable Components

**`KPI` card** — single metric tile:
```jsx
const KPI = ({ label, value, sub, color }) => (
  <div style={{ background:T.card, border:"1px solid #1e2330", borderRadius:12, padding:"18px 20px" }}>
    <div style={{ fontSize:11, color:T.muted, textTransform:"uppercase", letterSpacing:"0.08em", marginBottom:6 }}>{label}</div>
    <div style={{ fontSize:26, fontWeight:800, color: color || T.blue, lineHeight:1.1 }}>{value}</div>
    {sub && <div style={{ fontSize:12, color:T.sub, marginTop:4 }}>{sub}</div>}
  </div>
);
```

**`Section`** — labelled section wrapper with divider:
```jsx
const Section = ({ title, children }) => (
  <div style={{ marginBottom:24 }}>
    <div style={{ fontSize:11, fontWeight:700, color:T.muted, textTransform:"uppercase", letterSpacing:"0.1em", marginBottom:14, paddingBottom:8, borderBottom:"1px solid #1e2330" }}>{title}</div>
    {children}
  </div>
);
```

**`TT`** — custom Recharts tooltip:
```jsx
const TT = ({ active, payload, label }) => {
  if (!active || !payload || !payload.length) return null;
  return (
    <div style={{ background:"#1a1e2b", border:"1px solid #1e2330", borderRadius:8, padding:"10px 14px", fontSize:12 }}>
      <div style={{ color:"#9ca3af", marginBottom:4 }}>{label}</div>
      {payload.map((p, i) => (
        <div key={i} style={{ color: p.color || "#f1f5f9" }}>{p.name}: <b>{p.value}</b></div>
      ))}
    </div>
  );
};
```

---

## Tab Structure

### Tab 1 — Overview

**Purpose**: Executive snapshot. Answers "what is this company and how is it performing?"

**KPI Grid (3-column, 6 tiles)**:
- Revenue (current year, YoY growth %)
- Annual Recurring Revenue (ARR) or equivalent recurring metric
- Backlog / Future Contracted Bookings
- Net Revenue Retention (NRR) or equivalent retention metric
- Adjusted EBITDA Margin
- Total Addressable Market (TAM) with penetration context

**Business Description Panel**:
- Prose paragraph (~100 words) explaining what the company does
- Highlight key product lines, customer segments, mission/moonshot goals
- Use `<span style={{ color:T.white }}>` to emphasise product names inline within the muted body text

**Revenue Mix Panel**:
- Side-by-side segment cards showing quarterly revenue for each segment
- Include adjusted gross margin % per segment
- One-line description of what each segment contains

---

### Tab 2 — Financials

**Purpose**: Multi-year financial history with interactive charts.

**Charts to include**:

1. **Annual Revenue Bar Chart** — 4–5 years, yellow bars, CAGR in title
2. **Adjusted EBITDA & Margin Combo Chart** — bars for absolute EBITDA (left Y-axis), line for margin % (right Y-axis, green)
3. **Non-GAAP EPS Line Chart** — trend line, yellow, shows earnings power growth
4. **ARR or Recurring Revenue Bar Chart** — green bars, shows subscription momentum

**Key Financial Ratios KPI row (4-column)**:
- Non-GAAP Net Income
- GAAP Net Income (flag SBC drag if material)
- Free Cash Flow (flag if compressed or diverging from EBITDA)
- Adjusted Gross Margin %

**Chart configuration rules**:
- Always `vertical={false}` on `CartesianGrid`
- Always `axisLine={false}` and `tickLine={false}` on axes
- Dual Y-axis charts: `yAxisId="left"` and `yAxisId="right"` with `orientation="right"`
- Tick formatters use string concatenation: `tickFormatter={function(v) { return "$" + v + "M"; }}`

---

### Tab 3 — Verdict

**Purpose**: Investment conclusion. The most opinionated tab.

**Verdict Banner**:
- Full-width gradient panel (`linear-gradient(135deg, ...)`)
- Border colour matches verdict: green for bullish, red for bearish, amber for neutral
- Badge pill at top: `BULLISH` / `BEARISH` / `NEUTRAL`
- Two prose paragraphs (~120 words each):
  - **Para 1**: The bull thesis — why the company is exceptional, what makes it structurally advantaged
  - **Para 2**: The key risks and what to monitor — honest, balanced, not dismissive

**Scorecard Grid (2-column, 8 tiles)**:

Each tile contains:
- Dimension label
- Score (e.g. "9/10")
- Score colour (green if strong, amber if mixed, red if weak)
- One-line rationale

**Standard scorecard dimensions**:
1. Revenue Growth Quality
2. Margin Profile
3. Revenue Visibility / Backlog Quality
4. Competitive Moat
5. Free Cash Flow Quality
6. Valuation Risk
7. TAM / Market Opportunity
8. Governance / Management Quality

**Scoring guide**:
- 8–10: Genuinely exceptional, durable advantage
- 5–7: Solid but with caveats or execution dependency
- 1–4: Structural weakness or serious concern

---

### Tab 4 — Catalysts

**Purpose**: Bullish thesis drivers in detail. Expandable accordion format.

**Structure**: 5–7 catalyst cards, expandable on click

Each catalyst card:
- Numbered badge (green background)
- Title (bold, 13px)
- Expand/collapse toggle (`+` / `-`)
- On expand: detailed paragraph (80–120 words) explaining the catalyst with specific data points, metrics, and logical chain of impact

**Catalyst types to include**:
- Product-led revenue expansion (ARPU uplift, new SKUs, subscription tier upgrades)
- TAM penetration opportunity with specific % underpenetrated
- Backlog / contracted revenue visibility
- Geographic or vertical expansion runway
- New product category creation
- Margin expansion via mix shift

**Interaction**: `useState` for `openCat` (null or index). Click toggles same index closed, different index opens new one.

---

### Tab 5 — Risks

**Purpose**: Honest, severity-rated risk register.

**Structure**: 6–8 risk cards, expandable accordion (same pattern as Catalysts)

**Severity levels and colours**:
- `HIGH` → `T.red` (#f87171) — existential or near-term material risk
- `MED` → `T.amber` (#fbbf24) — real but manageable or longer-dated
- `LOW` → `T.green` (#34d399) — worth noting but unlikely to be decisive

**Badge styling**: `background: color + "22"` (hex with 22 = ~13% opacity), solid colour text

**Risk categories to consider**:
- Customer concentration (government, single vertical, single geography)
- Valuation / re-rating risk (multiple compression on miss)
- Free cash flow quality (divergence between EBITDA and FCF)
- Competitive threats (incumbents, new entrants, commoditisation)
- Regulatory / political risk
- Governance issues (material weakness, SBC dilution, related party)
- Macro sensitivity (budget cycles, interest rates, FX)

Each risk: detailed paragraph (80–120 words) with specific numbers, mechanisms, and impact chain. Never vague.

---

### Tab 6 — Outlook

**Purpose**: Forward-looking guidance, strategic targets, and trend momentum.

**Guidance KPI row (3-column)**:
- Revenue guidance (growth % + implied absolute range)
- Margin guidance (EBITDA or gross margin)
- Capital expenditure guidance

**Charts**:
1. **Backlog / Bookings trend** — bar chart, 4–5 quarters or years, shows compounding visibility
2. **NRR or retention trend** — line chart, with `ReferenceLine y={100}` in red as the critical floor

**Strategic Ambitions Grid (2-column, 4 tiles)**:
- Each tile: bold yellow title + 2–3 sentence description of the strategic priority
- Draw from management commentary, investor day targets, or stated product roadmap
- Frame as: what needs to happen, why it matters, what success looks like

---

### Tab 7 — Reverse DCF

**Purpose**: Valuation stress test. Answers "what growth does this price demand?"

#### Mathematical Model

**calcImpliedGrowth function** — binary search to find the FCF growth rate that makes DCF equity value equal to market cap:

```js
function calcImpliedGrowth(price, sharesM, fcfBase, waccR, termGrowth, years) {
  const mktCap = price * sharesM * 1e6;
  let g = 0.25, lo = 0, hi = 1.5;
  for (let iter = 0; iter < 80; iter++) {
    let pv = 0, fcf = fcfBase;
    for (let t = 1; t <= years; t++) {
      fcf *= (1 + g);
      pv += fcf / Math.pow(1 + waccR, t);
    }
    const tv = (fcf * (1 + termGrowth)) / (waccR - termGrowth);
    pv += tv / Math.pow(1 + waccR, years);
    if (Math.abs(pv - mktCap) < 1e6) break;
    if (pv < mktCap) { lo = g; g = (g + hi) / 2; }
    else { hi = g; g = (g + lo) / 2; }
  }
  return g;
}
```

**buildDCF function** — builds year-by-year cash flow table for a given growth rate:

```js
function buildDCF(fcfBase, growth, waccR, termGrowth, years, sharesM) {
  const rows = [];
  let fcf = fcfBase, pv = 0;
  for (let t = 1; t <= years; t++) {
    fcf *= (1 + growth);
    const disc = fcf / Math.pow(1 + waccR, t);
    pv += disc;
    rows.push({
      year: "Y" + t,
      fcf: Math.round(fcf / 1e6),
      pv: Math.round(disc / 1e6),
      cumPv: Math.round(pv / 1e6)
    });
  }
  const tvPv = (fcf * (1 + termGrowth)) / (waccR - termGrowth) / Math.pow(1 + waccR, years);
  const equity = pv + tvPv;
  return {
    rows,
    tv: Math.round(tvPv / 1e6),
    equity: Math.round(equity / 1e6),
    impliedPrice: equity / (sharesM * 1e6)
  };
}
```

#### FCF Base Selection Logic

When reported FCF is distorted (e.g. by large one-time capex, campus investments, M&A integration), use a **normalised FCF proxy**:

```
Normalised FCF = Adjusted EBITDA × (1 − blended tax rate)
```

- Use 20–25% blended tax rate for U.S. companies
- Document the adjustment and reason in a footnote
- If reported FCF is clean and consistent, use it directly

#### Interactive Sliders (4 controls)

| Control | Default | Range | Step | Purpose |
|---|---|---|---|---|
| WACC | 10% | 7–15% | 0.5 | Discount rate reflecting risk profile |
| Terminal Growth Rate | 3–4% | 2–6% | 0.5 | Long-run perpetuity growth |
| Forecast Horizon | 10 yrs | 5–15 | 1 | Explicit projection period |
| Scenario FCF Growth | 25% | 5–50% | 1 | User-defined scenario for the table |

All sliders use `accentColor: T.yellow`. Sliders update `useState` values which feed into both `calcImpliedGrowth` and `buildDCF` reactively.

#### Implied Growth Interpretation Logic — Dynamic Calibration

The implied growth thresholds and scenario growth rates must be **derived from the company's own historical and guided growth profile**, not hardcoded. A 25% implied growth rate is conservative for a hypergrowth SaaS company but heroic for a utility.

**Step 1 — Establish the company's growth context**

Before setting any thresholds, extract or calculate:

```js
const historicalCAGR    // 3-5yr revenue CAGR from the document (e.g. 0.34 for Axon)
const guidedGrowth      // midpoint of current year revenue guidance (e.g. 0.285 for Axon FY2026)
const analystGrowth     // consensus forward revenue growth if available via web search
const sectorMedian      // typical growth rate for this sector (search: "[sector] median revenue growth [year]")
```

**Step 2 — Derive scenario growth rates relative to the company's baseline**

```js
// Anchor: use guided growth midpoint as the base case
// If no guidance exists, use the average of historicalCAGR and analystGrowth
const baseG = guidedGrowth || (historicalCAGR + analystGrowth) / 2

// Bear: meaningful deceleration — roughly half the base case, floored at 2% above terminal growth
const bearG = Math.max(baseG * 0.5, termG / 100 + 0.02)

// Bull: modest acceleration above base, capped to avoid implausibility
// For high-growth companies (base > 25%): add ~10pp
// For moderate-growth companies (base 10-25%): add ~5pp
// For low-growth companies (base < 10%): add ~3pp
const bullG = baseG > 0.25 ? baseG + 0.10
            : baseG > 0.10 ? baseG + 0.05
            : baseG + 0.03

const scenarios = [
  { label: "Bear", g: Math.round(bearG * 100), color: T.red },
  { label: "Base", g: Math.round(baseG * 100), color: T.amber },
  { label: "Bull", g: Math.round(bullG * 100), color: T.green },
]
```


**Step 3 — Derive implied growth interpretation thresholds dynamically**

The colour and label for the implied growth rate are calibrated against the company's own guided growth, not fixed percentages:

```js
// If implied growth is below guided → market is being conservative → green (potential upside)
// If implied growth is near guided (within +5pp) → fairly priced → amber
// If implied growth significantly exceeds guided (>+5pp) → priced for perfection → red

function impliedGrowthColour(impliedG, baseG, T) {
  if (impliedG < baseG)        return T.green   // market pricing in less than guidance
  if (impliedG < baseG + 0.05) return T.amber   // market roughly in line with guidance
  return T.red                                   // market demanding growth above guidance
}

function impliedGrowthLabel(impliedG, baseG, historicalCAGR) {
  const headroom = impliedG - baseG
  if (impliedG < baseG)
    return "below current guidance — potential margin of safety"
  if (headroom < 0.03)
    return "broadly in line with guidance — fairly valued"
  if (headroom < 0.08)
    return "moderately above guidance — demands consistent execution"
  if (impliedG < historicalCAGR * 1.1)
    return "demanding but within historical range — high conviction required"
  return "well above guidance and historical rates — leaves no margin of safety"
}
```

**Step 4 — Surface the context in the interpretation panel**

The interpretation paragraph in the Reverse DCF tab should always include:

- The implied growth rate (coloured per Step 3)
- The company's guided growth rate as the explicit benchmark
- The company's historical CAGR as a second reference point
- The derived label from Step 3
- Whether the analyst consensus price target implies a higher or lower embedded growth rate than the current price

Example output text (generated, not hardcoded):

> *"At WACC 10% and terminal growth 4%, the market is embedding a **29.3% annual FCF growth** rate for the next 10 years. Axon guided revenue growth of 27–30% for FY2026 (midpoint: 28.5%), meaning the current price is broadly in line with guidance — fairly valued on a DCF basis. The company's 5-year revenue CAGR of 34% provides some historical support for this rate, though FCF growth will need to recover from the 2025 capex cycle to match it. The analyst median target of $802.50 implies a meaningfully higher embedded growth rate, suggesting the Street sees further acceleration ahead."*

#### Scenario Bar Chart

- Shows Bear / Base / Bull implied prices + Current price as bars
- `ReferenceLine y={SHARE_PRICE}` in yellow dashed = current price anchor
- Bars in T.blue

#### FCF Schedule Table

Columns: Year | FCF ($M) | PV of FCF ($M) | Cumulative PV ($M)

Final rows:
- Terminal Value row — yellow highlight
- Implied Share Price row — blue highlight, large font

Footnote must include:
- FCF base derivation formula
- Terminal value method (Gordon Growth Model)
- TV as % of total equity value: `Math.round(dcf.tv / dcf.equity * 100) + "%"`
- Diluted share count source

---

## Data Sourcing Rules

### Tiered Data Sourcing Strategy

Data should be gathered in priority order: **Document first → Web search to fill gaps → Web search to enrich context**. Never leave a data field blank if a web search can reasonably retrieve it.

---

### Tier 1 — Extract Directly from the Document

| Data Point | Source Location |
|---|---|
| Annual revenue (5 years) | Income statement / financial highlights slide |
| Quarterly segment revenue | Segment reporting table |
| Adjusted EBITDA & margin | Non-GAAP reconciliation table |
| Non-GAAP EPS | Non-GAAP reconciliation table |
| ARR / recurring revenue | KPI slide or statistical definitions appendix |
| Net Revenue Retention | Statistical definitions / KPI slide |
| Future contracted bookings / backlog | KPI slide or backlog disclosure |
| Gross margin by segment | Segment reporting |
| Free cash flow | Cash flow reconciliation table |
| Share count (diluted) | Non-GAAP EPS reconciliation (weighted avg diluted shares) |
| TAM figures | Market opportunity slide |
| Guidance (revenue, EBITDA, capex) | Guidance table / forward-looking statements section |
| Customer count / agency count | Overview or business metrics slide |
| Product pricing / ARPU | Subscription plan slide or product pages |

---

### Tier 2 — Always Retrieve via Web Search

These data points are either not present in investor decks or go stale quickly. Always search for them regardless of what the document contains.

| Data Point | Search Query Template | Why Search |
|---|---|---|
| Current share price | `"[TICKER] stock price [month year]"` | Decks never include live price |
| Market capitalisation | `"[TICKER] market cap [year]"` | Needed for Reverse DCF mktCap input |
| Diluted share count (latest) | `"[TICKER] diluted shares outstanding [quarter year]"` | May differ from deck if time has passed |
| Analyst consensus price target | `"[TICKER] analyst price target consensus [year]"` | Provides external valuation anchor for Reverse DCF tab |
| Analyst rating breakdown | `"[TICKER] buy hold sell ratings [year]"` | Buy/Hold/Sell counts for verdict context |
| 52-week high and low | `"[TICKER] 52 week high low"` | Contextualises current price in range |
| Post-earnings stock reaction | `"[TICKER] earnings reaction stock [date]"` | Critical if deck is from an earnings release date |
| Forward P/E ratio | `"[TICKER] forward PE ratio [year]"` | Valuation context alongside Reverse DCF |
| EV/Revenue or EV/EBITDA multiples | `"[TICKER] EV revenue EBITDA multiple [year]"` | Peer comparison anchor |
| Short interest | `"[TICKER] short interest percentage [year]"` | Risk tab input — signals market scepticism |

---

### Tier 3 — Contextual Enrichment via Web Search

Search for these when the document lacks sufficient context or when they would materially improve commentary quality.

| Data Point | When to Search | Search Query Template |
|---|---|---|
| Competitor landscape | When TAM or market position claims need validation | `"[Company] competitors [sector] [year]"` |
| Recent news / catalysts | When the deck is 1+ months old | `"[Company] news [month year]"` |
| Regulatory developments | When regulatory risk is flagged in the document | `"[sector] regulation [country] [year]"` |
| Customer win announcements | When customer growth metrics are cited | `"[Company] customer wins contracts [year]"` |
| Management changes | When governance is a risk factor | `"[Company] CEO CFO management [year]"` |
| Peer company multiples | When valuation needs benchmarking | `"[peer ticker] EV revenue multiple [year]"` |
| Industry growth rate | When TAM claims need validation | `"[sector] market size growth rate [year]"` |
| Government budget data | When customer base is public sector | `"[country] [agency type] technology budget [year]"` |

---

### Web Search Execution Rules

1. **Search before building the artifact** — do not start coding until all Tier 2 searches are complete. The share price and analyst consensus are inputs to the Reverse DCF model and must be known upfront.

2. **Use specific, short queries** — 2–5 words perform best. Avoid verbose queries. Bad: `"what is the current analyst consensus price target for Axon Enterprise stock in February 2026"`. Good: `"AXON analyst price target 2026"`.

3. **Prefer primary sources** — IR pages, SEC filings, Bloomberg, Reuters, Morningstar, Yahoo Finance, and official company newsrooms over aggregator sites or forums.

4. **Fetch full articles when needed** — if a `web_search` snippet is too brief (e.g. for an earnings call transcript or detailed analyst note), use `web_fetch` on the direct URL to retrieve the full content.

5. **Triangulate conflicting data** — if two sources give different share counts or prices, note the discrepancy, pick the most recent/authoritative source, and flag it in a footnote.

6. **Do not hallucinate missing data** — if a metric cannot be found in the document or via web search after 2 attempts, display `"N/A"` in the KPI card rather than estimating. Note the gap in a comment.

7. **Timestamp sensitive data** — for share price, analyst targets, and market cap, always note the date retrieved (e.g. "as of Feb 23, 2026") in the KPI sub-label.

8. **Re-search if the deck is old** — if the investor deck is more than 60 days old, treat all Tier 2 data as stale and refresh everything via search before building.

---

### Reverse DCF — Web Search Dependencies

The Reverse DCF tab has specific data requirements that must be sourced via web search if not in the document:

| Input | Source | Search Query |
|---|---|---|
| `SHARE_PRICE` | Live search — always | `"[TICKER] stock price close [date]"` |
| `SHARES_M` | Deck preferred; verify via search | `"[TICKER] diluted shares outstanding [quarter]"` |
| `FCF_BASE` | Deck preferred (EBITDA × (1 - tax)); verify tax rate | `"[Company] effective tax rate [year]"` |
| Analyst median target | Always search | `"[TICKER] analyst price target median [year]"` |
| Analyst rating breakdown | Always search | `"[TICKER] buy hold sell analyst ratings"` |

The `SHARE_PRICE` and analyst consensus target are displayed as KPI cards in the Reverse DCF tab and directly anchor the interpretation of whether the implied growth rate is demanding or reasonable.

---

### Search Output Integration

When web search results are used, integrate them as follows:

- **KPI sub-labels**: Include the data source date — e.g. `sub="As of Feb 23, 2026 (Yahoo Finance)"`
- **Verdict tab**: Analyst consensus and rating breakdown should appear in the verdict banner to support or challenge the internal view
- **Reverse DCF tab**: The analyst median price target is shown as a KPI card alongside the implied growth rate, creating a natural comparison between what the market is pricing in vs what analysts expect
- **Footnotes**: Any web-sourced figure used in a calculation (especially share price or share count) must be footnoted with source and date

---

## Commentary Standards

### Prose quality rules

- Every insight must be **anchored to a specific number** from the filing
- Avoid vague assertions ("strong growth") — always quantify ("33% YoY revenue growth on a $2.8B base")
- FCF divergence from EBITDA must always be called out with the % gap
- SBC must be flagged if it exceeds 15% of revenue (it masks true earnings)
- NRR above 120% is a tier-1 quality signal — explain why
- TAM figures must always be paired with current penetration % to contextualise the opportunity

### Verdict tone

- The verdict should read like a senior analyst's conviction note — not a balanced "pros and cons" list
- Take a clear stance: Bullish / Bearish / Neutral
- Paragraph 1: the affirmative case — why this is exceptional
- Paragraph 2: honest risk acknowledgment — what could break the thesis

### Risk writing standards

- Every HIGH risk must explain: the mechanism, the trigger, and the financial impact
- Never write a risk without a specific data point (e.g. "~75% of revenue from government agencies subject to non-appropriation clauses")
- Governance risks (material weakness, SBC, related party) should never be omitted if present in the filing

---

## Header Structure

```
[TICKER BADGE in yellow] | Company Name (TICKER) | Report Period
[BULLISH/BEARISH/NEUTRAL badge] | [Sector badge]
```

Badges:
- Verdict badge: green background with black text for BULLISH; red for BEARISH; amber for NEUTRAL
- Sector badge: blue border, blue text, semi-transparent blue background

---

## Layout Conventions

- Max content width: `1100px`, centred with `margin: "0 auto"`
- Page padding: `28px 24px`
- Card border radius: `12px` for major panels, `10px` for list items
- Section titles: `11px`, uppercase, `letterSpacing: "0.1em"`, `T.muted` colour, bottom border
- KPI grids: `repeat(3,1fr)` for 6-tile rows, `repeat(4,1fr)` for 4-tile rows
- Tab bar: horizontal scroll (`overflowX: "auto"`) to handle 7 tabs on mobile
- All tab buttons: `whiteSpace: "nowrap"`, active tab gets `borderBottom: "2px solid #facc15"`

---

## Footer

```
Built from [Company] [Report Period] · [Date] · Not investment advice
```

---

## Common Pitfalls to Avoid

| Pitfall | Fix |
|---|---|
| Template literals (backticks) | Use string concatenation always |
| `const` variables mutated in loops | Use `let` for all loop variables |
| Unicode characters in comments (→, –, ×) | Use plain ASCII only (`->`, `-`, `x`) |
| Arrow functions in `.map()` causing parser errors | Use `function(item) { return ...; }` syntax |
| Dual Y-axis charts missing `yAxisId` prop | Always add `yAxisId="left"` and `yAxisId="right"` |
| `ReferenceLine` without `y` prop | Always specify `y={value}` explicitly |
| Overly long single artifact causing truncation | Keep data arrays compact; use shorthand object notation |
| `localStorage` or `sessionStorage` | Never use — not supported in Claude artifacts |

---

## Example Companies This Works For

This skill is designed for any company with:
- Multi-year revenue and earnings history
- Subscription or recurring revenue metrics (ARR, NRR, backlog)
- Segment reporting
- Non-GAAP reconciliation tables
- Forward guidance
- A publicly traded share price

Ideal for: SaaS, defence tech, fintech, healthcare IT, industrial software, marketplace platforms, and any high-growth company with a complex product ecosystem.

---

*Skill authored from the Axon Enterprise Q4 2025 Investor Deck analysis — February 2026*
### Disclaimer: This serves as a boilerplate template, always verify
