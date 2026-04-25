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

Built around four NBA game narratives:

| Narrative | Description |
|---|---|
| **Home Blowout** | Home team dominates by 10–20+ pts |
| **Away Blowout** | Away team dominates by 10–20+ pts |
| **Grind** | Close, low-scoring defensive game |
| **Shootout** | Close, high-scoring open game |

**Setup flow:**
1. Select a narrative — Blowout tiles show the team logo, intensity header changes contextually
2. Mark key players — ↑ (green) for expected outperformers within the narrative, ↓ (red) for underperformers, unmarked for neutral
3. Set intensity — **Mild / Strong / Extreme** (not Conservative/Moderate/Aggressive — those are for Gates risk tiers). Header changes per narrative: "How big is the blowout?" / "How defensive is the game?" / "How high scoring is the game?"
4. Hit Build Narrative

**SGM Builder output:**
- Category pills (PTS / FGA / TPM / TPA / REB / AST / STL / BLK)
- Risk tier pills (LOW / MED / HIGH) — these map to Gates Conservative/Moderate/Aggressive
- Player table showing Over/Under lines, game-by-game actual values, position bubble, and mini range sparkline for all players from both teams sorted by highest series average
- Checkbox leg selection with running count
- Current Selections section — numbered list of all selected legs with player, team, category, risk badge, and lines

**Player Projections output:**
- Full projected stat sheet for every player on both rosters
- ↑ UP / ↓ DOWN direction badges for marked players
- Projected value vs series average in brackets
- Home/Away toggle + synced intensity dial

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
3. Set `const PASSWORD = 'your_password_here'`
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

## License

Private project. Not licensed for redistribution.
