# NBA Stats Tracker — Math & Logic Reference

> This document explains every formula, multiplier, and calculation used in the dashboard. It covers all three systems: the Gates model on the Stat Tracker page, the Narrative Builder player projections, and the SGM Builder over/under lines.

---

## Overview of the Three Systems

| System | Where | What it does |
|---|---|---|
| **System 1 — Gates** | Stat Tracker page (Gates table + Focused View) | Pure data-driven Over/Under lines using trend + standard deviation. No narrative. |
| **System 2 — Player Projections** | Narrative Builder → Player Projections table | Projects each player's stat using a narrative-adjusted anchor. Accounts for game script, key player selection, intensity dial, and playing time. |
| **System 3 — SGM Builder** | Narrative Builder → SGM Builder table | Combines the narrative-adjusted anchor from System 2 with the Gates buffer from System 1 to produce Over/Under lines for leg selection. |

The three systems are independent. System 1 never uses narrative data. Systems 2 and 3 share the same anchor calculation but System 3 adds the Gates buffer on top of it.

---

## System 1 — Gates (Stat Tracker Page)

**Used by:** Gates table on the main dashboard · Focused View Gates card

**Data source:** Player's actual game data from the current round only. DNP games excluded.

### Formula

```
1. mean       = average of all played games this round
2. slope      = linear regression slope across the game sequence
3. projection = mean + (slope × 0.4)
4. std_dev    = standard deviation of played games
5. data_floor = max(1.0, 5.0 − (games_played − 1))
6. buffer     = max(data_floor, std_dev × 0.65)
7. Over       = ceil(max(0, projection − buffer × tier_mult))
   Under      = ceil(projection + buffer × tier_mult)
```

### Why slope × 0.4?

The slope captures the trajectory of the player's recent games. Multiplying by 0.4 gives the trend a modest influence on the projection without over-reacting to short runs. A player going 20, 25, 30 has a strong upward slope — the projection nudges upward but doesn't extrapolate blindly.

### Tier Multipliers (fixed — no dial)

| Tier | Mult | Effect |
|---|---|---|
| Conservative | 1.80× | Wider window — easier lines to beat, lower risk |
| Moderate | 1.25× | Balanced — standard uncertainty |
| Aggressive | 0.50× | Tight window — lines close to projection, higher value |

### Data Floor

The data floor prevents the algorithm from being falsely confident on thin data. With only 1–2 games the standard deviation is mathematically unreliable, so a minimum buffer is enforced regardless of how consistent the player appears.

| Games played | Floor |
|---|---|
| 1 | 5.0 |
| 2 | 4.0 |
| 3 | 3.0 |
| 4 | 2.0 |
| 5+ | 1.0 |

The buffer takes whichever is larger — the floor or the real standard deviation:

```
buffer = max(data_floor, std_dev × 0.65)
```

By game 5 the floor drops to 1.0 and the actual series data fully drives the buffer width.

### All values rounded up (ceil)

There are no half-points in basketball. All Over/Under lines are rounded up to the nearest integer.

---

## System 2 — Narrative Builder Player Projections

**Used by:** Player Projections table in the Narrative Builder

**Data source:** Same game data as System 1, plus narrative scenario, dial intensity, key player selection, and average minutes.

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

Step 3 — Base multiplier (OR, not AND)
  if player is a star pick  →  base_mult = star_mult[stat]
  else                      →  base_mult = narrative_mult[stat]

Step 4 — Narrative-adjusted anchor
  anchor = gates_proj × (1.0 + (base_mult − 1.0) × dial_scale × min_scale)

Step 5 — Project
  projected = ceil(anchor)
