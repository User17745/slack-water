# Roguelite Runs, Meta-Progression, and the Daily Tide

> Part of the [Game Design Document](../GAME_DESIGN.md).

---

## 1. Run at a glance

| Item | Decision |
|---|---|
| Target length | **About 15 minutes** (12–18 min). |
| Structure | **8 stops** along one Shore, ending in a boss. |
| Squad | Start with **3 critters**. The squad grows to at most **4**, and you field 3 per battle (chosen before each battle). |
| Fights | 5 battles (one may be a Tough Pool) + 1 boss = 6 fights of about 2 minutes each. |
| Non-combat | 2 stops (Market, Rest Rock, or Stranger), about 30 s each. |
| Losing | A battle where all fielded critters are KO'd ends the run. So does failing to beat the boss within 8 rounds. |
| Persistence | Critter HP carries between battles. KO'd critters come back next battle at 1 HP (if Sea Star's Five Arms didn't save them). |

## 2. The map: "the Tide Chart"

The run is a walk along a Shore at falling tide. At each stop **you choose one of two pools**. You can see the next fork's icons, but not further. This is deliberately simpler than a full branching map: it's one decision per stop, readable one-handed, and fast.

Example of one seeded chart (each column is one fork, and you pick one of the two pools):

| Stop | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|---|---|---|---|---|---|---|---|---|
| Pool A | Skirmish | Skirmish (Hold) | Market | Skirmish (Rescue) | Skirmish | Rest Rock | Skirmish | **Boss** |
| Pool B | — | Tough Pool | Stranger | Skirmish (Clear) | Tough Pool | Stranger | Tough Pool | — |

Generation rules (seeded):
- Stop 1 is always a Skirmish, and stop 8 is always the Boss.
- Each fork offers **two options of the same category** (two battles, or two non-combat stops). Over stops 2–7 that's always **exactly 4 battles and 2 non-combat stops**, so run length stays predictable. Forks only change *which* battle or non-combat type you get.
- At least one fork offers a Tough Pool.

### Stop types

| Stop | What happens |
|---|---|
| **Skirmish** | A normal battle with an objective (see below). |
| **Tough Pool** | +2 enemy budget. The reward is a pick of 1 of 4 instead of 1 of 3, plus double Sand Dollars. |
| **Market** | Spend Sand Dollars: 3 Shells, 1 Molt, and a full heal for one critter. Stock is seeded. |
| **Rest Rock** | Choose one: heal all critters to full, **or** give one critter a Molt. |
| **Stranger** | A critter offers to join (a free recruit), or you get a small 2-choice event (e.g. "Trade 2 HP on your Hermit for a Shell"). |
| **Boss** | The Shore's boss. 8-round limit. |

### Battle objectives (for reward bonuses, never for failure)

| Objective | Bonus condition |
|---|---|
| **Clear** | KO all Murk within 6 rounds. |
| **Hold** | Hold a Pearl tile at the end of 3 different rounds. |
| **Rescue** | Walk a stranded critter (a free, controllable 4th unit) off the top edge. If rescued, it joins as a recruit offer. |
| **Weather** | A neutral Gull hazard is active. Finish with no critter KO'd. |

If round 6 ends and Murk remain, **the tide turns and they retreat**. You win the battle, but you don't get the bonus. Battles never end in a fail state other than a wipe. This keeps runs moving and avoids drawn-out stalemates.

## 3. Rewards

After each battle you get:
- **Sand Dollars** (run-only currency): 3 for a win, +2 for the objective, ×2 in a Tough Pool.
- **Pick 1 of 3** from a seeded offer. Each slot is one of the following:
  - **Recruit**: a new critter (at base stats) joins. If your squad is full (4), you release one.
  - **Shell**: equip it now or carry it (you can carry 2 unequipped).
  - **Molt**: upgrade one critter (one per critter per run).

There are no cards, no deck, and no draws. Everything you own in a run is visible on the squad screen at all times.

## 4. Difficulty curve

