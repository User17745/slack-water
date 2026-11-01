# Core Rules: Round Structure and Simultaneous Resolution

> Part of the [Game Design Document](../GAME_DESIGN.md). This is the heart of the game. Every other system depends on these rules being **deterministic, order-independent, and explainable in one sentence each**.

---

## 1. Design goals for the rules

| Goal | What it means in practice |
|---|---|
| **Outguess, don't out-click** | The skill is predicting where things will be, not reacting fast. |
| **No dice in combat** | Damage, collisions, and pushes have no randomness. RNG exists only in map generation, rewards, and AI tie-breaks. |
| **Order-independent** | No rule depends on which unit is processed first. If you shuffled unit IDs, the result would be identical. Tests enforce this (see [tech.md](tech.md#3-determinism-requirements)). |
| **Symmetric** | The same rules apply to players, AI, and both versus players. There is no initiative stat and no "player goes first". |
| **Explainable** | Every outcome maps to one of the five Tide Rules below. The replay can say *which* rule applied. |

### The five Tide Rules (shown on one screen in-game)

1. **Everyone plans, then everything happens in five steps:** Brace → Fast → Move → Strike → Tide.
2. **Attacks hit places, not units.** You aim at a tile (or down a lane). Whoever is there when it lands gets hit, friend or foe.
3. **Two units entering one tile: the heavier one gets in. Equal weight: both stop.**
4. **Units never pass through each other.** A swap is a standoff, and both stop.
5. **Damage in a step lands all at once.** Two units can knock each other out in the same step.

---

## 2. Board and coordinates

- The board is **6 × 6**, with columns `a–f` (left to right) and rows `1–6` (bottom to top). Player units deploy on rows 1–2 and enemies on rows 5–6, unless the map says otherwise.
- Movement and adjacency are **orthogonal only** (4 neighbours). There are no diagonals anywhere, so there are no "corner-cutting" edge cases.
- **Distance** means Manhattan distance.

---

## 3. The round

```
 PLAN (secret, untimed)  →  LOCK IN  →  REVEAL  →  RESOLVE: Brace → Fast → Move → Strike → Tide  →  next round
```

### 3.1 Plan

Each side gives each of its units **one order** per round. An order has two optional parts:

| Part | Rule |
|---|---|
| **Move** | A path of up to `Move` orthogonal steps (0 is allowed). It can't enter Rock or the board edge, and walkers can't enter Deep. The path is validated against terrain only; other units aren't known yet. |
| **Action** | One of the unit's two abilities: its **Fast** ability *or* its **Slow** ability. Never both. |

- **Fast ability:** fires in the Fast step, **from the unit's start tile, before anyone moves**. It's weaker or utility-focused. If a unit has both a Fast action and a Move, it acts first and then moves (hit-and-run).
- **Slow ability:** fires in the Strike step, **from wherever the unit actually ends up** after moving. It's stronger, but it needs prediction.
- **Hunker:** a unit with no Move and no Action automatically gets **Brace 1** (takes 1 less damage from each hit this round). Standing still is a real choice, not a wasted turn.
- Some abilities say *"no move this round"* (e.g. Shell Up). The planner greys out the Move control for them.

### 3.2 Aim types

Every ability has exactly one aim type. This is shown as an icon on the ability button.

| Aim | You choose | When it resolves |
|---|---|---|
| **Spot** | An absolute tile within Reach *of where you plan to be*. | Reach is re-checked from where the unit **actually is**. If the tile is out of reach, the ability **fizzles**, shown as a grey puff with the label "Out of reach". |
| **Lane** | A direction (N/E/S/W). | It fires from the unit's **actual** position and hits the first unit in that direction within Range. Rock blocks it. |
| **Self** | Nothing. | It affects the unit itself. |

> **Why absolute Spot aim?** "I'm punching *that* tile" is how people think about prediction. If you get bumped and miss, the replay shows exactly why. Relative aim ("punch north from wherever I end up") made playtest-style thought experiments hit allies unpredictably, so we rejected it.

### 3.3 Lock in

- Solo: tap and hold **Lock In** for 0.5 s, which prevents accidental locks. The AI's plan was **already fixed at the start of the planning step**, computed from the board alone. The AI can't see your plan, and this is architectural, not a promise (see [tech.md](tech.md#5-ai-determinism)).
- Versus on one phone: lock, then the privacy screen ([two-player.md](two-player.md#3-same-phone-flow-pass-the-phone)).
- Versus on two phones: lock sends a commitment hash first, then the plan is revealed ([two-player.md](two-player.md#7-netcode-sketch-for-versus-fairness)).
- There's no timer by default. An optional chess-clock (30 s per round) exists for Link versus.

### 3.4 Resolve: the five steps

Every step uses **snapshot semantics**: read the state at the start of the step, compute all effects, then apply them together.

#### Step 1: Brace
All guard effects switch on: Hunker (Brace 1), Shell Up (Brace 2), Cling, Puff, and so on. They last until the end of the round. Guards are on *before* any damage, so a Fast guard protects against Fast attacks.

#### Step 2: Fast
1. Every Fast ability resolves from its unit's **start tile**, against the board at the start of the round.
2. Damage is applied simultaneously (see §5).
3. Units at 0 HP are knocked out (KO) and removed. **A KO'd unit does nothing else this round**, so its Move and Slow are cancelled. This is the payoff of Fast: interrupting.
4. Displacements (push/pull) resolve (see §6).
5. **A unit displaced in the Fast step loses its Move this round.** Its planned path started from a tile it's no longer on. Its Slow ability (if any) still fires in Strike, if the target is within reach of its new tile.

#### Step 3: Move
Movement resolves in **ticks**. In tick *t* (1 to 3, since the max Move is 3), every unit that still has a step *t* and hasn't been stopped tries to take that step. The algorithm is in §4.

#### Step 4: Strike
1. Every Slow ability resolves from its unit's **current** tile (Spot reach is re-checked; Lane fires from here).
2. Damage is applied simultaneously, then KOs are removed, then displacements resolve, then displacement damage applies, then KOs are removed again.
3. Units KO'd in this step **still deal their Strike damage**, because everything landed at once (Tide Rule 5).

#### Step 5: Tide
In this fixed order:
1. **Heals** (Regrow, Leech Mend) are applied. They can't revive KO'd units.
2. **Currents:** each unit on a Current tile slides 1 tile in the arrow direction. Use the tick algorithm (§4), but a blocked slide means no movement and no damage. Maps never point currents into Deep.
3. **Hazards:** telegraphed hazards (e.g. a Gull Dive marked last round) hit their tiles.
4. **Objectives:** Pearl tiles are scored, and escort or survive objectives are checked.
5. **Expiry:** Brace ends and Ink ages by 1 round.
6. **Battle end check,** then the round counter goes up.

> **Battle end** is also checked at the end of the Fast and Strike steps. If one side has no units left, the battle ends right there. In solo, if all your fielded critters are KO'd you lose, **even if the Murk was wiped out in the same step**. Mutual KO is a loss in solo and goes to tiebreak in versus.

---

## 4. Movement tick algorithm (formal)

For each tick `t = 1..3`:

```
M  = units that are not stopped and have a planned step t
F(u) = u's current tile, T(u) = u's step-t tile

1. CONTEST   For every tile X with two or more movers where T(u)=X:
               - if exactly one mover has the highest Weight → it is the only one allowed to try X
               - otherwise (tie at the top) → all movers into X are STOPPED   ("Standoff")
             Losers are STOPPED ("Bounced").
2. SWAP      If u and v are both still in M with T(u)=F(v) and T(v)=F(u) → both STOPPED ("Standoff").
3. BLOCKED   Repeat until nothing changes:
               a mover u is STOPPED if T(u) is occupied by any unit that is not a still-valid mover
               ("Blocked"). Spiny units deal 1 damage to units blocked by them.
4. APPLY     All still-valid movers move to T(u) at once.
STOPPED units cancel all their remaining steps for this round.
```

Properties (each is a unit test):

- **Order-independent:** steps 1–2 are set operations, and step 3 only ever *adds* stopped units, so it always reaches the same fixed point.
- **Follow-the-leader works:** you can step into a tile that its occupant is leaving in the same tick.
- **Rotations work:** a cycle of 3 or more units (e.g. four units rotating around a 2×2 square) all move, because nobody passes through anybody. A 2-cycle is a swap, which is a standoff.
- **Losing a contest stops you even if the winner is later blocked.** The tile then stays empty. This is rare, it's shown in the replay, and it keeps the algorithm order-free.
- **Bump damage** (from Spiny) during Move is applied at the end of the Move step, followed by a KO check.

---

## 5. Damage

- Each hit deals its listed damage, **minus the target's Brace** (applied per hit, minimum 0).
- All hits in a step are summed and applied together, then the KO check runs.
- **Friendly fire is on.** Spot and Lane abilities hit whoever is there. The planner shows a red warning if one of your planned attacks targets a tile one of your own units plans to stand on.
- **Cover:** a unit standing in Kelp takes 1 less damage from Lane hits.

## 6. Displacement (push and pull)

Displacements in a step are resolved together, *after* that step's damage and KO removal:

1. Each displaced unit has exactly one intended 1-tile vector.
2. **Pinned:** a unit with two or more displacement sources in the same step doesn't move and takes 1 damage.
3. The remaining displacements resolve as **one tick of the movement algorithm**, with these changes:
   - Weight doesn't matter. Two pushed units entering the same tile both stop, and each takes 1 bump damage.
   - Blocked by a unit, Rock, or the board edge: the pushed unit stays and takes **1 bump damage**. The blocker takes none, unless the pushed unit is Spiny.
   - Pushed into Deep (non-swimmer): **Swept**. The unit is removed from this battle. A Swept enemy counts as defeated. A Swept critter returns next battle with its current HP. Being Swept is safer than being KO'd, but you lose the unit for this fight.
4. Bump damage is applied, then KOs are removed.

---

## 7. Readability and fairness tools

| Tool | What it does |
|---|---|
| **Step-by-step playback** | Each step has a banner (`FAST`, `MOVE`, `STRIKE`, `TIDE`) and a distinct sound. The speed can be 1×, 2×, or *Step* (tap to advance), and it remembers your setting. |
| **Outcome icons** | Bounced (small spiral), Standoff (two arrows touching), Blocked (wall), Fizzled (grey puff), Pinned (cross), Swept (wave). |
| **"Why?" log** | After resolution, tap any unit or the log to see one-line causes, e.g. *"Mite bounced: Hermit is heavier (3 vs 1)."* |
| **Replay round** | Rewatch the last round any number of times. |
| **Enemy Tells** (solo) | Each enemy shows its action type and the *possible* reach of its attack this round. Simple enemies show more (see [units-and-combat.md](units-and-combat.md#5-the-murk-enemies)). |
| **Threat map** (toggle) | Shades every tile any visible enemy could hit this round. It's computed from max Move plus max Reach, so it's honest but not exact. |
| **Self-collision warning** | If two of *your* planned paths will contest, swap, or block, a yellow marker appears before you lock. |
| **Planning ghosts** | Your planned paths and targets are drawn as translucent ghosts. In Link co-op, your ally's ghosts are drawn in their colour. |

> **Why partial information against the AI?** Full enemy telegraphs turn each turn into a solved puzzle (that's Into the Breach's identity, and we don't want to copy it). Hiding everything would feel random. Tells sit in between: you know *what* each enemy will do and roughly *where*, and the guess is about *exactly where*. The **Honest Tides** assist (see [GAME_DESIGN.md §8](../GAME_DESIGN.md#83-accessibility)) shows exact AI targets for players who want the puzzle version.

---

## 8. Worked example turns

Legend: player critters are uppercase, Murk enemies lowercase. `.` = Sand, `#` = Rock.
Stats used are from [units-and-combat.md](units-and-combat.md).

### Example 1: "Weight wins, aim stays put"

```
   a  b  c  d  e  f
6  .  .  .  .  .  .
5  .  .  .  .  .  .
4  .  .  m1 .  m2 .        m1, m2 = Mite (HP 2, Wt 1)
3  .  .  .  .  .  .
2  .  .  H  .  S  .        H = Hermit Crab (HP 6, Wt 3), S = Mantis Shrimp (HP 3, Wt 1)
1  .  .  .  .  .  .
```

**Tells shown to the player:** m1 → *Nip, targeting Hermit*. m2 → *Nip, targeting Mantis*. (Mite habit: step next to the nearest critter and nip the tile it **started** on.)

**Plans**

| Unit | Move | Action |
|---|---|---|
| H | c2 → c3 | Slow *Pinch* aimed at **c4** (betting the Mite stays put or bounces) |
| S | e2 → e3 | Slow *Haymaker* aimed at **e4** |
| m1 (AI) | c4 → c3 | Slow *Nip* aimed at **c2** |
| m2 (AI) | e4 → e3 | Slow *Nip* aimed at **e2** |

**Resolution**

1. **Brace:** nothing. Every unit is acting, so nobody hunkers.
2. **Fast:** nothing.
3. **Move, tick 1:**
   - H and m1 both want **c3**. This is a contest: Weight 3 beats 1. **H enters c3, and m1 is Bounced** (it stays on c4).
   - S and m2 both want **e3**. Weight 1 vs 1 is a tie: **Standoff, and both stop.** S stays on e2 and m2 stays on e4.
4. **Strike:**
   - H (c3) Pinch → c4: adjacent ✓. m1 is there, so it takes **2 damage → KO**.
   - m1 (c4) Nip → c2: not adjacent from c4, so it **fizzles**. (It would have fizzled even if m1 were KO'd later, because reach is checked from its actual tile.)
   - S (e2) Haymaker → e4: not adjacent from e2, so it **fizzles**.
   - m2 (e4) Nip → e2: not adjacent, so it **fizzles**.
5. **Tide:** nothing.

**Lesson the replay teaches:** S lost a coin-flip it didn't need to take. If S had **stayed on e2** and aimed Haymaker at **e3**, m2 would have walked into a 3-damage punch (KO), with no standoff. Predicting the enemy's step beats racing it.

---

### Example 2: "The swap and the trade"

```
   a  b  c  d  e  f
6  .  .  .  .  .  .
5  .  s  .  .  .  .        s = Spitter (HP 2)
4  .  .  .  L  .  .        L = Lurker (HP 3, Wt 2)
3  .  .  .  S  .  .        S = Mantis Shrimp (HP 3, Wt 1)
2  .  P  .  .  .  .        P = Pistol Shrimp (HP 3)
1  .  .  .  .  .  .
```

**Tells:** Spitter → *Spit, lane SOUTH down column b* (Spitters always show their lane). Lurker → *Rake, somewhere within the shaded 3×3 region around d3* (Lurkers only show a region).

**Plans**

| Unit | Move | Action |
|---|---|---|
| P | none | Slow *Snap Shot*, lane **N** (range 4) |
| S | d3 → d4 | Slow *Haymaker* aimed at **d5** (expects the Lurker to back off) |
| s (AI) | none | Slow *Spit*, lane **S** (range 5) |
| L (AI) | d4 → d3 | Slow *Rake* aimed at **d2** (predicts the Mantis will retreat) |

**Resolution**

1. **Brace:** nothing. P is acting, so it doesn't hunker.
2. **Fast:** nothing.
3. **Move, tick 1:** S wants d4 and L wants d3. They want each other's tiles: **Swap → Standoff, and both stop.**
4. **Strike** (all at once):
   - P Snap Shot north from b2: b3 and b4 are empty, and **s** is on b5 (distance 3 ≤ 4). s takes **2 → HP 0**.
   - s Spit south from b5: the first unit is **P** on b2 (distance 3 ≤ 5). P takes **2 → HP 1**.
   - S Haymaker → d5: S is still on d3, so d5 isn't adjacent and it **fizzles**.
   - L Rake → d2: L is still on d4, so d2 isn't adjacent and it **fizzles**.
   - Damage applies together. **s is KO'd, but its shot still landed** (Tide Rule 5).
5. **Tide:** nothing.

**What else could P have done?**
- Step a2 and shoot north: column a is empty, so P misses, and s's lane down column b is now empty too. Both miss.
- Do nothing (Hunker): P takes 2 − 1 = **1**, and s survives.
- The trade P took (lose 2 HP, kill the Spitter) was the aggressive choice. All three choices are readable before locking in, because the Spitter's lane was telegraphed.

---

### Example 3: "Fast beats Move"

```
   a  b  c  d  e  f
6  .  .  .  .  .  .
5  .  .  .  m  .  .        m = Mite (HP 2, Wt 1)
4  .  .  B  .  .  .        B = Bulwark (HP 6, Wt 3)
3  .  .  .  .  .  .
2  .  H  O  S  .  .        H = Hermit (HP 6), O = Octopus (HP 4), S = Mantis (HP 3)
1  .  .  .  .  .  .
```

**Tells:** Bulwark → *Shove, targeting Octopus*. Mite → *Nip, targeting Mantis*.

**Plans**

| Unit | Move | Action |
|---|---|---|
| O | none | **Fast** *Grab*, lane **N** (range 2): pull the first unit 1 tile toward O |
| H | b2 → b3 | Slow *Pinch* aimed at **c3** |
| S | d2 → d3 | Slow *Haymaker* aimed at **c3** (push goes away from S, which is west) |
| B (AI) | c4 → c3 | Slow *Shove* aimed at **c2** |
| m (AI) | d5 → d4 → d3 | Slow *Nip* aimed at **d2** |

**Resolution**

1. **Brace:** nothing.
2. **Fast:** O Grabs north from c2. c3 is empty, and **B** is on c4 (distance 2). B is pulled 1 tile to **c3**. **B was displaced, so it loses its Move.** Its Slow Shove (aimed at c2) is kept, to be checked from c3.
3. **Move:**
   - Tick 1: H b2→b3, S d2→d3, and m d5→d4. No conflicts.
   - Tick 2: m wants d3, but **S is there and has finished moving → Blocked**. m stops on d4.
4. **Strike** (positions: H b3, O c2, S d3, B c3, m d4):
   - S Haymaker → c3: adjacent ✓. B takes 3, and a push west is queued.
   - H Pinch → c3: adjacent ✓. B takes 2.
   - B Shove → c2: adjacent from c3 ✓. O takes 1, and a push south is queued.
   - m Nip → d2: not adjacent from d4, so it **fizzles**.
   - Damage applies together: B is at 6 − 5 = **1**, and O is at 4 − 1 = **3**. No KOs yet.
   - Displacements: B is pushed west into **b3**, but H is there, so B is **Blocked and takes 1 bump damage**. O is pushed south to **c1**, which is empty, so it moves.
   - Bump damage applies: **B is KO'd** (1 − 1 = 0).
5. **Tide:** nothing.

**Lesson:** the Fast Grab did two jobs. It cancelled the Bulwark's move, and it put the Bulwark on a tile both critters had pre-aimed at. The Octopus still took the Shove, because a displaced unit keeps its Slow if the target is still in reach. That trade was visible from the Tell.

---

## 9. Edge-case table (for tests and the "Why?" log)

| Situation | Result |
|---|---|
| A unit is KO'd in Fast | It doesn't move or Strike, and it doesn't block anything after it's removed. |
| A unit is displaced in Fast | It loses its Move. Its Slow fires if still in reach (Spot) or from its new tile (Lane). |
| A Spot target becomes unreachable after a bounce | It fizzles with the label "Out of reach". |
| Two allies aim at the same tile | Both hits land and the damage sums. |
| You attack a tile your own unit moved into | Your unit takes the hit. The planner warned you before you locked. |
| Three units contest one tile with weights 3, 3, 1 | The top weight is tied, so all three stop. |
| Three units contest one tile with weights 3, 2, 2 | The Wt 3 unit gets to try the tile. Both Wt 2 units stop. |
| A unit is pushed by two sources in one step | Pinned: it doesn't move and takes 1 damage. |
| A push goes into a tile being vacated by another pushed unit | Allowed, same as follow-the-leader. |
| Heal and KO in the same round | KOs happen in Fast and Strike, and heals happen in Tide, so a KO'd unit can't be healed. |
| All units on both sides are KO'd in one step | Solo: loss. Versus: tiebreak (see [two-player.md](two-player.md#1-versus-duel)). |
| A unit planned a Move but was Stopped at tick 1 | No Hunker. Hunker is decided at plan time (no Move and no Action planned). |