```

### Why OR not AND for star/narrative?

Previously the star boost was multiplied on top of the narrative multiplier, causing compounding. A star player in a Shootout would get `narrative × star` which inflated projections significantly. Now the star multiplier **replaces** the narrative multiplier entirely — it is simply a higher adjustment reflecting your conviction that this player will have a big game.

### Why use gates_proj instead of the raw average?

The Gates projection already accounts for momentum via the slope. Using the raw average as the anchor discards trend information. A player scoring 20, 25, 30 across three games has an average of 25 but a Gates projection of 28+. Using the trend-aware anchor means the narrative adjustment is applied on top of an already-intelligent baseline.

### Narrative Builder Dial Scale

Controls how strongly you believe the narrative will play out. Does not affect the buffer width — only shifts the anchor.

| Dial | Scale |
|---|---|
| Conservative | 0.7× |
| Moderate | 1.0× |
| Aggressive | 1.3× |

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

BLOW-B is the exact mirror of BLOW-A with home and away swapped.

**Rationale for key values:**

- **Blowout winner AST (0.90):** Stars are shooting more in a comfortable lead, reducing assists across the board
- **Grind TPM (0.90):** Defences collapse on the perimeter in close defensive games, fewer open 3s
- **Shootout TPM (1.20):** Most aggressive upside — open looks, both teams trading threes
- **Grind/Blowout REB (1.10):** More misses = more rebounds available
- **Shootout REB (0.90):** More makes = fewer misses = fewer boards

### Star Multipliers

Replaces the narrative multiplier for selected key players. Does not stack on top of it.

| Stat | Star mult | Condition | vs narrative |
|---|---|---|---|
| PTS | 1.15 | All stars | Higher than any scenario's PTS mult |
| FGA | 1.10 | All stars | More shot volume |
| AST | 0.85 | All stars | Shooting more = fewer assists |
| TPM | 1.20 | avg TPA ≥ 3 | High-volume 3PT shooter getting hot |
| TPM | 1.08 | avg TPA < 3 | Low-volume shooter, modest boost |
| TPA | 1.15 | avg TPA ≥ 3 | More 3PT attempts for a shooter |
| TPA | 1.05 | avg TPA < 3 | Modest increase |
| REB | narrative mult | No override | Game script drives rebounds |
| STL | narrative mult | No override | Hard to predict individually |
| BLK | narrative mult | No override | Hard to predict individually |

**Hard caps regardless of multiplier:**
- STL: floor 0, ceiling 4
- BLK: floor 0, ceiling 4

### Minutes Scale Rationale

Playoff rotations tighten significantly. A bench player averaging 14 minutes has limited influence on the final stat line regardless of game script. Applying the full narrative multiplier to every player regardless of minutes over-inflates team totals. The minutes scale weights the narrative adjustment by court time:

- **30+ minutes** — primary rotation player, full narrative effect
- **18–29 minutes** — secondary rotation, partial effect
- **Under 18 minutes** — spot/matchup minutes, narrative barely applies

---

## System 3 — SGM Builder Over/Under Lines

**Used by:** SGM Builder table (LOW / MED / HIGH risk)

**What it combines:** The narrative-adjusted anchor from System 2 (Steps 1–4) with the Gates buffer from System 1 (Steps 5–6). This means the centre of the Over/Under window shifts with the narrative, while the window width is driven by actual data.

### Formula

```
Step 1 — Gates projection
  gates_proj = mean + (slope × 0.4)

Step 2 — Minutes scale
  avg_min >= 30  →  min_scale = 1.0
  avg_min 18–29  →  min_scale = 0.6
  avg_min < 18   →  min_scale = 0.2

Step 3 — Base multiplier (OR, not AND)
  star pick      →  base_mult = star_mult[stat]
  non-star       →  base_mult = narrative_mult[stat]

Step 4 — Narrative-adjusted anchor
  anchor = gates_proj × (1.0 + (base_mult − 1.0) × dial_scale × min_scale)

Step 5 — Buffer (data-driven)
  std_dev    = standard deviation of played games
  data_floor = max(1.0, 5.0 − (games_played − 1))
  buffer     = max(data_floor, std_dev × 0.65)

Step 6 — Over/Under lines
  Over  = ceil(max(0, anchor − buffer × tier_mult))
  Under = ceil(anchor + buffer × tier_mult)
