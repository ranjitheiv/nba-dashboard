# NBA Stats Tracker — Math & Logic Reference

> Complete formula, multiplier, and calculation reference for all three analytical systems. Last updated: April 2026.

---

## Overview

The dashboard uses three separate calculation systems. They share some helper functions but are independent — changing the narrative has no effect on System 1, and System 1 never uses narrative data.

| System | Where | Inputs |
|---|---|---|
| **System 1 — Gates** | Stat Tracker: Gates table + Focused View | Player's actual game data only |
| **System 2 — Player Projections** | Narrative Builder: Player Projections table | Gates projection + narrative + player direction + dial + minutes |
| **System 3 — SGM Builder** | Narrative Builder: SGM Builder Over/Under | Same anchor as System 2 + Gates buffer |

---

## System 1 — Gates

*Used by: Gates table on main dashboard · Focused View Gates card*
*No narrative, no dial, no multipliers — pure series data only*

### Formula

```
1. mean       = average of played games this round (DNP excluded)
2. slope      = linear regression slope across the game sequence
3. projection = mean + (slope × 0.4)
4. std_dev    = standard deviation of played games
5. data_floor = max(1.0, 5.0 − (games_played − 1))
6. buffer     = max(data_floor, std_dev × 0.65)
7. Over       = ceil(max(0, projection − buffer × tier_mult))
   Under      = ceil(projection + buffer × tier_mult)
```

### Why slope × 0.4

The slope captures the trajectory of the player's recent games. Multiplying by 0.4 gives the trend a modest influence without over-reacting to short runs. A player going 20, 25, 30 has a strong upward slope — the projection nudges upward but doesn't extrapolate blindly.

### Tier Multipliers (fixed — no dial)

| Tier | Mult | Window at buffer=3.0 |
|---|---|---|
| Conservative | 1.80× | ±5.4 pts (10.8 pt window) |
| Moderate | 1.30× | ±3.9 pts (7.8 pt window) |
| Aggressive | 0.70× | ±2.1 pts (4.2 pt window) |

### Data Floor

Prevents false confidence when only 1–2 games have been played. Standard deviation is mathematically unreliable on small samples.

| Games played | Floor | Reason |
|---|---|---|
| 1 | 5.0 | Single data point — maximum uncertainty |
| 2 | 4.0 | Very thin sample |
| 3 | 3.0 | Pattern starting to form |
| 4 | 2.0 | Reasonable sample |
| 5+ | 1.0 | Real data fully drives the buffer |

```
buffer = max(data_floor, std_dev × 0.65)
```

The buffer takes whichever is larger — the floor or the real standard deviation. By game 5, the floor drops to 1.0 and the actual series data fully drives the window.

### All values rounded up (ceil)

There are no half-points in basketball. All Over/Under lines are rounded up to the nearest integer.

---

## System 2 — Narrative Builder Player Projections

*Used by: Player Projections table in the Narrative Builder*

### Formula

```
Step 1 — Gates projection (trend-aware anchor)
  mean       = average of played games this round
  slope      = linear regression slope
  gates_proj = mean + (slope × 0.4)

Step 2 — Minutes scale
  avg_min >= 30  →  min_scale = 1.0   (full narrative effect)
  avg_min 18–29  →  min_scale = 0.6   (partial)
  avg_min < 18   →  min_scale = 0.2   (minimal — bench/spot minutes)

Step 3 — Base multiplier (AND logic — direction delta applies ON TOP of narrative)
  base_mult = narrative_mult[stat] × direction_delta[stat]
  (direction_delta = 1.0 if player is unmarked)

Step 4 — Narrative-adjusted anchor
  anchor = gates_proj × (1.0 + (base_mult − 1.0) × dial_scale × min_scale)

Step 5 — Project
  projected = ceil(anchor)
```

### Why AND (not OR) for direction delta

Previously the star boost replaced the narrative multiplier entirely (OR logic), which caused a player selected as UP in a Grind to be projected *above* their average — ignoring that it's a low-scoring game. With AND logic, the direction delta is a small modifier on top of the narrative, so context is always preserved. An UP player in a Grind still scores less than their average — just less suppressed than an unselected player.

### Why gates_proj instead of raw average

The Gates projection already accounts for momentum via the slope. Using the raw average discards trend information. A player scoring 20, 25, 30 across three games has an average of 25 but a Gates projection of 28+. The narrative adjustment is applied on top of an already-intelligent baseline.

### Narrative Builder Dial Scale

Controls how strongly you believe the narrative will play out. Shifts the anchor — does not affect buffer width.

| Dial | Scale | Meaning |
|---|---|---|
| Conservative | 0.7× | Narrative will partially play out |
| Moderate | 1.0× | Standard belief in the narrative |
| Aggressive | 1.3× | High conviction — narrative plays out strongly |

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

