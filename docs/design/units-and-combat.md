# Units and Combat

> Part of the [Game Design Document](../GAME_DESIGN.md). The rules for resolving all of this are in [core-rules.md](core-rules.md).

Scope principle: **few units, each with a sharp identity**. The MVP has 6 critters, 5 enemies, and 1 boss. v1.0 has 10 critters, 8 enemies, and 3 bosses. Every critter has exactly 2 abilities (1 Fast, 1 Slow) and at most 1 trait.

---

## 1. Board

- **6 × 6** tiles in portrait. At 360 pt width with 18 pt margins, tiles are 54 pt, comfortably above the 44 pt touch minimum.
- **Solo:** 3 critters vs 2–5 Murk (scaled by stop and Tide Level).
- **Co-op:** 4 critters (2 per player) vs 1.5× the enemy budget.
- **Versus:** 3 vs 3 on symmetric arenas.
- Battles last at most **6 rounds** (boss fights 8). A round takes about 15–25 s to plan, so a battle takes about 2–2.5 minutes.

## 2. Unit stats

| Stat | Range | Meaning |
|---|---|---|
| **HP** | 3–6 (bosses 12–18) | Small numbers make math done at a glance possible. |
| **Move** | 0–3 | Max orthogonal steps per round. |
| **Weight** | 1–3 | Settles tile contests (Tide Rule 3). Shown as 1–3 pebbles under the portrait. |
| **Traits** | 0–1 | Swimmer (can enter Deep, immune to Swept), Spiny, etc. |

## 3. Critter roster

Aim abbreviations: **Spot R1** = an absolute tile at distance 1 (adjacent). **Lane R4** = a direction, first unit within 4. **Lob R3** = any tile within distance 3, ignoring blockers.

### MVP (6)