| Stop | Budget (Tide 0) | Typical enemies |
|---|---|---|
| 1 | 4 | 2 Mites + Spitter |
| 2–3 | 5–6 | Mites, Spitter, Leech or Lurker |
| 4–5 | 7–8 | Bulwark + Lurker + support |
| 6–7 | 9–10 | 4–5 enemies, mixed |
| 8 | Boss + 2–3 escorts | Shore-specific |

**Tide Levels 0–10** are the difficulty ladder. Beating a run at Tide *N* unlocks Tide *N+1*. Each level adds **one** named modifier, and the modifiers stack:

| Tide | Modifier |
|---|---|
| 1 | Enemies +1 HP on the strongest unit in each battle |
| 2 | Lurkers use the full prediction set |
| 3 | Market prices +1 |
| 4 | Enemy budget +1 |
| 5 | The boss gains a 4th pattern |
| 6 | AI avoids your threat map |
| 7 | Rest Rock heals 50% instead of full |
| 8 | Enemy budget +1 |
| 9 | One fewer reward choice (pick 1 of 2) |
| 10 | Boss round limit 7 |

## 5. Meta-progression

### 5.1 Principles

- **Every run earns something, including losses.** You never finish a run with zero progress.
- **Unlocks add options, not raw power.** A Tide 0 player with everything unlocked is barely stronger than a new player. Mostly they have more variety.
- **No grind walls, no time gates, no premium currency.** Unlocks come from playing, never from waiting.
- **Fully unlockable in about 20 runs (about 5 hours).** After that, depth comes from Tide Levels, Keepsakes, and mastery.
- **An "Unlock everything" toggle in Settings.** It's for returning players, reviewers, and someone who wants to play versus with a friend on day one. Using it changes nothing else.

### 5.2 Tide Marks: the single progression track

You earn **Tide Marks** at the end of every run: **1 per stop reached, +1 per objective bonus, +3 for beating the boss**. A normal run earns 5–14. Marks fill a **fixed, visible track**. You don't spend them, so there's no "what should I buy" anxiety and no hoarding.

| # | Marks | Unlock |
|---|---|---|
| 0 | start | Hermit Crab, Mantis Shrimp, Sea Star; Rockpool Shore; 6 Shells |
| 1 | 5 | **Octopus** |
| 2 | 12 | +4 Shells |
| 3 | 20 | **Pistol Shrimp** |
| 4 | 28 | **Kelp Shallows** Shore (+ Gloom Lantern) |
| 5 | 36 | **Sea Urchin** |
| 6 | 45 | Alternate Molts for the first 3 critters |
| 7 | 54 | **Moray Eel** |
| 8 | 64 | +6 Shells |
| 9 | 74 | **Wreck Reef** Shore (+ Drifting Bell) |
| 10 | 84 | **Keepsakes**: bring one into a run |
| 11 | 95 | **Pufferfish** |
| 12 | 106 | **Cuttlefish** + alternate Molts for the next 3 |
| 13 | 118 | **Anemone** + 6 Shells |
| 14 | 130 | Alternate Molts for the rest + an Aquarium decor set |

The game is free, so every step is earned by playing alone. There is nothing to buy.

### 5.3 Journal feats (side goals)

About 30 feats, such as "Win a battle where an enemy is KO'd by a bump" or "Beat the Undertow without moving the Hermit". They unlock **cosmetics only**: critter colour morphs, Aquarium decorations, and board palettes. They're optional, never required, and have no timers.

### 5.4 Keepsakes and the Aquarium

- When you **win a run**, you may **bottle one critter** from your final squad as a **Keepsake**. It keeps:
  - its species, plus a **generated name** (e.g. "Pinch the Stubborn")
  - a **colour morph**: 1 of 8, plus a 1-in-64 Pearlescent
  - one **Quirk**: a sidegrade like "+1 Move, −1 HP" or "Weight +1, Move −1", rolled from 12
  - its **Home Shell**: one Shell it wore during the run
  - its **provenance**: who bottled it, and later who it was traded to and when