### Direction Deltas

Applied on top of the narrative multiplier (AND logic). Small adjustments — not replacements.

**↑ UP delta** — player expected to outperform narrative:

| Stat | UP delta | Condition |
|---|---|---|
| PTS | 1.08 | All |
| FGA | 1.06 | All |
| AST | 0.94 | Shooting more = fewer assists |
| REB | 1.05 | All |
| STL | 1.05 | All |
| BLK | 1.05 | All |
| TPM | 1.10 | avg TPA ≥ 3 (high-volume shooter) |
| TPM | 1.04 | avg TPA < 3 |
| TPA | 1.08 | avg TPA ≥ 3 |
| TPA | 1.03 | avg TPA < 3 |

**↓ DOWN delta** — player expected to underperform narrative:

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

**Unmarked player** — direction delta = 1.0 (plain narrative applies)

### Minutes Scale Rationale

Playoff rotations tighten significantly. The narrative affects a 35-minute player far more than a 12-minute bench player. The minutes scale weights the narrative adjustment proportionally to actual court time.

| avg MIN | Scale | Effect |
|---|---|---|
| 30+ | 1.0 | Full narrative adjustment |
| 18–29 | 0.6 | Partial adjustment |
| < 18 | 0.2 | Minimal — narrative barely applies |

### Hard Caps

STL and BLK are capped at 0 minimum and 4 maximum per player regardless of multiplier. No player can be projected to record more than 4 steals or blocks in a single game.

---

## System 3 — SGM Builder Over/Under Lines

*Used by: SGM Builder table (LOW / MED / HIGH risk)*

Combines the narrative-adjusted anchor from System 2 with the Gates buffer from System 1. The centre of the Over/Under window shifts with the narrative and player direction. The window width is driven by actual series data.

### Formula

```
Step 1 — Gates projection
  gates_proj = mean + (slope × 0.4)

Step 2 — Minutes scale (same as System 2)
  avg_min >= 30  →  min_scale = 1.0
  avg_min 18–29  →  min_scale = 0.6
  avg_min < 18   →  min_scale = 0.2

Step 3 — Base multiplier (AND logic — same as System 2)
  base_mult = narrative_mult[stat] × direction_delta[stat]

Step 4 — Narrative-adjusted anchor
  anchor = gates_proj × (1.0 + (base_mult − 1.0) × dial_scale × min_scale)

Step 5 — Buffer (data-driven — same as System 1)
  std_dev    = standard deviation of played games
  data_floor = max(1.0, 5.0 − (games_played − 1))
  buffer     = max(data_floor, std_dev × 0.65)

Step 6 — Over/Under lines
  Over  = ceil(max(0, anchor − buffer × tier_mult))
  Under = ceil(anchor + buffer × tier_mult)
```

### SGM Risk Tier Multipliers

| SGM Risk | Tier | Mult | Window at buffer=3.0 |
|---|---|---|---|
| LOW | Conservative | 1.80× | ±5.4 pts |
| MED | Moderate | 1.30× | ±3.9 pts |
| HIGH | Aggressive | 0.70× | ±2.1 pts |

### Key distinction from System 1

In System 1, the centre of the window is the raw Gates projection. In System 3, the centre is the narrative-adjusted anchor — so in a Shootout, lines shift upward; in a Grind, they shift downward. The buffer calculation is identical in both systems.

### The dial in System 3

The dial_scale used in Step 4 is the Narrative Builder intensity dial (0.7 / 1.0 / 1.3). Changing the dial shifts the anchor for every player simultaneously, which shifts all Over/Under lines in the SGM Builder. The buffer width does not change with the dial.

---

## Worked Examples

### Example 1 — Jalen Brunson, Grind, Moderate dial, marked ↑ UP

*avg MIN=37 · games=3 · mean=27.7 · slope=+1.0 · std_dev=3.2*

```
System 2 (Player Projection):
  gates_proj    = 27.7 + (1.0 × 0.4) = 28.1
  min_scale     = 1.0   (37 mins ≥ 30)
  narrative PTS = 0.90  (Grind — suppressed)
  UP delta PTS  = 1.08
  base_mult     = 0.90 × 1.08 = 0.972
  dial_scale    = 1.0   (Moderate)
  anchor        = 28.1 × (1.0 + (0.972−1.0) × 1.0 × 1.0)
                = 28.1 × 0.972 = 27.3
  projected     = ceil(27.3) = 28

System 3 (SGM Builder LOW risk):
  data_floor    = max(1.0, 5.0−2) = 3.0
  buffer        = max(3.0, 3.2 × 0.65) = 3.0
  Over          = ceil(27.3 − 3.0 × 1.80) = ceil(21.9) = 22
  Under         = ceil(27.3 + 3.0 × 1.80) = ceil(32.7) = 33
```

