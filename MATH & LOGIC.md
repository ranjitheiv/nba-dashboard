# NBA Stats Tracker — Math & Logic Reference

> Complete formula, multiplier, and calculation reference for all three analytical systems. Last updated: April 2026.

---

## Overview

| System | Where | Inputs |
|---|---|---|
| **System 1 — Gates** | Stat Tracker: Gates table + Focused View | Player's actual game data only |
| **System 2 — Player Projections** | Narrative Builder: Player Projections table | Gates projection + narrative + direction + intensity + minutes |
| **System 3 — SGM Builder** | Narrative Builder: SGM Builder Over/Under | Same anchor as System 2 + Gates buffer |

---

## System 1 — Gates

*Used by: Gates table on main dashboard · Focused View Gates card*
*No narrative, no dial — pure series data only*

### Formula

```
1. mean        = average of played games this round (DNP excluded)
2. slope       = linear regression slope across the game sequence
3. slope_wt    = slope weight based on games played (see table below)
4. projection  = mean + (slope × slope_wt)
5. std_dev     = standard deviation of played games
6. data_floor  = max(1.0, 5.0 − (games_played − 1))
7. buffer      = max(data_floor, std_dev × 0.65)
8. Over        = ceil(max(0, projection − buffer × tier_mult))
   Under       = ceil(projection + buffer × tier_mult)
```

### Slope Weight by Games Played

The slope captures trajectory, but with only 1–3 games it is unreliable noise rather than signal. The slope weight ramps up gradually as real data accumulates.

| Games played | Slope weight | Reason |
|---|---|---|
| 1 | 0.0 | Single point — no trend possible |
| 2 | 0.0 | One jump could be a fluke |
| 3 | 0.0 | Still too few to trust a slope |
| 4 | 0.2 | Trend starting to form |
| 5 | 0.3 | More data, more weight |
| 6+ | 0.4 | Full weight — sufficient series data |

**Why this matters:** A player scoring 26, 36 across two games has a slope of +10. Under the old approach (fixed 0.4 weight) this produced a projection of 35 — treating one big game as a confirmed upward trend. With slope_wt=0.0 at 2 games, the projection is simply the mean (31), which is far more honest.

### Tier Multipliers (fixed — no dial)

| Tier | Mult | Window at buffer=4.0 |
|---|---|---|
| Conservative | 2.50× | ±10.0 pts (20 pt window) |
| Moderate | 1.80× | ±7.2 pts (14.4 pt window) |
| Aggressive | 1.10× | ±4.4 pts (8.8 pt window) |

Windows naturally tighten as more games are played and the buffer shrinks:

| Games | Typical buffer | Conservative window | Moderate window | Aggressive window |
|---|---|---|---|---|
| 2 | 4.0 | 20 pts | 14 pts | 9 pts |
| 3 | 3.0 | 15 pts | 11 pts | 7 pts |
| 4 | 2.5 | 13 pts | 9 pts | 6 pts |
| 5 | 2.0 | 10 pts | 7 pts | 4 pts |
| 6 | 1.95 | 10 pts | 7 pts | 4 pts |

### Data Floor

| Games | Floor |
|---|---|
| 1 | 5.0 |
| 2 | 4.0 |
| 3 | 3.0 |
| 4 | 2.0 |
| 5+ | 1.0 |

```
buffer = max(data_floor, std_dev × 0.65)
```

### Worked Example — Jaylen Brown, GM1=26, GM2=36

```
mean       = (26 + 36) / 2 = 31.0
slope      = 10.0  (large — one big game)
slope_wt   = 0.0   (only 2 games)
projection = 31.0 + (10.0 × 0.0) = 31.0  ← pure mean, ignores slope

std_dev    = 5.0
data_floor = max(1.0, 5.0 − 1) = 4.0
buffer     = max(4.0, 5.0 × 0.65) = 4.0

Conservative (2.50×): Over = ceil(31 − 10.0) = 21  / Under = ceil(31 + 10.0) = 41
Moderate     (1.80×): Over = ceil(31 − 7.2)  = 24  / Under = ceil(31 + 7.2)  = 39
Aggressive   (1.10×): Over = ceil(31 − 4.4)  = 27  / Under = ceil(31 + 4.4)  = 36
```

---

## System 2 — Narrative Builder Player Projections

*Used by: Player Projections table*

### Formula

```
Step 1 — Gates projection (trend-aware anchor)
  gates_proj = mean + (slope × slope_wt)

Step 2 — Minutes scale
  avg_min >= 30  →  min_scale = 1.0
  avg_min 18–29  →  min_scale = 0.6
  avg_min < 18   →  min_scale = 0.2

Step 3 — Base multiplier (AND logic)
  base_mult = narrative_mult[stat] × direction_delta[stat]
  (direction_delta = 1.0 if player is unmarked)

Step 4 — Narrative-adjusted anchor
  anchor = gates_proj × (1.0 + (base_mult − 1.0) × intensity_scale × min_scale)

Step 5 — Project
  projected = ceil(anchor)
```