| Critter | Role | HP | Mv | Wt | Fast ability | Slow ability | Trait |
|---|---|---|---|---|---|---|---|
| **Hermit Crab** | Anchor | 6 | 2 | 3 | **Shell Up** (Self): Brace 2 this round. No move. | **Pinch** (Spot R1): 2 dmg. | **Shell Collector**: holds 2 Shells instead of 1. |
| **Mantis Shrimp** | Striker | 3 | 3 | 1 | **Jab** (Spot R1): 1 dmg. | **Haymaker** (Spot R1): 3 dmg, push 1 away. | none |
| **Octopus** | Controller | 4 | 2 | 2 | **Grab** (Lane R2): pull the first unit 1 tile toward you. | **Ink** (Lob R3): target tile + 4 neighbours are Inked for the next round (Inked units can't use Slow). | Swimmer |
| **Pistol Shrimp** | Artillery | 3 | 2 | 1 | **Pop** (Spot R1): push 1 away, 0 dmg. | **Snap Shot** (Lane R4): 2 dmg. | none |
| **Sea Star** | Medic | 4 | 1 | 2 | **Cling** (Self): Brace 1, can't be displaced this round. No move. | **Regrow** (Spot R1 or self): heal 2 in the Tide step. | **Five Arms**: the first KO each battle leaves it at 1 HP instead. |
| **Sea Urchin** | Zoner | 4 | 1 | 2 | **Roll** (Self): move 1 tile *in the Fast step* (resolved as a displacement with no damage). | **Spine Burst** (Self): 1 dmg to all 4 adjacent tiles. | **Spiny**: units blocked by it, or pushed into it, take 1. |

### v1.0 additions (4)

| Critter | Role | HP | Mv | Wt | Fast ability | Slow ability | Trait |
|---|---|---|---|---|---|---|---|
| **Moray Eel** | Lancer | 4 | 2 | 2 | **Recoil** (Self): move 1 in the Fast step, away from the nearest enemy. | **Lunge** (Lane R3): 2 dmg to the first unit, and the Eel slides to the tile before it. | Swimmer |
| **Pufferfish** | Bouncer | 5 | 2 | 1 | **Puff** (Self): Weight 3, Brace 1, and Spiny this round. Can still move. | **Gust** (Self): push all adjacent units 1. | none |
| **Cuttlefish** | Trickster | 3 | 2 | 1 | **Shimmer** (Spot R1): swap places with an adjacent ally (both lose Move). | **Dazzle** (Lob R2): the target unit can't use Fast next round. | Swimmer |
| **Anemone** | Turret | 5 | 0 | 3 | **Clamp** (Self): Brace 2. | **Sting** (Self): 2 dmg to all adjacent *enemies* (the only friendly-safe area attack). | **Rooted**: never displaced. |

### Molts (in-run upgrades)

Each critter has **2 Molts**, and you can take **one per critter per run** (alternates unlock later). Examples:

| Critter | Molt A | Molt B |
|---|---|---|
| Hermit Crab | **Big Claw**: Pinch deals 3. | **Barbed Shell**: Shell Up also makes it Spiny. |
| Mantis Shrimp | **Follow-through**: Haymaker pushes 2. | **Quick Hands**: Jab hits two different adjacent tiles (1 dmg each). |
| Octopus | **Long Arms**: Grab Lane R3. | **Thick Ink**: Inked tiles also give Brace 1 to your units. |
| Pistol Shrimp | **Ricochet**: Snap Shot also hits the next unit behind for 1. | **Recoil Pop**: Pop also moves the Shrimp 1 tile back. |
| Sea Star | **Spread**: Regrow heals 1 to *all* adjacent allies. | **Hardy**: +2 max HP. |
| Sea Urchin | **Longer Spines**: Spine Burst deals 2. | **Tumble**: Roll moves 2. |

### Shells (gear)

Each critter holds 1 Shell (the Hermit holds 2). There are 10 in the MVP and 22 in v1.0. Shells are **rules modifiers**, not stat soup:

| Shell | Effect |
|---|---|
| Cowrie | +1 max HP |
| Conch | +1 Move |
| Limpet | Can't be displaced |
| Turban | Wins Weight ties |
| Scallop | The first hit each battle is reduced by 2 |
| Razor | +1 damage on Slow, −1 max HP |
| Whelk | Slow deals +1 in round 1 |
| Moon Snail | Heal 1 after each battle |
| Periwinkle | Fast range +1 |
| Auger | Lane attacks ignore Kelp cover |

## 4. Terrain

| Tile | Look | Rule |
|---|---|---|
| **Sand** | Pale | Normal. |
| **Rock** | Grey boulder | Impassable. Blocks Lanes. Pushing a unit into it causes 1 bump damage. |
| **Kelp** | Swaying fronds | −1 damage from Lane hits (cover). Doesn't block movement. |
| **Shallows** | Light ripples | Entering it **ends your movement** (Swimmers ignore this). |
| **Deep** | Dark blue | Walkers can't enter. Being pushed in sweeps the unit away (Swept: removed from the battle). Swimmers move freely. |
| **Current** | Arrow ripples | In the Tide step, the unit slides 1 tile in the arrow direction. |
| **Pearl** | Glowing pearl | Objective tile (held = your unit on it, no enemy on it, at the end of Tide). |

**Neutral hazard (v1.0):** the **Gull** casts a shadow on 2 tiles at the end of a round, and dives on those tiles in the next round's Tide step (3 dmg). It's always fully telegraphed and hits both sides.

Maps are hand-authored **6×6 templates** (MVP 12, v1.0 36). Each one has seeded variations: mirrored, terrain swaps, and spawn shuffles. Hand-authoring keeps every board fair and readable. Procedural boards on a 6×6 grid produce too many degenerate layouts.

## 5. The Murk (enemies)

The antagonist is **the Murk**: a living silt that animates debris into creatures. Defeated Murk dissolve into clean water. The game never shows blood or death, so it's suitable for all ages.

Every enemy has a **Habit**. This is a one-sentence, learnable rule printed on its info sheet. The AI follows the Habit and uses scoring only to break ties.

| Enemy | HP | Mv | Wt | Attack | Habit (printed in-game) | Tell (what you see) |
|---|---|---|---|---|---|---|
| **Mite** | 2 | 2 | 1 | Nip (Spot R1, 1 dmg) | "Steps next to the nearest critter and nips the tile it **started** on." | Exact target critter. |
| **Spitter** | 2 | 1 | 1 | Spit (Lane R5, 2 dmg) | "Picks a lane with a critter in it. Never moves when it spits." | The exact lane. |
| **Bulwark** | 6 | 1 | 3 | Shove (Spot R1, 1 dmg + push 1) | "Walks at the critter closest to a Pearl and shoves it." | Target critter. |
| **Lurker** | 3 | 2 | 2 | Rake (Spot R1, 2 dmg) | "Guesses where the nearest critter is going, and waits there." | A 3×3 region only. |
| **Leech** | 3 | 2 | 1 | Mend (heal 2 to an adjacent Murk in the Tide step) | "Stays out of reach and patches up the most hurt Murk." | Heal target. |
| *v1.0* **Splitter** | 4 | 1 | 2 | Slam (Spot R1, 2 dmg) | "When knocked out, splits into two Mites on adjacent tiles." | Target critter. |
| *v1.0* **Barnacle Mine** | 2 | 0 | 3 | Burst | "Bursts for 2 on all neighbours if any unit ends a round next to it." | Always visible. |
| *v1.0* **Drifter** | 3 | 3 | 1 | Sting (Spot R1, 1 dmg + Inked) | "Rides currents and stings whoever is alone." | Region only. |

### Bosses

| Boss | Shore | Shape | HP | Gimmick |
|---|---|---|---|---|
| **The Undertow** (MVP) | Rockpool | 2×1 eel-shape, occupies 2 tiles | 14 | Cycles three patterns (Lash a full row → Coil (Brace 2) → Drag: shifts every Current tile's direction). Only its **tail tile** takes full damage, and the head takes 1 less. The next pattern is always shown. |
| **Gloom Lantern** (v1.0) | Kelp Shallows | 1 tile + lure | 12 | Each Fast step, it pulls the nearest critter 1 toward itself (a known, unavoidable pull you plan around). Its lure tile moves and makes it invisible to Lanes. |
| **The Drifting Bell** (v1.0) | Wreck Reef | 2×2 jelly | 18 | Moves 1 every round, stinging tiles it passes. At 9 HP it splits into two 1×1 Bells, each with half the remaining HP. |

Boss fights last up to **8 rounds**. If the boss isn't beaten by then, the tide turns and **the run ends**. The round counter is always visible.

## 6. AI design

### 6.1 Principles

1. **The AI never sees your plan.** It plans at the start of the planning step, from the board alone (enforced in code, see [tech.md](tech.md#5-ai-determinism)).
2. **Habits first, cleverness second.** Each enemy's Habit picks its *intent* (target critter or Pearl). Scoring picks the exact path and aim among the options that fit the Habit.
3. **Coordinated, not omniscient.** The Murk plan together so they don't bounce off each other, except Mites, which are deliberately clumsy (that's part of their character).
4. **Deterministic and bounded.** It uses fixed iteration counts, never wall-clock time limits, so both phones in co-op compute the identical AI plan with no network traffic.

### 6.2 How "prediction" works for the AI

To guess where critters will go, the AI builds a small **prediction set** for each critter:

- "Stays put"
- "Moves toward the nearest Pearl / objective"
- "Moves toward the nearest enemy"
- "Retreats from the most threatened tile"

Each guess gets a weight. At **Tide Level 0** the AI assumes "stays put" 70% of the time, so it's predictable and punishes passivity. At higher Tide Levels it spreads probability across the set and aims at the tile with the highest expected hit value. Ties are broken by the seeded AI RNG stream.

| Tide Level | AI change |
|---|---|
| 0–2 | Mites and Bulwarks use the start tile. Lurkers use "toward the nearest objective". |
| 3–5 | Lurkers use the full prediction set. Spitters prefer lanes that cover two critter guesses. |
| 6+ | Enemies avoid your threat map unless they can trade favourably. Leeches stand behind Rock. |

### 6.3 Enemy budget

Each enemy has a cost (Mite 1, Spitter 2, Leech 2, Lurker 3, Bulwark 3, Splitter 3, Drifter 2, Mine 1). A battle at stop *n* has a **budget of 3 + n** (×1.5 in co-op, +1 per 2 Tide Levels). A pool generator, seeded by the run, picks the composition with these limits: at most 2 of any type, and at most 5 units.