Compare to Brunson **unmarked** in Grind Moderate:
```
  base_mult = 0.90 (narrative only, delta = 1.0)
  anchor    = 28.1 × 0.90 = 25.3
  projected = 26
  LOW Over  = ceil(25.3 − 5.4) = 20
  LOW Under = ceil(25.3 + 5.4) = 31
```

The ↑ selection gives him slightly higher numbers than an unmarked player, but still suppressed by the Grind — which is the correct behaviour.

### Example 2 — Brunson marked ↓ DOWN in Grind Moderate

```
  DOWN delta PTS = 0.88
  base_mult      = 0.90 × 0.88 = 0.792
  anchor         = 28.1 × (1.0 + (0.792−1.0) × 1.0 × 1.0)
                 = 28.1 × 0.792 = 22.3
  projected      = 23
  LOW Over       = ceil(22.3 − 5.4) = 17
  LOW Under      = ceil(22.3 + 5.4) = 28
```

### Example 3 — Brunson in Shootout, Aggressive dial, marked ↑ UP

```
  narrative PTS  = 1.10  (Shootout)
  UP delta PTS   = 1.08
  base_mult      = 1.10 × 1.08 = 1.188
  dial_scale     = 1.3   (Aggressive)
  anchor         = 28.1 × (1.0 + (1.188−1.0) × 1.3 × 1.0)
                 = 28.1 × 1.244 = 35.0
  projected      = 35
  LOW Over  = ceil(35.0 − 5.4) = 30
  LOW Under = ceil(35.0 + 5.4) = 41
  HIGH Over = ceil(35.0 − 2.1) = 33
  HIGH Under= ceil(35.0 + 2.1) = 38
```

### Example 4 — Bench player, 14 avg MIN, unmarked, Grind Moderate

*mean=8.0 · slope=0.5 · std_dev=2.1*

```
  gates_proj = 8.0 + (0.5 × 0.4) = 8.2
  min_scale  = 0.2  (14 mins < 18)
  base_mult  = 0.90 (narrative Grind PTS, delta=1.0)
  anchor     = 8.2 × (1.0 + (0.90−1.0) × 1.0 × 0.2)
             = 8.2 × 0.98 = 8.0
  projected  = 9
  data_floor = 3.0
  buffer     = max(3.0, 2.1 × 0.65) = 3.0
  LOW Over   = ceil(8.0 − 5.4) = 3
  LOW Under  = ceil(8.0 + 5.4) = 14
```

The narrative barely affects this player — their lines are almost identical to what they'd be without any narrative.

---

## Summary — Key Design Decisions

| Decision | Rationale |
|---|---|
| Slope × 0.4 as projection | Captures momentum without over-extrapolating short runs |
| Data floor decreasing by game | Prevents false confidence on 1–2 game samples |
| AND logic for direction delta | Preserves game script context — UP in Grind still suppressed, just less so |
| Small direction deltas (1.08 max) | Avoids compounding — direction is a modest adjustment, not an override |
| Minutes scale | Bench players shouldn't feel the full narrative regardless of game script |
| Gates projection as anchor (not raw avg) | Uses trend-aware baseline across all three systems |
| Dial affects anchor only | Intensity of narrative belief shifts the centre — not the uncertainty width |
| Tier mult affects buffer width only | Risk tolerance on the bet is separate from belief in the narrative |
| STL/BLK hard capped at 4 | Individual players cannot realistically record more than 4 steals or blocks |
| All values ceil() | No half-points in basketball |
| Same tier mults in System 1 and System 3 | Consistent risk interpretation across the whole dashboard |

---

## Multiplier Changelog

| Date | System | Change | Reason |
|---|---|---|---|
| Apr 2026 | All | Initial implementation | — |
| Apr 2026 | Narrative matrix | All mults reduced (e.g. PTS 1.15→1.10) | Gates projection is trend-aware, so smaller adjustments needed |
| Apr 2026 | Logic | OR replaced with AND for star/narrative | OR ignored game script; AND preserves narrative context |
| Apr 2026 | Logic | Star boost → direction delta (UP/DOWN) | More precise — can now mark underperformers, not just outperformers |
| Apr 2026 | Tier mults | 2.0/1.25/0.50 → 1.80/1.30/0.70 | Moderate was too narrow; Aggressive at 0.50 produced unusably tight lines |
| Apr 2026 | Dial scale | Aggressive 1.5→1.3 | 1.5 was compounding too aggressively with the narrative mults |

*Add new rows as you adjust values during the season.*

---

*For implementation questions refer to the source code in `index.html` and `nba_boxscore_importer.gs`*
