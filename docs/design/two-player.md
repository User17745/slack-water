# Two-Player: Versus, Co-op, Link Up, Trading

> Part of the [Game Design Document](../GAME_DESIGN.md). Message formats are in [tech.md](tech.md#6-netcode-messages).

**Defaults:** same-phone play is the default and always one tap away. Two-phone **Link Up** is optional and must connect in about 2 taps. Nothing in solo ever requires a second player.

---

## 1. Versus duel

| Rule | Decision |
|---|---|
| Board | 6×6 **symmetric arenas** (MVP 3, v1.0 8), each with 3 Pearl tiles on the centre rows. |
| Squads | **Snake draft** from 8 random critters drawn from the host's unlocked pool: A1, B2, A2, B1 (picks) → 3 each. Then each player picks 1 Shell per critter from 3 offered, **simultaneously and blind**. |
| Alternative | **Quick match:** mirrored random squads, no draft. Good for first-timers. |
| Win | First to **6 Pearl points**, **or** KO all enemy critters. A Pearl scores 1 point in the Tide step if only your critter stands on it. |
| Round limit | 10 rounds. Tiebreak order: points → total remaining HP → critters alive → draw. |
| KO | Permanent for the match. No healing beyond what critters can do themselves. |
| Keepsakes | Off by default ("Fair rules"). The **Bring-your-own** toggle allows each player one Keepsake. |
| Match length | About 6–8 minutes. **Best of 3** is optional. |
| Timer | Off by default. The optional chess clock gives 30 s per plan plus a 60 s reserve. On timeout, unplanned units Hunker. |

> **Why Pearls?** Pure elimination with simultaneous turns turns into "both sides wait" stalemates. Pearls force units to commit to known tiles, which is exactly where prediction gets interesting.

## 2. Co-op runs

- **The same roguelite run as solo** (8 stops, same Shores) with **4 critters on the board, 2 per player**. The enemy budget is ×1.5 and the boss has +50% HP.
- **Squad:** each player picks 2 starters from **the host's** unlocked roster. Each player's own Keepsake is allowed if Keepsakes are unlocked for that player.
- **Planning together:** there's no hidden information between partners.
  - **Same phone:** one board. Either player can plan any critter (each critter has a coloured ring for its "owner", which only matters for rewards). There's one Lock In.
  - **Two phones:** each phone plans its own 2 critters. **The partner's plan appears live as coloured ghosts**, and you can **ping** any tile with a long-press ("I'm going here", "Hit this"). Each player locks separately, and the round resolves when both lock.
- **AI plans are computed identically on both phones** from the shared seed, so they're never sent over the network.
- **Map choice:** both players vote. If you disagree, the **lead alternates** each stop (shown as a small crown).
- **Rewards:** each player picks from **their own** 1-of-3 offer for their own critters. Sand Dollars are shared.
- **Failure:** the run ends when all 4 fielded critters are KO'd in one battle, or the boss isn't beaten in time.
- **Progression:** both players earn Tide Marks on their own profiles. A guest earns Marks too (see §5).
- **Save:** both phones store the run. Either player can resume it with the same partner, or **finish it solo on either phone** (the AI takes no control; you play all 4 critters).

## 3. Same-phone flow (pass the phone)

The goal is that **each player plans once per round**, and the phone changes hands only twice per round.

```
┌───────────────┐   ┌───────────────┐   ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│  PASS TO BLUE │ → │ BLUE PLANNING │ → │  PASS TO RED  │ → │  RED PLANNING │ → │  BOTH LOCKED  │
│ full-screen   │   │ board + own   │   │ full-screen   │   │ board + own   │   │ "Put the phone│
│ blue tide     │   │ ghosts only   │   │ red tide      │   │ ghosts only   │   │  between you" │
│ "Hold to plan"│   │ Hold Lock In  │   │ "Hold to plan"│   │ Hold Lock In  │   │ Tap to reveal │
└───────────────┘   └───────────────┘   └───────────────┘   └───────────────┘   └───────────────┘
                                                                                        ↓
                                                                     Resolution plays for both
```

Rules and details:
- **Hold to start (0.6 s)** on the privacy screen. A tap won't open it, which prevents accidental reveals while the phone is handed over.
- **The privacy screen is opaque.** It's a solid colour flood with the player's name and critter emblem, and no board is visible behind it.
- **Once locked, a plan can't be reopened.** The planner shows only "Locked ✓" for that player.
- **Who plans first alternates each round.** There's no information advantage either way, but it feels fair.
- **Resolution is watched together.** Afterwards, each player can rewatch the round.
- **Peek protection while planning:** if the app goes to the background mid-plan, it returns to that player's privacy screen.
- **Co-op on one phone skips all of this,** since there's nothing to hide.
- **Accidental-lock safety:** after locking, "Undo lock" is available for 3 s, *before* the pass screen appears.

## 4. Two-phone pairing: "Link Up"

### 4.1 Transports, in order of preference

| # | Transport | When | Notes |
|---|---|---|---|
| 1 | **Same Wi-Fi (LAN)** | Both phones are on one network | TCP for the session. A UDP beacon handles the "Nearby" list. |
| 2 | **Phone hotspot** | No shared Wi-Fi (bus, park) | It's just LAN on the hotspot's network. The app helps set it up. |
| 3 | **Bluetooth LE** | Hotspot isn't possible or allowed | Slow (only fine because we send tiny inputs). A native plugin is required. |

The main menu has a **Link Up** button. It goes straight to the screen below. There's no "Host or Join?" question.

### 4.2 Screen by screen (happy path)

**Screen L1: Link Up.** The same screen on both phones.

```
┌──────────────────────────────┐
│  Link Up                  ✕  │
│                              │
│      ┌──────────────┐        │
│      │   QR CODE    │        │   ← this phone's code, already live
│      │              │        │
│      └──────────────┘        │
│   CORAL · MAPLE · 41         │   ← spoken/typed fallback
│                              │
│  Nearby                      │
│  ◉ Abhi's phone      [Join]  │   ← auto-discovered on the same network
│                              │
│  [ 📷 Scan a friend's code ] │   ← big, thumb zone
│  Not working? Get help       │
└──────────────────────────────┘
```

- **Phone A (host by default):** taps **Link Up**. That's 1 tap. Its QR is showing and it's listening.
- **Phone B:** taps **Link Up**, then either:
  - taps **Join** next to "Abhi's phone" (2 taps, no camera), or
  - taps **Scan**, then points at A's QR (2 taps plus aiming).
- **Whoever scans or joins becomes the guest.** Neither player has to understand the word "host".
- **If B joined from the Nearby list** (no QR), A sees a one-tap prompt: "Riya's phone wants to link. **[Accept]**". If B scanned the QR, there's no prompt, because the QR's key proves B is physically there.

**Screen L2: Linking (under 2 s typically).** A wave animation runs between two critter avatars with the text "Linking to Abhi's phone…". It times out after 6 s, which leads to F2.

**Screen L3: Lobby** (both phones). This is the same layout as the same-phone lobby.

```
┌──────────────────────────────┐
│  🦀 Abhi   ⟷   Riya 🐙  ✕    │
│  Linked · Wi-Fi · Strong     │
│                              │
│  [ Versus ]   [ Co-op run ]  │
│  [ Trade Keepsakes ]         │
│  Continue: Co-op, stop 5 →   │   ← if a saved link game exists with this partner
└──────────────────────────────┘
```

Either player can pick a mode, and the other gets a one-tap **Ready** confirmation.

### 4.3 First-run permissions (explain first, then ask)

| Permission | When it's asked | Our explainer copy |
|---|---|---|
| iOS **Local Network** | The first time Link Up opens on iOS | "To find your friend's phone on this Wi-Fi, iPhone needs to ask once. We never use the internet." |
| **Camera** | The first Scan tap | "To read your friend's code. Nothing is recorded." |
| **Bluetooth** | Only when the Bluetooth fallback is chosen | "For linking without Wi-Fi. Slower, but works anywhere." |
| Android **Nearby devices / location** | Only for BLE or the hotspot helper on versions that need it | Plain one-liner about why. |

If a permission is denied, show the specific fix ("Allow in Settings → [Open Settings]") and the **other routes**. Never dead-end.

### 4.4 Failure and recovery screens

| # | Trigger | Screen copy (short) | Actions |
|---|---|---|---|
| **F1** | Camera denied or QR unreadable | "Can't use the camera. Type the 3 words on your friend's screen instead." | Word entry with autocomplete, Open Settings |
| **F2** | No connection after 6 s | "Can't reach Abhi's phone." plus an **automatic check** with one line of result: *Not on Wi-Fi* / *You're on different networks* (subnet mismatch with the QR's IPs) / *Local network access is off* / *This Wi-Fi blocks phones from seeing each other* (connection refused or timed out on the same subnet, common on school and café Wi-Fi) | **Use a hotspot** · **Try Bluetooth (slower)** · **Play on one phone** · Retry |
| **F3** | Hotspot helper | **Android host:** "Tap to create a game hotspot." The app starts a local-only hotspot and shows a **Wi-Fi QR** that iPhone and Android cameras can join natively. Once the friend joins, the screen switches to the game QR automatically. **iPhone host:** iOS doesn't let apps create hotspots, so we show 3 illustrated steps (Settings → Personal Hotspot → friend joins), then return to L1. Tip shown: "If one of you has Android, let Android make the hotspot." | Next, Back |
| **F4** | Bluetooth route | Both phones show "Looking for nearby phones…". The guest list shows "Abhi's phone (Bluetooth)", and you tap to join. Banner: "Bluetooth link: fine for battles, trading takes a few seconds." | Join, Back |
| **F5** | Version mismatch | Minor difference: auto-negotiates the older ruleset, with an info line "Playing with v1.2 rules." Major difference: "Abhi's game is too new for yours. One of you needs to update, then you can play anywhere offline." | OK |
| **F6** | Link lost mid-match | A non-blocking overlay, "Link lost, reconnecting…", auto-retries for 30 s on all known routes. The game is paused for both players. | **Keep trying** · **Continue on one phone** (either phone switches to pass-the-phone with the full current state) · **Save & stop** (both phones save, and resume when linked again) |
| **F7** | Desync detected (state hashes differ) | No user-facing error. The host silently sends a full snapshot (about 4 KB), the guest adopts it, and a local diagnostic file is written (exportable from Settings → Help). | none |
| **F8** | Different-network false positive (e.g. carrier-grade isolation) | F2 covers this. All routes are offered. | |