### Intensity Dial Scale

Controls how strongly the narrative shifts the anchor. Different vocabulary from Gates tiers to avoid confusion — these are about how extreme you think the game script will be.

| Dial | Scale | Blowout meaning | Grind meaning | Shootout meaning |
|---|---|---|---|---|
| Mild | 0.7× | ~8 pt margin | Slightly defensive | Both teams scoring well |
| Strong | 1.0× | ~15 pt margin | Low scoring, physical | High pace, open looks |
| Extreme | 1.3× | 20+ pt margin | Lockdown battle | Both teams 120+ |

### Narrative Matrix

| Stat | BLOW-A Win | BLOW-A Lose | BLOW-B Win | BLOW-B Lose | GRIND | SHOOT |
|---|---|---|---|---|---|---|
| PTS | 1.10 | 0.90 | 0.90 | 1.10 | 0.90 | 1.10 |
| FGA | 1.08 | 0.95 | 0.95 | 1.08 | 0.95 | 1.08 |
| TPM | 1.12 | 0.90 | 0.90 | 1.12 | 0.90 | 1.20 |
| TPA | 1.12 | 0.95 | 0.95 | 1.12 | 0.95 | 1.15 |
| REB | 1.10 | 0.92 | 0.92 | 1.10 | 1.10 | 0.90 |
| AST | 0.90 | 0.90 | 0.90 | 0.90 | 0.90 | 1.10 |
| STL | 1.12 | 0.90 | 0.90 | 1.12 | 1.10 | 0.85 |
| BLK | 1.10 | 0.92 | 0.92 | 1.10 | 1.10 | 0.85 |

BLOW-B is the exact mirror of BLOW-A.

### Direction Deltas (AND logic — multiplies on top of narrative)

**↑ UP delta** — player expected to outperform within the narrative:

| Stat | UP delta | Condition |
|---|---|---|
| PTS | 1.08 | All |
| FGA | 1.06 | All |
| AST | 0.94 | Shooting more = fewer assists |
| REB | 1.05 | All |
| STL | 1.05 | All |
| BLK | 1.05 | All |
| TPM | 1.10 | avg TPA ≥ 3 |
| TPM | 1.04 | avg TPA < 3 |
| TPA | 1.08 | avg TPA ≥ 3 |
| TPA | 1.03 | avg TPA < 3 |

**↓ DOWN delta** — player expected to underperform within the narrative:

| Stat | DOWN delta | Condition |
|---|---|---|
| PTS | 0.88 | All |
| FGA | 0.90 | All |
| AST | 1.05 | Shooting less = more passing |
| REB | 0.92 | All |
| STL | 0.90 | All |
| BLK | 0.90 | All |
| TPM | 0.85 | All |
| TPA | 0.88 | All |

**Unmarked** — direction_delta = 1.0, plain narrative applies.

### Why AND not OR

Previously star boost replaced the narrative (OR logic), causing a player selected ↑ in a Grind to be projected above their average — ignoring the low-scoring game script. With AND logic, the direction delta is a small modifier on top of the narrative. An ↑ player in a Grind still scores less than their average — just slightly less suppressed than an unmarked player.

### Minutes Scale

| avg MIN | Scale |
|---|---|
| 30+ | 1.0 |
| 18–29 | 0.6 |
| < 18 | 0.2 |

Bench players get minimal narrative adjustment regardless of game script.

### Hard Caps

STL and BLK: floor 0, ceiling 4.

---

## System 3 — SGM Builder Over/Under Lines

*Used by: SGM Builder table (LOW / MED / HIGH)*

Combines the narrative-adjusted anchor from System 2 with the Gates buffer from System 1.

### Formula

```
Step 1–4: Same as System 2 to calculate anchor

Step 5 — Buffer (data-driven)
  std_dev    = standard deviation of played games
  data_floor = max(1.0, 5.0 − (games_played − 1))
  buffer     = max(data_floor, std_dev × 0.65)

Step 6 — Over/Under
  Over  = ceil(max(0, anchor − buffer × tier_mult))
  Under = ceil(anchor + buffer × tier_mult)
```

### SGM Risk Tier Multipliers

Same multipliers as System 1 — consistent risk interpretation across the whole dashboard.

| SGM Risk | Tier | Mult |
|---|---|---|
| LOW | Conservative | 2.50× |
| MED | Moderate | 1.80× |
| HIGH | Aggressive | 1.10× |

The intensity dial (Mild/Strong/Extreme) shifts the anchor centre. The risk tier (LOW/MED/HIGH) sets the window width around it. These are two separate decisions.

---

