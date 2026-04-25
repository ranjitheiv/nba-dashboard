# 🏀 NBA Stats Tracker

> A private, password-protected playoff analytics dashboard for same-game multi (SGM) betting strategy.

---

## What It Is

NBA Stats Tracker is a personal analytics tool built for the NBA Playoffs. It pulls box score data directly from ESPN, stores it in a Google Sheet, and surfaces it through a sleek web dashboard with statistical analysis, trend modelling, and a narrative-based projection engine.

The tool exists to solve one problem: building a profitable same-game multi requires 10–20+ correlated legs that all align to a single game narrative. Doing this manually is time-consuming and error-prone. This tool automates the data collection and provides the analytical framework to make those decisions faster and more confidently.

---

## How It Works

```
ESPN API → Google Apps Script → Google Sheets → Apps Script Web App → GitHub Pages Dashboard
```

1. **Import** — Paste an ESPN box score URL into the NBA Tools menu in Google Sheets. The Apps Script fetches the ESPN JSON API, formats every player's stats, and appends them as rows to the sheet.
2. **Serve** — The same Apps Script is deployed as a Web App that returns the sheet data as JSON when the dashboard calls it.
3. **Analyse** — The GitHub Pages dashboard fetches that JSON and renders it into interactive tables, charts, and projections — all in the browser, no server required.

---

## Features

### Stats Tracker (Main Dashboard)

- **Round + Team filters** populate from your imported data automatically
- **Team logo** displayed in the banner alongside the team name
- **vs. subheading** — opposing team shown below the team name for quick context
- **Flip button** — switches to the opposing team instantly
- **Paired categories** — selecting PTS always shows FGA alongside it; TPM always shows TPA
- **Stats table** — game-by-game breakdown across PTS, FGA, TPM, TPA, REB, AST, STL, BLK with blue gradient conditional formatting
- **Analysis table** — MIN / MAX / AVG / MED per player with current vs prior game comparison and VAR%
- **Gates table** — three-tier Over/Under lines (Conservative / Moderate / Aggressive) using the Gates algorithm
- **Focused view** — single player + category deep-dive with area chart, range strip, trend comparison, and fan chart

### Narrative Builder

Accessed via the **Narrative Builder** button on the team banner (only active once a round and team are selected).

Built around four NBA game narratives:

| Narrative | Description |
|---|---|
| **Home Blowout** | Home team dominates by 10–20+ pts |
| **Away Blowout** | Away team dominates by 10–20+ pts |
| **Grind** | Close, low-scoring defensive game |
| **Shootout** | Close, high-scoring open game |

**Setup flow:**
1. Select a narrative — Blowout tiles show the team logo. Step 3 header changes contextually: "How big is the blowout?" / "How defensive is the game?" / "How high scoring is the game?"
2. Mark key players — ↑ (green) for expected outperformers within the narrative, ↓ (red) for underperformers, unmarked for neutral
3. Set intensity — **Mild / Strong / Extreme** (default: Strong)
4. Hit Build Narrative

---

### SGM Builder

The SGM Builder is the core leg-building workspace inside the Narrative Builder output. It lets you construct your same-game multi category by category.

**Controls:**
- **Category pills** (PTS / FGA / TPM / TPA / REB / AST / STL / BLK) — switch stat category
- **Risk pills** (LOW / MED / HIGH) — maps to Conservative / Moderate / Aggressive Gates tiers
- **Player table** — all players from both teams sorted by highest series average in the selected category, showing:
  - Checkbox to select a leg
  - Player name + team abbreviation (colour coded) + position bubble
  - **Bet** input — type your chosen line threshold. Leave blank to use narrative projection
  - Over / Under lines (narrative-adjusted, Gates-buffered)
  - Conditional extra column — AVG FGA when PTS selected, AVG TPA when TPM selected
  - Game-by-game actual values with conditional formatting
  - Mini range sparkline (min/avg/med/max)

**Current Legs section** (collapsible, default collapsed):
- Numbered list of all selected legs showing: player · team · stat · risk badge · **bet line** · (series avg) · Ovr/Und lines
- **Build Bet button** (blue outlined) — when clicked, fills any blank bet inputs with the narrative projection for that player, then locks to solid "Bet Built ✓"
- **Clear All button** — resets all selections and the Build Bet state
- **Team Summary table** (right panel, 30% width) — after Build Bet is clicked:
  - Rows: PTS / TPM / REB / AST / STL / BLK
  - Columns: Home team · STAT · Away team (with logos in header)
  - Checked legs use your entered bet value
  - All other players contribute their narrative projection to the team totals
  - Each cell shows `total (series avg)` — the series average is the sum of each player's actual series average for that stat across the whole team
  - PTS TOTAL row at the bottom
  - Card styling with lighter body rows and darker header/footer