```

### SGM Risk Tier Multipliers

| SGM Risk | Tier | Mult | Meaning |
|---|---|---|---|
| LOW | Conservative | 1.80× | Wide window — safer legs, easier to beat |
| MED | Moderate | 1.25× | Balanced — standard bet |
| HIGH | Aggressive | 0.50× | Tight window — higher value, higher risk |

### How the dial affects the SGM Builder

The dial used in Step 4 is the **Narrative Builder intensity dial** — the same one that affects the Player Projections. Changing the dial shifts the anchor for every player simultaneously, which in turn shifts all the Over/Under lines in the SGM Builder. The buffer width does not change with the dial — only the centre point moves.

---

## Worked Examples

### Example 1 — Jalen Brunson, Shootout, Moderate dial, Star selected
*avg MIN=37 · games=3 · mean=27.7 · slope=+1.0 · std_dev=3.2*

```
System 2 (Player Projection):
  gates_proj = 27.7 + (1.0 × 0.4) = 28.1
  min_scale  = 1.0   (37 mins ≥ 30)
  base_mult  = 1.15  (star PTS — replaces narrative)
  dial_scale = 1.0   (Moderate)
  anchor     = 28.1 × (1.0 + (1.15−1.0) × 1.0 × 1.0)
             = 28.1 × 1.15 = 32.3
  projected  = ceil(32.3) = 33

System 3 — LOW risk:
  data_floor = max(1.0, 5.0−2) = 3.0
  buffer     = max(3.0, 3.2 × 0.65) = max(3.0, 2.08) = 3.0
  Over       = ceil(32.3 − 3.0 × 1.80) = ceil(26.9) = 27
  Under      = ceil(32.3 + 3.0 × 1.80) = ceil(37.7) = 38

System 3 — HIGH risk:
  Over       = ceil(32.3 − 3.0 × 0.50) = ceil(30.8) = 31
  Under      = ceil(32.3 + 3.0 × 0.50) = ceil(33.8) = 34
```

### Example 2 — Same player, NOT a star, Shootout Moderate

```
  base_mult  = 1.10  (narrative PTS Shootout)
  anchor     = 28.1 × (1.0 + (1.10−1.0) × 1.0 × 1.0)
             = 28.1 × 1.10 = 30.9
  projected  = 31
  LOW Over   = ceil(30.9 − 5.4) = 26
  LOW Under  = ceil(30.9 + 5.4) = 37
```

### Example 3 — Bench player, 14 avg MIN, not a star, Shootout Moderate
*mean=8.0 · slope=+0.5 · std_dev=2.1*

```
  gates_proj = 8.0 + (0.5 × 0.4) = 8.2
  min_scale  = 0.2  (14 mins < 18)
  base_mult  = 1.10
  anchor     = 8.2 × (1.0 + (1.10−1.0) × 1.0 × 0.2)
             = 8.2 × 1.02 = 8.4
  projected  = 9

  data_floor = 3.0
  buffer     = max(3.0, 2.1 × 0.65) = 3.0
  LOW Over   = ceil(8.4 − 5.4) = 3
  LOW Under  = ceil(8.4 + 5.4) = 14
```

### Example 4 — Same star player, Aggressive dial instead of Moderate

```
  dial_scale = 1.3  (Aggressive)
  anchor     = 28.1 × (1.0 + (1.15−1.0) × 1.3 × 1.0)
             = 28.1 × 1.195 = 33.6
  projected  = 34
  LOW Over   = ceil(33.6 − 5.4) = 29
  LOW Under  = ceil(33.6 + 5.4) = 40
  HIGH Over  = ceil(33.6 − 1.5) = 33
  HIGH Under = ceil(33.6 + 1.5) = 36
```

---

## Summary — Key Design Decisions

| Decision | Rationale |
|---|---|
| Slope × 0.4 as projection | Captures momentum without over-extrapolating short runs |
| Data floor dropping game by game | Prevents false confidence on 1–2 game samples |
| Star mult replaces narrative (OR not AND) | Avoids compounding — one adjustment per player per stat |
| Minutes scale | Bench players shouldn't feel the full narrative regardless of game script |
| Gates projection as anchor for Systems 2 & 3 | Uses trend-aware baseline rather than flat average |
| Dial affects anchor only | Intensity of your narrative belief shifts the centre — not the uncertainty width |
| Tier mult affects buffer width only | Risk tolerance on the bet is separate from belief in the narrative |
| STL/BLK hard capped at 4 | Individual players cannot realistically record more than 4 steals or blocks |
| All values ceil() | No half-points in basketball |

---

*Last updated: April 2026*
*For implementation questions refer to the source code in `index.html` and `nba_boxscore_importer.gs`*
