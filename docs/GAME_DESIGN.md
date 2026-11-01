# Game Design Document: *Slack Water* (working title)

> **Status:** Draft v0.2 · 2026-11-01. v0.2 applies the owner's decisions: *Slack Water* is the working title, the game is **completely free**, and the team is **the owner plus Claude**.
> **Audience:** the two-person team. Written to be read without prior context.
> **What this is:** the complete design for an original, fully offline, completely free mobile tactics roguelite with simultaneous turns and local 2-player play. No code exists yet.

## Contents

| # | Section | Where |
|---|---|---|
| 1 | Pitch, names, pillars, target player, inspirations | this file |
| 2 | Core loop and simultaneous resolution rules (the heart) | **[design/core-rules.md](design/core-rules.md)** · summary in §2 |
| 3 | Units, terrain, enemies, AI | [design/units-and-combat.md](design/units-and-combat.md) · summary in §3 |
| 4 | Roguelite runs and meta-progression | [design/runs-and-progression.md](design/runs-and-progression.md) · summary in §4 |
| 5 | 2-player: versus, co-op, pass-the-phone, Link Up pairing, trading | [design/two-player.md](design/two-player.md) · summary in §5 |
| 6 | Daily Tide | [design/runs-and-progression.md §7](design/runs-and-progression.md#7-daily-tide-optional-seeded-run) · summary in §6 |
| 7 | Theme, setting, art direction | this file |
| 8 | UX: onboarding, save anywhere, accessibility, controls | this file |
| 9 | Business model: free, and what we never do | this file |
| 10 | Technical design notes | [design/tech.md](design/tech.md) · summary in §10 |
| 11 | Scope, roadmap, effort, riskiest assumptions | this file |
| 12 | Open questions | this file |

## Key decisions at a glance

| Area | Decision |
|---|---|
| Genre | Roguelite tactics on a **6×6 grid**, **3 vs a few** units, **simultaneous secret planning**, fixed 5-step resolution. |
| Heart | **Attacks hit places, not units.** Weight settles tile contests. No dice in combat. Every outcome is explainable by one of 5 rules. |
| Solo | Complete game: **15-minute runs** of 8 stops, 3 Shores, 3 bosses, 10 critters, 10 Tide Levels. Every run earns progress. Unlocks take about 20 runs. |
| 2-player | **One phone** (pass-the-phone, once per round) is the default. **Two phones** via LAN/hotspot with a QR code (about 2 taps), with Bluetooth LE as a fallback. Versus and co-op. |
| Social | **Free, unlimited, face-to-face Keepsake trading.** Anyone with the app can Link Up with anyone else. Spoiler-free **Daily Tide** share code. |
| Theme | **Tidepool critters vs. the Murk.** Flat-vector, top-down, portrait, one-handed. |
| Business | **Completely free.** No purchases of any kind, no ads, no accounts, no tracking, no internet, ever. Everything is in the download. |
| Team | **The owner plus Claude.** Claude writes the code, tests, and code-built vector art. The owner holds the vision, makes the taste calls, and runs real-device tests and playtests. |
| Tech | Godot 4, integer-only deterministic sim, own seeded RNG, input-only lockstep over a pluggable `Link`. |
| Scope | Milestone-gated, not calendar-gated: risk spikes (pairing first), then an MVP slice, then v1.0. |

---

## 1. Pitch

### 1.1 One-liner

> **A pocket-sized tactics roguelite where everyone moves at once: outguess the tide on a 6×6 rockpool, alone in 15-minute runs or face to face with a friend, on one phone or two, no internet ever.**

### 1.2 Proposed names

| Name | Why | Concern |
|---|---|---|
| **Slack Water** *(chosen working title)* | "Slack water" is the still moment between tides, which is literally the planning phase before everything moves at once. It's distinctive, calm, and searchable. | It's a nautical term, so some players won't know it. That's fine: the tagline explains it. |
| **Tidepool Gambit** | Says the setting and the mind-game at once. | "Gambit" is common in game titles. |
| **Low Tide Link** | Nods to link-cable play. | Sounds like a utility app. |
| **Shell Game** | Hidden plans plus shells as gear. A perfect pun. | Generic phrase, so it's hard to own in store search and trademarks. |
| **Rockpool Rally** | Friendly, all-ages. | Undersells the depth. |

*Slack Water* is the working title. It still needs a store-search and trademark check before launch (see [Open questions](#12-open-questions)). The repo folder stays `link-tactics` for now.

### 1.3 Pillars

1. **Outguess, don't out-click.** Every round both sides plan in secret and everything resolves at once. Skill is reading habits and predicting tiles, never reflexes. There are no dice in combat, and every outcome can be explained.
2. **Together, no internet.** Two people in the same room can play on one phone or two, iPhone with Android, on a bus with no signal. Pairing takes about 2 taps, and trading is free and face to face.
3. **Respect the player.** Pay once. Pause anywhere. Complete a run in 15 minutes. Nothing nags, expires, or tracks you.

### 1.4 Target player

- **Primary:** 22–40-year-olds who grew up with handheld link cables and are tired of gacha and energy timers. They play in 10–30 minute gaps (commute, couch, lunch) and like Into the Breach, Balatro, and Slay the Spire. They often have a partner, sibling, or friend physically nearby who isn't a "gamer".
- **Secondary:** teens (13+) who want a game they can play with friends at school or on a trip with no data, no accounts, and no ads. The game is age-appropriate by design (no violence beyond cartoon bonks).
- **Not targeted:** competitive online ladder players. There's no online play by design.

### 1.5 Inspirations and how we differ

| Inspiration | What we take | How *Slack Water* is different |
|---|---|---|
| **Into the Breach** | Small grid, short battles, pushes as tactics, readability obsession | ItB is **perfect information, player-acts-after-enemy-telegraphs** on an 8×8 board: a solvable puzzle. We are **simultaneous with partial information** (Tells, not exact targets) on 6×6, so the skill is prediction. We add PvP and co-op, which ItB has no equivalent for. We have no mechs, buildings, or time travel. |
| **Frozen Synapse** | Simultaneous secret turns, then a shared resolution | FS is continuous-space squad shooting with line of sight and real-time-like ticks, sold on PC. We are a **discrete 6×6 grid with integer rules**, melee and lane abilities, a roguelite structure, and **local, offline 2-player**. |
| **Diplomacy** | Simultaneous orders and the "standoff bounce" | We openly borrow the bounce idea and add **Weight** so contests aren't always a standoff. It's tactical units on a grid, not territory negotiation. |
| **Pokémon (GBA era)** | The feeling of link-cable trading in person | **No catching, no collecting-to-complete, no 1v1 menu battles, no type chart.** We trade **Keepsakes** (individual named critters you finished runs with), and only face to face. Battles are grid tactics. |
| **Advance Wars / Fire Emblem** | Grid tactics on handhelds, unit roles | Those are **alternating turns** (I-go-you-go) with long campaign maps. We have **no turn order**, tiny boards, and 2-minute battles inside 15-minute roguelite runs. There's no story campaign or permadeath roster drama. |
| **Slay the Spire / Balatro** | Roguelite run structure. The premium, no-ads business model | **No cards, decks, hands, or draws.** Everything you own is visible units, Shells, and Molts. Our map is a 2-choice-per-stop Tide Chart, not a branching node map. |
| **The Battle of Polytopia** | Mobile strategy with pass-and-play | Polytopia is a 4X empire game with alternating turns. Ours is squad tactics with simultaneous turns, which removes pass-and-play's constant phone-swapping. |
| **RoboRally** | Simultaneous programmed movement and collisions | RoboRally programs movement **with cards** and has chaotic chain reactions. We have **one order per unit**, no cards, and deterministic, readable resolution. |
| **Hoplite** | Mobile roguelike tactics, short sessions | Hoplite is a single hero on hex, player-first, solo only. We are a squad on a square grid, simultaneous, with 2-player. |
| **Wordle** | Spoiler-free daily share text | We use the same idea for the Daily Tide, with **no streaks** and a playable archive of past days. |

---

## 2. Core loop (summary: full rules in [design/core-rules.md](design/core-rules.md))

**Moment to moment (one round, about 20 s):**

```
PLAN (secret)  →  LOCK IN  →  RESOLVE:  Brace → Fast → Move → Strike → Tide
```

- Each unit gets **one order**: an optional **Move** (path) plus one **Action**: a **Fast** ability (fires *before* anyone moves, from the start tile) **or** a **Slow** ability (fires *after* moves, from wherever the unit ended up). A unit with no orders **Hunkers** (takes 1 less damage per hit).
- **Attacks hit places, not units.** Spot abilities aim at a tile, and Lane abilities aim in a direction. If the target isn't reachable from where you actually end up, the ability fizzles, and the replay says why.
- **Movement resolves in ticks.** Two units entering one tile: heavier wins, equal weight means both stop. Swaps are standoffs. Units never pass through each other. Following a unit that's leaving its tile works.
- **Damage in a step lands all at once.** Mutual KOs happen. Units KO'd in Fast never act. Units displaced in Fast lose their move.
- **No dice.** The rules are order-independent and symmetric, and the step-by-step playback has a "Why?" log.

**Battle loop (about 2 min):** up to 6 rounds → win (KO all Murk, or survive until they retreat) → pick a reward.
**Run loop (about 15 min):** 8 stops on a Tide Chart → boss → Keepsake and Tide Marks.
**Meta loop:** Tide Marks unlock critters, Shores, Shells, and Molts. Tide Levels raise difficulty. Keepsakes fill your Aquarium and can be traded.

The design doc includes **three fully worked example turns** ([core-rules.md §8](design/core-rules.md#8-worked-example-turns)) covering weight contests, standoffs, swaps, mutual trades, Fast interrupts, push-into-blocker bump KOs, and fizzles.

## 3. Units and combat (summary: full detail in [design/units-and-combat.md](design/units-and-combat.md))

- **Board:** 6×6, orthogonal only, 7 terrain types (Sand, Rock, Kelp, Shallows, Deep, Current, Pearl). The maps are hand-authored templates with seeded variations.
- **Critters:** MVP 6 (Hermit Crab, Mantis Shrimp, Octopus, Pistol Shrimp, Sea Star, Sea Urchin). v1.0 adds 4 (Moray Eel, Pufferfish, Cuttlefish, Anemone). Each has HP 3–6, Move 0–3, Weight 1–3, exactly one Fast and one Slow ability, at most one trait, and two Molt upgrades.
- **Shells:** gear that modifies rules (10 in MVP, 22 in v1.0).
- **The Murk:** 5 enemies in the MVP and 8 in v1.0. Each follows a printed one-line **Habit** and shows a **Tell**. There's 1 boss in the MVP and 3 in v1.0, each with a visible pattern and an 8-round limit.
- **AI:** Habit-driven intent plus small deterministic scoring. It plans before you act and never sees your plan. It predicts your moves with a small, weighted guess set that gets sharper at higher Tide Levels.

## 4. Roguelite runs (summary: full detail in [design/runs-and-progression.md](design/runs-and-progression.md))

- **Run:** about 15 min, 8 stops along one Shore. Each fork offers 2 options of the same category, so the path is always 5 battles + 2 non-combat stops + a boss.
- **Squad:** start with 3 critters, grow to 4, field 3 per battle. HP carries over between battles. **Losing:** a wipe, or failing the boss's 8-round limit.
- **Rewards:** Sand Dollars plus pick 1 of 3 (Recruit, Shell, Molt). Objectives give bonuses and never cause failure.
- **Difficulty:** an enemy budget that grows each stop, and **Tide Levels 0–10**, each adding one named modifier.
- **Meta:** **Tide Marks** from every run (including losses) fill one fixed 14-step track. That's about 20 runs or about 5 hours to unlock everything. Journal feats unlock cosmetics. There's an **Unlock everything** toggle in Settings. Winning a run lets you **bottle a Keepsake** for your Aquarium.
- **Never:** grind walls, time gates, energy, premium currency, streaks.

## 5. Two-player (summary: full detail in [design/two-player.md](design/two-player.md))

| Mode | One phone | Two phones (Link Up) |
|---|---|---|
| **Versus** (3v3, Pearls, about 7 min) | Pass-the-phone privacy screens, **each player plans once per round** | Commit-then-reveal plans, so nobody can wait for the other's plan |
| **Co-op run** (2 critters each vs. AI) | Plan together on one screen | See your partner's plan as live ghosts, plus tile pings |
| **Trading** | n/a (one profile per phone) | Free, unlimited, face to face. Two-phase commit plus journal and tombstones |

- **Pairing:** a **Link Up** button shows your QR immediately. Your friend taps Link Up and then either **Join** (Nearby list) or **Scan**, which is about 2 taps. Specific failure screens (F1–F8) always offer a hotspot, Bluetooth, or "Play on one phone". **"Continue on one phone"** rescues any dropped match, because both phones hold the full state.
- **Anyone can join anyone:** the game is free and everyone has all content. In a Link session, the critters and Shores available are those the **host** has unlocked, so a newcomer can play with a veteran friend's full roster on day one.

## 6. Daily Tide (summary: full detail in [runs-and-progression.md §7](design/runs-and-progression.md#7-daily-tide-optional-seeded-run))

- It's seeded from the local date on the phone. Everyone gets the same 5-stop run and the same fixed squad (**from the whole roster, even critters you haven't unlocked yet**), taking about 10 min.
- **Share text:**

  ```
  Slack Water · Daily Tide #142 · 1 Nov
  🦀🐙⭐ first try
  🟦🟦⬜🟨👑
  Boss down in 5 rounds · 1 KO
  1,840 pts · SW142-7KQ9-M2XD-A4
  ```

- There's also a plain-text variant, and a **1080×1350 postcard image** that uses shapes as well as colours. A friend can paste the result code to compare offline.
- **No streaks.** We show a total "Tides played", and a calendar lets you play any past day.

---

## 7. Theme, setting and art direction

### 7.1 Theme options

| Option | Fantasy | Pros | Cons |
|---|---|---|---|
| **A. Tidepool (recommended)** | Rockpool critters defend their pool from **the Murk**, a living silt. Each run is a walk down the shore at falling tide. | The mechanics *are* the theme: **Weight** (heavy crabs hold ground), **Currents**, **Deep** (swept away instead of killed), **Shells as gear**, and **tide as the round limit**. It's cozy-adjacent, which is a strong market. It's non-violent and all-ages. Shell-trading mirrors real beach-combing. Silhouettes are naturally distinct. | Sea-critter games exist, so execution has to be distinctive (see 7.2). |
| B. Paperfold | Origami figures fight inkblots across a school desk. | Strong school/link-cable nostalgia. Papercraft is cheap to draw. | Papercraft is a common indie look. The mechanics feel arbitrary (why does paper have weight?). |
| C. Signal Garden | Tiny radio-robots on a circuit-board garden fight Static. | Literal "link" theme. Easy hard-surface art. | Cold and generic. Robots are an overused theme, and it has weaker cozy appeal. |

**Decision: A, Tidepool.** It's the only option where the rules and the world explain each other, which lowers tutorial cost and raises readability.

### 7.2 Setting and tone

- **World:** the shoreline of a small, nameless island. Shores are biomes: **Rockpool** (bright, starter), **Kelp Shallows** (green-gold, cover and currents), and **Wreck Reef** (teal-rust, deep water and hazards).
- **The Murk** animates debris into creatures with glowing eyes. When defeated, it **dissolves into clear water** and the tile visibly brightens. Winning literally cleans the board.
- **Tone:** warm, a little funny (critter names like "Pinch the Stubborn"), never preachy. Critters don't die; they're "knocked out" or "swept away" and retreat into crevices.

### 7.3 Art style (achievable by a tiny team)

- **Built in code first.** Every critter, enemy, tile, and effect is made from Godot shapes (`Polygon2D`, `Line2D`, `Path2D`) or hand-written SVG, so Claude can make and iterate on all of it. The owner directs the look and approves it. If a style test shows code-built art isn't good enough, we bring in a contract artist for key art only (see Open questions).
- **Flat vector, top-down orthographic, with chunky shapes.** Each critter is about 5–8 vector parts with thick, consistent outlines, one fill and one shadow tone. Each critter has a **unique silhouette**, readable at 54 pt.
- **Animation is tween-based, not frame-by-frame:** squash and stretch, bobbing, part rotation (Godot `AnimationPlayer`/`Tween`). A critter takes about 1–2 days to build, not weeks.
- **The board is the hero:** sand and rock tiles with one **water-caustics shader** over the whole board (cheap, gorgeous), plus gentle ripples for currents.
- **VFX:** about 12 hand-made one-shot effects (hit splash, ink burst, fizzle puff, bounce spiral, swept wave, and so on).
- **Palette:** each Shore has a 6-colour palette plus a fixed **team-colour pair** (Coral vs. Teal) that's checked for colour-blind safety. Teams are also distinguished by **shape**: the player's base ring is round and the Murk's is jagged.
- **Audio:** kalimba, marimba, and soft percussion. Sound effects are generated (sfxr-style) or taken from CC0 libraries. Music comes from free CC0 tracks. Each resolution step has its own sound cue so the order becomes audible. Haptics on hits, bounces, and lock-in.
- **Target app size:** 150 MB or less.

### 7.4 Portrait, one-handed layout

```
┌──────────────────────────────┐
│ ⏸  Round 3/6   ●●○○○ Tide   │  ← top bar: pause, round, step indicator (thin)
│                              │
│   ┌──────────────────────┐   │
│   │                      │   │
│   │      6 × 6 BOARD     │   │  ← upper-middle: square, full width
│   │                      │   │
│   └──────────────────────┘   │
│  Murk: 🔹Mite→Hermit 🔸Lurker?│  ← Tells strip (tap = threat map)
│ ──────────────────────────── │
│  🦀 Hermit  ♥6  ⚖3  🐚Cowrie │  ← selected critter
│  [ Move ] [ Fast ] [ Slow ]  │  ← thumb zone
│  [◀ prev]  [↶ Undo] [next ▶] │
│  [■■■■ Hold to Lock In ■■■■] │  ← biggest control, bottom
└──────────────────────────────┘
```

All interactive controls except board taps sit in the **bottom 40%**. **Left-handed mirror** is in Settings.

---

## 8. UX

### 8.1 Onboarding and tutorial

- **No text walls, no forced account, no name entry before play.** The first launch goes straight into **"First Tide"**: 3 tiny battles of about 4 minutes in total, each teaching *one* idea through play.
  1. **Plan, then everything moves.** You and a Mite both move. Your first attack probably misses, and the replay shows why. The lesson is "aim where it *will* be."
  2. **Weight and bouncing.** A Hermit and a Mite contest a tile, then two equal Mites stand off.
  3. **Fast vs. Slow.** The Octopus grabs a Bulwark before it moves.
- After that, the first real run starts on Rockpool with a hint layer: contextual one-liners shown once, and the **"Why?" log** pulses after the first surprising outcome.
- The **Tide Rules** sheet (the 5 rules) is always one tap away from the pause menu.
- Link Up and pass-the-phone have their own 20-second interactive explainer the first time each is opened.
- The tutorial can be skipped by experienced players.

### 8.2 Pause and resume (save anywhere)

- **Every input is saved** (see [tech.md §8](design/tech.md#8-save-format)). Killing the app, getting a phone call, or a dead battery resumes on **exactly the same screen**, including a half-planned round.
- Slots: one solo run, one Daily Tide, one co-op run per partner, and one versus match. Starting a new solo run while one exists asks first.
- **No timers anywhere by default.** Planning can take as long as you like. Link play pauses for both players when either app goes to the background.

### 8.3 Accessibility

| Feature | Detail |
|---|---|
| **Colour-blind safe** | Team colours checked for deuteranopia, protanopia, and tritanopia. **Shapes and patterns** back up every colour meaning (base rings, outcome icons, share postcard). |
| **Text size** | 3 sizes. The layout reflows and the board shrinks before text gets clipped. |
| **Reduced motion** | No screen shake, no caustic animation, and instant tweens. Playback defaults to Step mode. |
| **Step playback** | Tap to advance each resolution step. Replay any round. |
| **Honest Tides** (assist) | Shows the **exact** AI targets, which turns solo into a perfect-information puzzle. Marked on the run summary, with no shame. |
| **Undo plan** | Unlimited before lock. There's no "undo round" in normal play. The **Second Look** assist lets you redo one round per battle. |
| **One-handed** | Bottom-weighted controls and a left-handed mirror. |
| **Haptics and sound cues** | Each step has a distinct sound and haptic pattern. Both can be toggled independently. |
| **Screen reader** | Menus and the "Why?" log are text-first. Board narration ("Hermit at c2, 6 HP") is a v1.x goal, depending on Godot's mobile accessibility support **(verify)**. |
| **Plain-text share** | For screen readers and colour-blind friends. |
| **No time pressure** | Except the opt-in Link versus chess clock. |

### 8.4 Controls on small screens

- **Select:** tap a critter, or use prev/next.
- **Move:** drag from the critter along tiles. The path snaps orthogonally, stays within Move, and shows a ghost. Tapping a destination tile auto-paths (shortest path, with ties going clockwise from north, shown before confirming).
- **Act:** tap **Fast** or **Slow**. Valid targets light up. Tap a tile for Spot aim, or tap one of 4 arrows for Lane aim.
- **Inspect:** long-press any tile or unit to open a bottom sheet with a sticky header and an always-visible close ✕.
- **Lock In:** a 0.5 s hold with a fill animation. The button warns (yellow) if a critter is unplanned ("Will Hunker").
- **Minimums:** 54 pt tiles at 360 pt width, and 48 pt buttons. Supports **4.7" and larger phones**. Tablets show the portrait layout centred, and landscape comes after launch.

---

## 9. Business model: completely free

### 9.1 Model

**Slack Water is completely free on iOS and Android.** There's one download with everything in it, and nothing to buy. We're not trying to make money from it; the aim is to make the honest offline game players keep asking for.

What this simplifies:
- **No demo/full split**, no store purchase code, no entitlement cache, and no "In the full game" labels.
- **Anyone can Link Up with anyone.** No guest/owner distinction exists. Every phone with the app is a full host.
- **Trading** never needs "Held for you" states. Every received Keepsake is playable straight away.
- **The store page pitch is simple:** "Free. No ads. No purchases. No internet. Ever."

### 9.2 Things we will never do

- Ads of any kind, including "watch an ad for a reward"
- In-app purchases of any kind: no unlocks, consumables, currencies, loot boxes, gacha, battle or season passes, or subscriptions
- Energy, stamina, timers, or "come back in 4 hours"
- Daily login rewards, streaks, or FOMO events
- Accounts, logins, or online requirements. **It works in airplane mode forever.**
- Analytics SDKs, ad IDs, tracking, or data collection. Crash diagnostics are a **file the user can choose to export**.
- Paying for trades or trade slots, or limiting trades
- Engagement push notifications
- Locking solo content behind multiplayer

### 9.3 Ongoing support

- We patch for OS compatibility after each major iOS and Android release, on a best-effort basis, because "it broke after an update" is players' top complaint.
- Device coverage is the owner's own phones plus borrowed friends' phones (aim for at least one older and one newer phone per OS).
- **Open source.** The code is public, so the game can outlive us and others can learn from the netcode. The licence is still to be chosen (see Open questions).

### 9.4 Costs we still carry

- Apple Developer Program, $99 a year, and a Google Play developer account, $25 once. *(Verify current fees before enrolling.)*
- Optional: a contract artist or composer, only if code-built art or CC0 audio falls short.

---

## 10. Technical design notes (summary: full detail in [design/tech.md](design/tech.md))

- **Determinism:** integers only in Sim and AI. Our own **xoshiro128\*\*** RNG with separate seeded streams. No clocks or floats. Canonical unit ordering. Rules are order-independent, which is proven by property tests. Ruleset data is versioned and hashed, and **old rulesets ship in new builds** so offline phones on different versions can still play.
- **Netcode:** input-only lockstep. Plans are under 20 bytes per round. Versus uses **commit-then-reveal**. A state hash is checked every round, and a mismatch triggers an automatic snapshot resync. A session layer with seq and ack makes reconnects transport-agnostic.
- **Transport:** one `Link` interface with `LocalLink` (same phone and tests), `LanLink` (TCP + UDP beacon, QR handshake), and `BleLink` (native plugin).
- **Saves:** JSON, **snapshot + input log** (resume = replay, which is also a determinism check), atomic writes, 2 backups, schema migrations.
- **Risks:** pairing UX, the iOS multicast entitlement, Wi-Fi client isolation, BLE plugin effort, engine-upgrade determinism. All are mitigated and listed in [tech.md §11](design/tech.md#11-technical-risks).

---

## 11. Scope and roadmap

**Team: the owner plus Claude.**

| | Owner | Claude |
|---|---|---|
| Owns | Vision, taste and design calls, final say on every gate | Code, tests, tooling, code-built vector art, docs |
| Does | Runs builds on real iPhones and Android phones, recruits and runs playtests, records feedback, handles store accounts and signing | Sim, AI, UI, netcode, transports, native plugins, content data, balance tooling, debug overlays |
| Can't delegate | Physical device testing, pairing tests with real people, the "is this fun?" judgement | n/a |

Estimates are in **milestones, not weeks**. Claude's coding speed isn't the bottleneck. The bottlenecks are the owner's time for device testing and playtests, and the iteration loops those need. Each phase ends at a gate the owner signs off.

### 11.1 Phases

#### Phase 0: Risk spikes ✦ *prove the scary parts first*

- [ ] **P0 Paper prototype:** printed 6×6 grid, tokens, a printed sheet of the 5 Tide Rules, 4 critters vs. 4 "Habit" enemies run by a human. The owner runs 3–5 sessions with friends. Claude prepares the printables.
- [ ] **P1 Pairing spike:** a bare Godot app on iPhone + Android. QR show/scan, TCP over LAN, the Nearby beacon, Android LocalOnlyHotspot + Wi-Fi QR, reconnect, and the F2 diagnosis. Claude builds it; the owner runs the pairing test protocol in §11.3.
- [ ] **P2 Sim core + determinism:** headless `resolve_round`, all edge-case tests, the 10k-battle golden hash suite, run in CI and then on the owner's iOS and Android phones.
- [ ] **P3 Greybox solo battle:** planner UI, playback, 3 enemies with Habits and Tells. Fun check with the owner plus 4–8 friends.
- [ ] **P4 Art style test:** Claude builds 2 critters, 1 Murk enemy, and the board in code-built vector. The owner decides whether it's good enough or we bring in an artist.
- **Gate:** continue only if P0/P3 fun criteria and P1 pairing criteria pass. Otherwise redesign (rules) or re-scope (Link becomes same-phone only at launch).

#### Phase 1: MVP vertical slice

- [ ] Rockpool Shore: 12 maps, 5 enemies, the Undertow boss
- [ ] 6 critters with Molts, 10 Shells
- [ ] Full 8-stop run, Tide Chart, rewards, Market/Rest/Stranger
- [ ] Tide Marks track (first 6 steps), Tide Levels 0–3
- [ ] Save anywhere (snapshot + log), First Tide tutorial
- [ ] **Same-phone versus** (pass-the-phone) with 3 arenas
- [ ] **Link Up versus over LAN/hotspot** with QR (from P1)
- [ ] Art for Rockpool and the 6 critters, core VFX and audio
- [ ] Accessibility basics: colour-blind palette, text size, step playback
- **Out of MVP:** co-op, trading, Keepsakes, Daily Tide, BLE, other Shores.
- **Gate demo:** a 15-minute solo run on device, and a versus match between an iPhone and an Android phone in airplane mode using a hotspot.

#### Phase 2: v1.0

- [ ] Kelp Shallows and Wreck Reef Shores, 2 more bosses, 3 more enemies, 24 more maps
- [ ] 4 more critters, 22 Shells total, alternate Molts, Tide Levels to 10
- [ ] **Co-op runs** (one phone and Link)
- [ ] **Keepsakes, Aquarium, trading** (with crash-recovery tests)
- [ ] **Daily Tide**: share text, image, and code compare
- [ ] **BLE fallback**: not in v0.1, but it ships before v1.0. Start with a timeboxed spike early in Phase 2 so there is time to iterate.
- [ ] Journal feats and cosmetics, full accessibility pass, localization (see Open questions)
- [ ] **Open beta** via TestFlight and Google Play internal or closed testing with friends, then launch

#### Phase 3: Post-launch

- [ ] OS-compat patches (best effort, per §9.3)
- [ ] Free content update: 4th Shore, 2 critters, versus arenas
- [ ] Tablet landscape layout, screen-reader board narration
- [ ] "Move to new phone" over Link Up
- [ ] Link Cup: an on-device bracket for 3–8 friends sharing phones (no server)

### 11.2 Relative size by workstream (to v1.0)

These are relative weights to help prioritise, not calendar time.

| Workstream | Size | Who's the bottleneck |
|---|---|---|
| Sim, rules, tests | M | Claude |
| AI (Habits, prediction, bosses) | M | Claude, with the owner's playtest feedback |
| Battle UI: planner, playback, "Why?" log | L | Owner's feel feedback on device |
| Run, map, rewards, meta, saves | M | Claude |
| Content design and balance | L | Playtests |
| Art (code-built vector) | L | Owner's taste calls |
| Audio | S | Sourcing |
| Link Up: LAN, QR, hotspot, session, recovery | L | Owner's device testing |
| BLE plugin | M | Owner's device testing |
| Co-op, trading | M | Two-device testing |
| Daily Tide | S | Claude |
| Accessibility and localization | M | Translators or reviewers |
| QA and device testing | L | Owner |
| Store pages, screenshots, trailer | S | Owner |

### 11.3 Riskiest assumptions and how to test them first

| # | Assumption | Test (when) | Pass criteria |
|---|---|---|---|
| 1 | **Ordinary people can pair two phones across iOS/Android in about 2 taps** | **P1 pairing spike** (Phase 0): 10 pairs of non-gamer friends, mixed OS, in 3 settings (home Wi-Fi, one phone's hotspot, café/school Wi-Fi). No help given. We time it and note where they get stuck. | Home Wi-Fi: 8/10 linked in under 60 s. Hotspot: 6/10 in under 3 min with the helper. Isolation is correctly diagnosed in 100% of cases. |
| 2 | **Simultaneous resolution is readable and fun on 6×6** | **P0 paper prototype**, then **P3 greybox** (Phase 0) | After 3 rounds, players predict the result of a shown plan correctly at least 70% of the time. At least 6/8 ask for another game. |
| 3 | **Solo vs. AI with partial info feels fair, not random** | P3 greybox, asking after each loss: "Was that your mistake or bad luck?" | At least 75% answer "my mistake" or "I misread". |
| 4 | **Determinism holds across devices and engine builds** | P2 golden suite on 4 real devices | Identical final hash on all devices for 10k battles. |
| 5 | **Pass-the-phone versus isn't tedious** | MVP playtest with 6 couples or friend pairs | Median round under 60 s. 4/6 would play it again on their own. |
| 6 | **Code-built vector art looks good enough to attract players** | P4 art style test in Phase 0. The owner shows screenshots to 5 people next to 2 comparable store listings. | The owner is happy with it, and at least 3/5 would tap the screenshot. Otherwise bring in an artist for key art. |
| 7 | **BLE is reliable enough to ship** | Timeboxed spike early in Phase 2 | It connects in under 15 s at least 80% of the time across the 4 lab devices. If not, it moves to post-launch. |

### 11.4 Next documentation steps (per AGENTS.md Phase 1)

After this design is approved: `docs/architecture.md` (from [tech.md](design/tech.md)), ADRs (engine, determinism/RNG, transport, save format, free model), `docs/roadmap.md` with granular status, and `README.md`.

---

## 12. Open questions

**Decided (2026-11-01):**
- *Slack Water* is the working title.
- The game is completely free.
- The team is the owner plus Claude.
- The project is **open source**.
- **No tip jar** in v1.0.
- **Bluetooth (BLE)** is not in v0.1, but ships before v1.0.
- **Music** uses free (CC0) tracks.

Still open. **My conviction** is noted on each.

| # | Question | My conviction |
|---|---|---|
| **1** | **Name check:** run a store-search and trademark check on *Slack Water*. | Do it before Phase 1 art, so we don't build branding twice. *Tidepool Gambit* is the backup. |
| **2** | **Licence:** which open-source licence for the code, and which for the art? | **MIT** for code, **CC BY 4.0** for art. Keep the name and icon as reserved trademarks so nobody ships a copy under the same name. |
| **3** | **Open source from day one, or at launch?** | From day one, in a public repo. It costs nothing, and progress in the open builds interest. |
| **4** | **Art:** is code-built vector good enough, or do we hire an artist for key art? | Decide after the P4 style test. Start code-built. |
| 5 | Launch platforms: simultaneous, or iOS first? | Simultaneous. Cross-play is the headline feature. |
| 6 | Multiple profiles per phone (siblings sharing a device)? | Not in v1.0. Revisit post-launch. |
| 7 | Launch languages beyond English? | English only at launch. Add languages when volunteers offer translations. |
| 8 | Zero telemetry, even opt-in? | Zero, with an exportable diagnostic file only. It's a selling point. |
| 9 | Tablet landscape in v1.0? | No. Post-launch. |