## Worked Example — Full System 3 Calculation

**Jaylen Brown, Grind narrative, Strong intensity, marked ↑ UP**
*avg MIN=38 · games=2 · mean=31 · slope=10 · std_dev=5.0*

```
Step 1 — Gates projection
  slope_wt   = 0.0  (2 games)
  gates_proj = 31 + (10 × 0.0) = 31.0

Step 2 — Minutes scale
  min_scale  = 1.0  (38 mins)

Step 3 — Base multiplier
  narrative PTS Grind = 0.90
  UP delta PTS        = 1.08
  base_mult           = 0.90 × 1.08 = 0.972

Step 4 — Anchor
  intensity_scale = 1.0  (Strong)
  anchor = 31.0 × (1.0 + (0.972−1.0) × 1.0 × 1.0)
         = 31.0 × 0.972 = 30.1

System 2 projected = ceil(30.1) = 31

Step 5 — Buffer
  data_floor = max(1.0, 5.0−1) = 4.0
  buffer     = max(4.0, 5.0×0.65) = 4.0

Step 6 — SGM lines
  LOW  Over = ceil(30.1 − 4.0×2.50) = ceil(20.1) = 21
  LOW  Under= ceil(30.1 + 4.0×2.50) = ceil(40.1) = 41
  MED  Over = ceil(30.1 − 4.0×1.80) = ceil(22.9) = 23
  MED  Under= ceil(30.1 + 4.0×1.80) = ceil(37.3) = 38
  HIGH Over = ceil(30.1 − 4.0×1.10) = ceil(25.7) = 26
  HIGH Under= ceil(30.1 + 4.0×1.10) = ceil(34.5) = 35
```

Compare to Brown **unmarked** in Grind Strong:
```
  base_mult = 0.90 (narrative only, delta=1.0)
  anchor    = 31.0 × 0.90 = 27.9
  LOW Over  = ceil(27.9 − 10.0) = 18
  LOW Under = ceil(27.9 + 10.0) = 38
```

The ↑ selection gives him higher lines than an unmarked player, but still suppressed by the Grind — correct behaviour.

---

## Summary — Key Design Decisions

| Decision | Rationale |
|---|---|
| Slope weight 0 for GM1-3 | Prevents noise being interpreted as signal. A single big game at GM2 has a slope of +10 — meaningless with only 2 data points. |
| Slope weight ramps 0→0.4 by GM6 | Trend becomes reliable as more games accumulate. By game 6 you have genuine signal. |
| Tier mults 2.50/1.80/1.10 | Produces clean 20/14/9 pt windows at buffer=4.0. Each tier is meaningfully different. Windows naturally compress as the series progresses. |
| AND logic for direction deltas | Preserves narrative context. ↑ in a Grind is still suppressed — just less so than unmarked. |
| Small direction deltas (1.08 max) | One big game is a nudge, not an override. Keeps projections realistic. |
| Minutes scale | Bench players shouldn't feel the full narrative. A 14-minute player is barely affected. |
| Mild/Strong/Extreme for intensity | Completely different vocabulary from Conservative/Moderate/Aggressive Gates tiers — avoids confusion between two separate concepts. |
| Gates projection as anchor | Uses trend-aware baseline. Better than flat average which ignores momentum. |
| Dial shifts anchor, tier shifts width | Two separate decisions: how extreme the game script, and how much risk on the bet. |
| STL/BLK capped at 4 | No player realistically records more than 4 steals or blocks. |
| All values ceil() | No half-points in basketball. |

---

## Multiplier Changelog

| Date | Change | Reason |
|---|---|---|
| Apr 2026 | Initial implementation | — |
| Apr 2026 | Narrative matrix mults reduced (e.g. PTS 1.15→1.10) | Gates projection is trend-aware, so smaller adjustments needed |
| Apr 2026 | OR logic → AND logic for direction | OR ignored game script; AND preserves narrative context |
| Apr 2026 | Star boost → direction delta (↑/↓) | Can now mark underperformers, not just outperformers |
| Apr 2026 | Tier mults 2.0/1.25/0.50 → 2.5/1.8/1.1 | Wider windows, more useful. Aggressive at 0.50 was too tight. |
| Apr 2026 | Slope weight: fixed 0.4 → tiered 0/0/0/0.2/0.3/0.4 | Prevents 2-game noise inflating projections. Brown 26→36 was projecting at 35 instead of 31. |
| Apr 2026 | Intensity dial: Conservative/Moderate/Aggressive → Mild/Strong/Extreme | Avoids confusion with Gates tier language. Different concept, different words. |
| Apr 2026 | Aggressive dial scale: 1.5→1.3 | 1.5 was too aggressive in combination with narrative mults |

*Add new rows as you adjust values during the season.*

---

*For implementation details refer to `index.html` and `nba_boxscore_importer.gs`*
