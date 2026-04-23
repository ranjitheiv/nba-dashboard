# 🏀 NBA Stats Tracker

> A private, password-protected playoff analytics dashboard for same-game multi (SGM) betting strategy.

---

## What It Is

NBA Stats Tracker is a personal analytics tool built for the NBA Playoffs. It pulls box score data directly from ESPN, stores it in a Google Sheet, and surfaces it through a sleek web dashboard with statistical analysis, trend modelling, and a narrative-based projection engine.

The tool exists to solve one problem: building a profitable same-game multi requires 10–20+ correlated legs that all align to a single game narrative. Doing this manually is time-consuming and error-prone. This tool automates the data collection and provides the analytical framework to make those decisions faster and more confidently.

---

## Features

### Stats Tracker (Main Dashboard)

- **Round + Team filters** populate from your imported data automatically
- **Flip button** switches to the opposing team instantly
- **Stats table** — game-by-game breakdown for every player across PTS, FGA, TPM, TPA, REB, AST, STL, BLK with a blue gradient conditional formatting heat-map
- **Analysis table** — MIN / MAX / AVG / MED per player per stat for the selected round
- **Gates table** — three-tier Over/Under projection lines (Conservative / Moderate / Aggressive) using the Gates algorithm
- **Focused view** — select a single player + category to see a deep-dive with area chart, range strip, trend comparison table, and fan chart

### Narrative Builder

A prediction engine built around four NBA game narratives:

| Narrative | Description |
|---|---|
| **Home Blowout** | Home team wins by 10–20+ pts. Stars shine, loser's stats crater. |
| **Away Blowout** | Mirror of Home Blowout. |
| **Grind** | Close, low-scoring game. Strong defence, high rebounds, low 3PT. |
| **Shootout** | Close, high-scoring game. Open looks, high assists, low rebounds. |

For each narrative you:
1. Select the game script you predict
2. Pick up to 3 key players per team you expect to have big games
3. Set the intensity dial (Conservative / Moderate / Aggressive)
4. Get a full projected stat sheet for every player, ranked SGM legs, team summary, and radar chart

### Gates Algorithm

A trend-adjusted projection model using:
- **Linear regression** on the series game sequence to detect momentum
- **Standard deviation** to measure consistency and set buffer width
- **Data-aware minimum floor** — wide lines early in the series, tightens as games accumulate
- **Three tier multipliers** — Conservative (2.0× buffer), Moderate (1.3×), Aggressive (0.25×)
- All values rounded up to nearest integer (no half-points in basketball)

---

## Privacy & Access

The dashboard is password-protected. Anyone who visits the GitHub Pages URL sees a lock screen and cannot access any data without the password. The password is stored in `index.html` — change it by editing the `PASSWORD` constant and re-uploading to GitHub.

The GitHub repository itself is public (required for free GitHub Pages), so the source code is visible. The data is protected by the Apps Script Web App URL, which is embedded in the HTML. For additional security, consider GitHub Pro ($4/month) which allows private repositories with Pages support.

---

## Documentation

Full internal documentation including:
- Screenshot walkthroughs of every section
- Complete Gates algorithm technical reference
- Narrative Builder multiplier matrix and rationale
- Multiplier changelog

See `NBA_Stats_Tracker_Documentation.pdf` in the repository.

---

## License

Private project. Not licensed for redistribution.