- Keepsakes live in your **Aquarium**, which is the home screen: a living tidepool where your Keepsakes swim around. It's cozy, it's pure display, and there's nothing to maintain or feed.
- **Use:** once unlocked (track step 10), you may start a run with **one Keepsake** in place of a base starter of any species. It starts at base stats plus its Quirk and Home Shell. Its run Molts aren't kept.
- **Keepsakes are the tradeable item** (see [two-player.md](two-player.md#6-trading)). Every Keepsake is earnable solo, and trading only adds variety and memories.

## 6. What carries over vs. resets

| Carries over | Resets every run |
|---|---|
| Unlocked critters, Shores, Shells, Molts | Squad, HP, Shells equipped, Molts taken |
| Tide Level unlocks | Sand Dollars |
| Tide Marks track, Journal feats | Map |
| Keepsakes / Aquarium, cosmetics | Recruits |
| Settings, stats (runs, wins, per-critter wins) | |

---

## 7. Daily Tide (optional seeded run)

### 7.1 Format

- **Seed:** `hash("daily" + local date YYYY-MM-DD + ruleset major version)`. It's generated on the phone, so everyone with the same date and ruleset gets the same run. No server is involved.
- **Shorter:** 5 stops (3 battles, 1 non-combat, and a mini-boss). It takes about 10 minutes.
- **Fixed squad:** given by the seed and the same for everyone. **It can use critters you haven't unlocked yet**, which gives new players a taste of the whole roster.
- **Keepsakes and Tide Levels are disabled** so results are comparable.
- **First try counts for the share.** Replays are allowed and marked as practice.
- **No streaks, ever.** The stats screen shows "Tides played: 37", never "Day 12 streak". **Past Tides** is a calendar where you can play any previous day's Tide. Missing a day costs nothing.
- **Date honesty:** the device clock is trusted. Changing your clock gives you another day's Tide, which is fine, because Past Tides already allows that.

### 7.2 Share text (spoiler-free, Wordle-style)

The text never reveals map layout, enemy positions, or rewards. It only shows how well you did.

```
Slack Water · Daily Tide #142 · 1 Nov
🦀🐙⭐ first try
🟦🟦⬜🟨👑
Boss down in 5 rounds · 1 KO
1,840 pts · SW142-7KQ9-M2XD-A4
```

| Symbol | Meaning |
|---|---|
| 🦀🐙⭐ | Squad species. These are the same for everyone, so they confirm the day, not the run. |
| 🟦 | Battle won, no KOs |
| 🟨 | Battle won with at least 1 KO |
| 🟥 | Battle lost (run ended here) |
| ⬜ | Non-combat stop |
| 👑 / 💀 | Mini-boss beaten / lost |

**Plain-text variant** (for screen readers and colour-blind sharing, toggled in Settings): `Daily Tide 142: W W - W* BOSS(5) | 1840 | SW142-7KQ9-M2XD-A4`.

**Score:** 500 per battle won + 100 per HP remaining at the end + 200 per objective − 50 per round used. The formula is shown in-game.

### 7.3 Result code

`SW142-7KQ9-M2XD-A4` is `SW` + the Tide number in readable decimal (`142`), followed by 10 Base32 characters packing the fields below (47 bits + padding). It has no secrets and no server.

| Field | Bits |
|---|---|
| Ruleset version | 6 |
| Score | 14 |
| Per-stop outcome (5 × 2 bits) | 10 |
| Boss rounds | 4 |
| KOs | 4 |
| First-try flag | 1 |
| Checksum (CRC-8) | 8 |

- A friend can **paste the code** into the Daily screen. Their phone decodes it offline and shows a side-by-side comparison: "Riya: 1,840 · You: 1,620".
- **It isn't anti-cheat.** Any offline code can be forged, and we say so. The checksum only catches typos. There are no leaderboards, so there's nothing worth cheating for.

### 7.4 Share image

A **1080 × 1350** postcard rendered on the phone and shared through the OS share sheet:
- The top half is a painterly tidepool in the day's seeded palette, with the three squad critters as silhouettes.
- The middle is the outcome row, drawn as shapes as well as colours: ■ won, ▲ won with KO, ✕ lost, ○ non-combat, ♛ boss.
- The bottom has the score, the result code, the date, and the game's name. **No QR, no URL tracking, no app-store link baked in.** It's a postcard, not an ad.