**How to use the SGM Builder:**
1. Select PTS → LOW risk → review the player table → check legs you want to back → type your bet line in the Bet column
2. Switch to REB → check rebounding legs → type bet lines
3. Repeat for any other categories
4. Open Current Legs — review your selections
5. Hit **Build Bet** — blank bet inputs are filled with narrative projections, the team summary fills in showing your totals vs the full team projection
6. Use the team summary to check you're not over-indexing on one category (e.g. too many PTS bets on one team)

### Player Projections

Full projected stat sheet for every player on both rosters:
- ↑ UP / ↓ DOWN direction badges for marked players
- Projected value vs series average in brackets `(avg)`
- Home/Away toggle + synced intensity dial (Mild/Strong/Extreme)
- Total PTS row showing sum of projections vs narrative score anchor

---

## Three Analytical Systems

| System | Where | What it does |
|---|---|---|
| **System 1 — Gates** | Stat Tracker | Pure data-driven Over/Under using trend-aware projection + standard deviation buffer |
| **System 2 — Player Projections** | Narrative Builder | Gates projection as anchor, adjusted by narrative × direction delta × intensity × minutes scale |
| **System 3 — SGM Builder** | Narrative Builder | Same narrative anchor as System 2, with Gates buffer applied for Over/Under window |

See `MATH_AND_LOGIC.md` for complete formulas, multipliers, and worked examples.

---

## Tech Stack

| Component | Technology |
|---|---|
| Data storage | Google Sheets |
| Data import | Google Apps Script |
| Data API | Google Apps Script Web App |
| Dashboard | Vanilla HTML + CSS + JavaScript |
| Hosting | GitHub Pages |
| Team logos | ESPN CDN (public URLs) |
| Fonts | Bebas Neue, Barlow Condensed, DM Mono |

No server. No database. No ongoing cost.

---

## Setup

### Step 1 — Apps Script

1. Open the Google Sheet → **Extensions → Apps Script**
2. Paste contents of `nba_boxscore_importer.gs` → Save
3. **Deploy → New deployment → Web App**
4. Execute as: **Me** · Who has access: **Anyone**
5. Deploy and copy the Web App URL

### Step 2 — Dashboard

1. Open `index.html` in a text editor
2. Set `const WEB_APP_URL = 'your_url_here'`
3. Set `const PASSWORD = 'letsgo'`
4. Save

### Step 3 — GitHub Pages

1. Create a new public repository
2. Upload `index.html` (must be named `index.html`)
3. **Settings → Pages → Branch: main → Save**
4. Live at `https://yourusername.github.io/repo-name/`

### Step 4 — Sheet sharing

Set Google Sheet to **"Anyone with the link can view"** under Share settings.

---

## Importing Box Scores

1. Open Google Sheet → **NBA Tools → Import Box Score**
2. Paste the ESPN URL (espn.com or espn.com.au both work)
3. Enter Round (1–4) and Game (1–7)
4. Click Import

---

## Google Sheet Column Structure

| Col | Field | Col | Field |
|---|---|---|---|
| A | Round | K | TPM |
| B | Game | L | TPA |
| C | Player | M | REB |
| D | Position | N | AST |
| E | Team | O | STL |
| F | # (Jersey) | P | BLK |
| G | MIN | Q | Home Team |
| H | PTS | R | Away Team |
| I | FGM | S | Home Score |
| J | FGA | T | Away Score |

---

## Files

| File | Purpose |
|---|---|
| `index.html` | The dashboard web app |
| `nba_boxscore_importer.gs` | Google Apps Script for import + data API |
| `README.md` | This file |
| `MATH_AND_LOGIC.md` | Complete formula and multiplier reference |
| `Narrative_Analyst_Prompt.pdf` | System prompt for a separate Claude chat acting as NBA Narrative Analyst |
| `NBA_Stats_Tracker_Documentation.pdf` | Internal documentation with screenshot placeholders |

---

## Companion Tool — Narrative Analyst

A separate Claude chat set up using `Narrative_Analyst_Prompt.pdf` acts as your NBA Narrative Analyst. Use it before building bets to:
- Research upcoming games via web search (injuries, form, home/away records, series context)
- Get a recommended narrative (Blowout / Grind / Shootout) with confidence level and reasoning
- Get player ↑/↓ suggestions with explanations
- Get intensity recommendation (Mild / Strong / Extreme)

Keep the Narrative Analyst chat separate from this repo's chat, which is reserved for dashboard and code changes only.

---

## License

Private project. Not licensed for redistribution.