> **Why "Continue on one phone" matters:** both phones always hold the full game state, because the simulation is deterministic. A dead battery or a flaky link never kills a match.

### 4.5 What's in the QR

```
sw1:<base32 payload>
payload = proto_version | host IPv4 list (≤3) | TCP port | 128-bit pairing key | host display name | ruleset major
```

The **3-word code** encodes the host's last IPv4 octet, a port slot (one of 16 fixed ports), and a 16-bit session token. The guest assumes its own /24 subnet. That works for home Wi-Fi and every hotspot.

## 5. Playing with a newer player

The game is completely free and every phone has all content, so there is no guest/owner split. Anyone with the app can Link Up with anyone.

- **A session's content comes from the host**: both players can use every critter, Shell, and Shore the host has unlocked, in versus and co-op. A brand-new player can play with a veteran friend's full roster on day one.
- **What the joiner keeps:**
  - Tide Marks earned in co-op, added to their own track.
  - Keepsakes received in trades (see §6).
  - Nothing else from the host's unlocks.

## 6. Trading

### 6.1 What can be traded

- **Keepsakes**, and only Keepsakes: a named critter with its morph, Quirk, Home Shell, and provenance (see [runs-and-progression.md](runs-and-progression.md#54-keepsakes-and-the-aquarium)).
- That's one item type: one UI and one rule set. Shells inside a run aren't tradeable.

### 6.2 Free, fair, face to face

| Promise | How |
|---|---|
| **Free and unlimited** | No tokens, no stamina, no cooldowns, no rarity restrictions. Gifts (one-sided trades) are allowed. |
| **Face to face** | Trading only happens over an active Link Up (LAN, hotspot, or BLE). There's no mail, no codes, and no internet. |
| **Not pay-to-win** | Everything tradeable is earnable solo. Versus defaults to Fair rules (no Keepsakes). |
| **Newcomers included** | Anyone can trade from their first session. A received Keepsake is playable right away, even if its species isn't unlocked on your track yet. |
| **Memories** | Each Keepsake shows its journey: "Bottled by Abhi, 3 Nov · Traded to Riya on the 47 bus" (the location label is optional, typed by the player). |

### 6.3 Trade flow (UI)

1. Lobby → **Trade Keepsakes**. Both see their own Aquarium (a grid) on the bottom and the **Trade Tray** on top. The tray is split: *Yours* | *Theirs*.
2. Drag (or tap) Keepsakes into your side. The other side updates live. Up to 3 per side.
3. Both **Hold "Shake on it"** (1 s). Changing the offer resets both holds.
4. A shell-swap animation plays, and both phones show "Traded ✓" with the new provenance line.

### 6.4 Preventing duplication offline

**What we can and can't promise:**
- With no server, a determined cheater can always edit their save. We don't fight that. Instead, **duplication is made pointless**: there's no economy, no scarcity market, no leaderboards, and every Keepsake is earnable solo.
- What we *do* prevent: **accidental and casual duplication** (force-quitting mid-trade, offering the same Keepsake to two people, getting one back through a loop of trades with backups).

**Mechanisms:**

| Mechanism | Detail |
|---|---|
| **Unique IDs** | Each Keepsake has `id = device_id (128-bit) + counter`, created when it's bottled. |
| **Lock while offered** | A Keepsake in a tray can't be offered in another trade or brought into a run. |
| **Two-phase commit with a journal** | See below. |
| **Tombstones** | A phone remembers every Keepsake ID it has given away and refuses to receive an ID it already holds. Receiving back an ID it gave away is allowed (A→B→A is a legit gift back) **only if** the incoming copy's provenance shows the return hop. |
| **Atomic save writes** | Write to a temp file and rename, so a crash never leaves half a trade on disk. |

**Commit protocol:**

```
1. OFFER (live updates)       both sides            items + their full data
2. ACCEPT(trade_id, H)        both sides            H = hash(sorted offer A + sorted offer B)
3. both ACCEPTs match →       each phone writes journal: PENDING{trade_id, give[], get[]}  (atomic)
                              then sends READY(trade_id)
4. on receiving peer READY →  each phone applies: remove given (tombstone), add received (+provenance hop),
                              journal DONE, sends DONE(trade_id)
```

**Crash recovery:**
- If a phone restarts with a **PENDING** trade and never received the peer's READY, it rolls back (keeps its own items) and shows nothing as lost.
- If it **did** receive READY, it applies the trade.
- **Mismatch window:** one phone applied and the other rolled back. This happens only if a crash lands in the milliseconds between step 3 and step 4. The next time **the same two phones link**, they compare journals for `trade_id` and finish the trade.
- Until then, the outgoing Keepsake shows **"In transit"**. It's visible but can't be used.
- There's a manual **"Cancel stuck trade"** option, and our bias is **never lose an item**. That can create a duplicate in this rare case, and we accept that.

## 7. Netcode sketch for versus fairness

- **Lockstep per round.** The only things sent are **plans** (about 10–20 bytes per round). Each phone simulates everything itself.
- **Commit, then reveal (versus only):** on lock, a phone sends `PLAN_COMMIT(round, sha256(plan ‖ salt))`. Only after it holds the **peer's commit** does it send `PLAN_REVEAL(plan, salt)`. Each phone verifies the reveal against the commit. A modified client can't wait for the opponent's plan before choosing its own.
- **State check:** after each round, each side sends `STATE_HASH(round, sha256(canonical state))`. A mismatch triggers F7.
- **Co-op** skips commit/reveal, since partners see each other's plans. It sends `PLAN_DRAFT` updates (throttled to 4/s) for live ghosts.
- Full message list: [tech.md](tech.md#6-netcode-messages).
