# Dao Chess — Game Spec (Battlegrounds-style draft)

> **Note:** Converted from the living spec page ([Dao Chess — Game Spec (Battlegrounds-style draft)](https://cursor.com/canvas/store-6da585ab-5050-4fdf-a247-a64824009292/a6643106-02f6-4ee5-86fb-789e7c35ba24)). All numbers are starting drafts for tuning.

What Hearthstone Battlegrounds does today, and a first-pass spec adapting it to a phone-first xianxia auto-battler. Every number is a starting draft for tuning.

## Contents

- [Research: how Battlegrounds works](#research-how-battlegrounds-works)
- [Design pillars and key choices](#design-pillars-and-key-choices)
- [Lobby format: bots first, ghosts next](#lobby-format-bots-first-ghosts-next)
- [Bot opponents (v1)](#bot-opponents-v1)
- [Core loop and match structure](#core-loop-and-match-structure)
- [Economy](#economy)
- [Cultivation realms (tavern tiers)](#cultivation-realms-tavern-tiers)
- [Tribes and rotation](#tribes-and-rotation)
- [Keywords](#keywords)
- [Xianxia keywords](#xianxia-keywords)
- [Talismans (spells)](#talismans-spells)
- [Combat rules](#combat-rules)
- [Sample units](#sample-units)
- [Heroes (patriarchs) and hero powers](#heroes-patriarchs-and-hero-powers)
- [Ascension (triples) and seasonal layer](#ascension-triples-and-seasonal-layer)
- [Progression, ranking, monetization](#progression-ranking-monetization)
- [Art direction: vivid anime xianxia](#art-direction-vivid-anime-xianxia)
- [Mobile UI notes](#mobile-ui-notes)
- [MVP scope vs later](#mvp-scope-vs-later)
- [Open questions](#open-questions)
- [Sources](#sources)

## Research: how Battlegrounds works

| Topic | Battlegrounds (as of patch 37.0, Oct 2026) |
| --- | --- |
| Format | 8 players, each picks a hero; alternate recruit phase and automatic combat vs one opponent; last alive wins, top 4 counts as a win |
| Health | 30 Health for every hero (Patchwerk excepted) plus per-hero Armor, originally 5 to 20; Armor differs for Duos and low ranks |
| Gold | Refills each turn and grows to a 10 Gold start-of-turn max on turn 8; selling can exceed 10 in-turn |
| Actions | Buy 3 Gold, refresh 1 Gold, sell +1 Gold, freeze free |
| Tiers | 6 tiers (Tier 7 only via Secrets of Norgannon anomaly or Thorim). Upgrade base costs 5/7/8/10/10 (Duos 5/7/10/13/12); each costs 1 less per turn you wait |
| Shop size | 3 minions at T1, 4 at T2-3, 5 at T4-5, 6 at T6 |
| Shared pool | Copies per minion: 15/15/13/11/9/7 by tier (5 at T7); sold minions return to the pool |
| Tribes | About 10 minion types; historically 5 per lobby, newer notes say each type has a 50% chance per lobby. Aberrations replaced Naga in Sep 2026 |
| Triples | 3 copies merge into a golden minion (keeps buffs, bigger stats/effect) plus a reward: Discover a minion from the next tier up |
| Combat | Leftmost minion attacks first, sides alternate, random target unless Taunt. Which player starts is not stated in Blizzard posts |
| Damage | Base is the winner's Tavern Tier; community sources add the tiers of surviving minions (not confirmed in Blizzard posts we read). Solo cap: 5, 10 from turn 4, 15 from turn 8, removed at top 4 |
| Matchmaking | Since Aug 2025: face each opponent once before repeats while all 8 live; avoid rematch for 3 rounds after deaths |
| Seasonal layer | Season 14 (Aug 2026): Dark Gifts (3 Gold Discover, up to 3/game), Lesser and Greater Trinkets, new Activate keyword. Earlier: Anomalies, Buddies, Quests, Tavern spells |
| Monetization | Free to play; optional Season Pass gives premium track cosmetics and 2 extra hero choices per game |

Keywords across the mode include Taunt, Divine Shield, Reborn, Windfury, Venomous (formerly Poisonous), Deathrattle, Battlecry, Avenge, Magnetic and the new Activate. Duos (2v2v2v2 shared health) exists with its own costs and caps. MMR details and mobile UX specifics were not verified this pass.

## Design pillars and key choices

- **Short phone matches:** Target 12 to 15 minutes, landscape 16:9 play
- **Five sect tribes (decided):** Righteous Sect, Demonic Sect, Rogue Cultivator, Spirit Beast, Alchemist Association plus Neutral; no elements
- **Realms as tiers:** Six cultivation realms replace tavern tiers
- **Bots at launch (decided):** v1 is single-player vs 7 AI bots, playable offline; async ghost lobbies in v2
- **Fair monetization:** Free to play, $4.99 cosmetic-only Dao Pass, no extra patriarch choices for payers

## Lobby format: bots first, ghosts next

> **Decided.** v1 is played against 7 AI bots, single-player and offline-capable. Async ghost lobbies come in v2; live 8-player lobbies maybe later.

| Version | Opponents | Needs |
| --- | --- | --- |
| v1 (MVP) | 7 AI bots, true shared pool among all 8 seats | No server; local play |
| v2 | Async ghosts: snapshots of real boards at same turn and rating | Light backend to store and serve boards |
| Later, maybe | Live 8-player lobbies | Real-time servers, matchmaking, reconnects |

Bots keep the full Battlegrounds feel in v1: because every seat is simulated locally, the shared unit pool works exactly as in Battlegrounds. Ghost lobbies can fill empty seats with bots later.

## Bot opponents (v1)

| Difficulty | Behavior (draft) |
| --- | --- |
| Apprentice | Buys the highest-tier unit it can, rarely refreshes, advances realm on a fixed schedule, random placement, no triples planning |
| Disciple | Follows a standard realm curve (turn 2/5/7/9 style), sells weakest unit when board full, holds pairs for triples, Iron Body units placed right |
| Elder | Commits to a tribe focus by turn 4, freezes good shops, levels when ahead on health, rolls when behind |
| Patriarch | Elder plus lookahead: simulates a few combats vs likely boards to choose placement and buys; adapts to the run's tribes |
| Heavenly (optional) | Only tier allowed to bend rules, clearly labeled (e.g. +1 Spirit Stone per turn) |

- **Patriarch choice:** Bots are offered 3 patriarchs like the player; lower tiers pick at random, higher tiers pick by a strength rating and personality fit
- **Tribe personalities:** Each bot has a preferred tribe and style: Righteous turtles behind Iron Body, Demonic sacrifices and Devours, Rogue keeps one lone Rogue and Secludes, Spirit Beast goes wide with cubs, Alchemist banks Stones and pills. Strong at Elder+, soft at low tiers; switches if its tribe is out of the run
- **Heuristics:** Scoring per shop unit: tier value + tribe fit + triple progress + keyword need; thresholds and roll/level budgets vary by difficulty. Rogue bots score a second Rogue negatively
- **Fairness:** At normal difficulties bots use the same shop, gold, pool and timers, with no peeking at the player's shop or future rolls
- **Names and flavor:** Bots appear as rival sect elders with names and portraits so lobbies feel alive

Shared tech with v2: the combat sim is deterministic from a seed and lives in one module used by bots now and ghost validation later. v1 records each player's board per turn (locally, uploaded when online with consent) to seed the future ghost pool and to tune bots against real boards.

## Core loop and match structure

1. **Choose a patriarch:** Pick 1 of 3 offered (Battlegrounds offers more to pass owners; we do not)
2. **Market phase (Spirit Pavilion):** Draft 45 to 60 s timer: buy, sell, refresh, seal (freeze), advance realm, use hero power
3. **Duel phase:** Automatic combat vs a bot (v1) or ghost (v2) board, about 15 to 25 s, skippable
4. **Dao Heart damage:** Loser loses Dao Heart; reach 0 and you are out
5. **Repeat:** Until one remains; final placement 1 to 8 drives rating

- **Players:** 8 (you + 7 bots in v1; ghosts in v2)
- **Dao Heart:** 30 plus 0 to 10 Qi Shield (armor) per patriarch
- **Board:** 7 slots
- **Hand:** 10 slots
- **Target length:** 12 to 15 turns

## Economy

| Item | Draft value |
| --- | --- |
| Spirit Stones (gold) | 3 on turn 1, +1 per turn, cap 10 on turn 8 |
| Recruit a unit | 3 |
| Refresh market | 1 |
| Release (sell) | +1 |
| Seal market (freeze) | Free |
| Realm breakthrough | See tier table; -1 per turn you wait |
| Hero power | 0 to 2 typical, once per turn |

## Cultivation realms (tavern tiers)

| Realm | Tier | Advance cost | Shop | Units in MVP |
| --- | --- | --- | --- | --- |
| Qi Condensation | 1 | - | 3 | 8 |
| Foundation Establishment | 2 | 5 | 4 | 8 |
| Core Formation | 3 | 7 | 4 | 8 |
| Nascent Soul | 4 | 8 | 5 | 7 |
| Spirit Severing | 5 | 10 | 5 | 6 |
| Immortal Ascension | 6 | 10 | 6 | 5 |

42 units for MVP: 7 per tribe (5 x 7 = 35) plus 7 Neutral. Neutral by tier 3/1/1/1/1/0; each tribe has 1 at Tier 1, 1 to 2 at Tiers 2 to 4, and 1 each at Tiers 5 and 6. Pool copies per unit: 15/15/13/11/9/7.

## Tribes and rotation

> **Decided.** Elements are dropped entirely: no element tags, no generating or overcoming cycles.

| Tribe | Style | Tools | Signature |
| --- | --- | --- | --- |
| Righteous Sect | Disciplined, defensive | Master/Disciple bonds, Iron Body, Golden Bell, steady scaling | When a Righteous unit dies, sect brothers gain stats |
| Demonic Sect | Aggressive, greedy | Devour, sacrifice, Qi Deviation drawbacks | Grows whenever any friendly unit dies |
| Rogue Cultivator | Lone wolf | Steals enemy stats and keywords, Seclusion, Breakthrough | Big bonus when it is the only Rogue on board |
| Spirit Beast | Numbers and growth | Cub summons, Breakthrough evolution, taming (second tribe tag or cross-tribe buff) | Last Breath cub spawns |
| Alchemist Association | Economy | Pills that buff shop or board units, extra Spirit Stones, refine sold units into pills | Turns Spirit Stones into stats |

> **Decided.** 4 of 5 tribes per run, rotating, plus Neutral; tutorial and Apprentice may use a fixed set.

Why: with 7 units per tribe, all 5 would put 42 units in every shop and make triples and tribe focus rare; dropping one raises hit rates by about a fifth and gives 5 distinct run types, which keeps bot runs feeling different. Rogue wants few Rogues, so it suffers least from rotation.

## Keywords

| Dao Chess | Battlegrounds | Effect |
| --- | --- | --- |
| Iron Body | Taunt | Must be attacked first |
| Golden Bell | Divine Shield | Ignore the first damage |
| Rebirth | Reborn | Returns once with 1 Health |
| Twin Strike | Windfury | Attacks twice |
| Soul Venom | Venomous | Destroys any unit it damages, once |
| Last Breath | Deathrattle | Effect on death |
| Arrival | Battlecry | Effect when played from hand |
| Vengeance (N) | Avenge | Triggers after N allies die |
| Fuse | Magnetic | Merge into an allied construct |
| Channel | Activate | Pay Spirit Stones in market to trigger, once per turn |

## Xianxia keywords

Five Dao Chess-only keywords, used alongside the renamed Battlegrounds set. Tribe fit: Devour and Qi Deviation for Demonic; Breakthrough for Spirit Beast and Rogue; Seclusion for Rogue and Righteous; Tribulation anywhere. All numbers are drafts.

### Breakthrough (X)

- **Rules text:** After this unit survives X combats, it breaks through: permanent +stats and its effect upgrades. One breakthrough per unit unless stated.
- **Counting:** Checked at end of combat: +1 if the unit is alive on the board when combat ends (win, loss or tie). A Rebirth copy that is alive counts. Units in hand or in Seclusion do not count.
- **Golden Bell:** No interaction; shields only help it survive.
- **Rebirth:** Progress is stored on the permanent unit, not the combat copy, so a reborn unit keeps its counter.
- **Selling:** Counter is lost; the unit returns to the pool fresh.
- **Ascension:** Ascended unit takes the highest counter of the three; if any copy had already broken through, the Ascended unit is broken through. Ascended needs X-1 combats and gains double the bonus.
- **UI:** Rising-meridian icon with pips filling (e.g. 2/3); golden flash and short animation on breakthrough.
- **Bot AI:** Value scales with turns remaining and expected survival; Elder+ place Breakthrough units behind Iron Body and avoid selling near completion.
- **Balance:** Weak base stats, strong payoff; X of 2 to 4. Watch early-buy snowballing.

### Tribulation

- **Rules text:** Start of combat: heavenly lightning deals N damage to this unit. If it survives, it has double Attack and Health for this combat.
- **Ordering:** Resolves first among start-of-combat effects, left to right, before other start-of-combat buffs; doubling applies to stats after lightning (current Attack and Health x2).
- **Golden Bell:** Lightning pops Golden Bell: the unit survives undamaged and doubles, but enters combat without its shield. Decided: the shield absorbs the lightning.
- **Rebirth:** If lightning kills it, Last Breath and Rebirth trigger normally; the reborn copy does not face Tribulation again.
- **Ascension:** Ascended: lightning deals the same N, and the survival bonus is triple instead of double.
- **Death counts:** Death from lightning counts toward Vengeance for both sides? Draft: counts for its owner only.
- **UI:** Storm-cloud icon showing N; preview of survive or fail on the board in market phase.
- **Bot AI:** Buy and keep when Health is above N or it has Golden Bell (which absorbs the lightning); Elder+ buff Health or give Golden Bell in market to clear N, knowing the shield is spent.
- **Balance:** N fixed per unit (not scaling) so Health buffs are the skill test.

### Devour

- **Rules text:** Arrival: destroy the friendly unit to the right of this (else left). Gain its Attack, Health and keywords.
- **Empty neighbor:** No neighbor: nothing happens; the Devour unit is played normally.
- **What transfers:** Current stats incl. permanent buffs, keywords (Golden Bell, Iron Body, Rebirth...) and counters (Breakthrough progress). Not the eaten unit's Arrival text, tribe tag or Ascended status.
- **Death triggers:** A devoured unit is consumed, not killed: no Last Breath, does not count for Vengeance, and returns to the pool as one plain copy (Ascended returns three).
- **Ascension:** An Ascended Devour unit eats both neighbors. Stats gained from Devour persist through ascension like other buffs.
- **UI:** Fanged-maw icon; when dragging to play, highlight the target neighbor in red with a confirm step.
- **Bot AI:** Play next to the lowest-value unit with high stats or keywords; never eat a pair needed for a triple (Disciple+).
- **Balance:** Loses a board slot and gold; risk of stacking many keywords on one unit (cap duplicates).

### Seclusion

- **Rules text:** Channel (free, once per turn in market, from turn 4 on): this unit enters Seclusion. It skips the next combat, then returns with its seclusion bonus permanently.
- **Slot:** The unit stays on the board shown as meditating and keeps its slot; it is removed from the combat copy, so the other units close the gap in combat. It does not free a slot for buying.
- **Combat:** Cannot be attacked, targeted or buffed by combat effects, and does not count as on board for tribe-count effects (so a secluded Rogue does not break the lone-Rogue bonus); if all other units die you lose that combat.
- **Selling:** Can be sold while secluded; bonus is lost.
- **Ascension:** Tripling a secluded unit ends Seclusion with no bonus (draft), or blocks ascending until it returns.
- **Interactions:** Does not count as surviving for Breakthrough; cannot be Devoured while secluded.
- **UI:** Lotus icon; secluded unit greyed with a turn timer; damage preview warns if board is weak.
- **Bot AI:** Never before turn 4; seclude when well ahead on health or when facing a likely loss anyway; never in the final 4 at Elder+.
- **Balance:** Bonus about +50% stats or a new keyword; tempo loss is the cost. Locked before turn 4 so it cannot exploit the 5-damage cap turns; still watch the 10-cap turns 4 to 7.

### Qi Deviation

- **Rules text:** Qi Deviation (N): this unit's power comes with a drawback that triggers when its condition is met, e.g. at start of combat take N damage, or at end of market destroy your lowest-Attack ally. Printed per unit.
- **Ordering:** Start-of-combat deviations resolve right after Tribulation, left to right. Market deviations resolve at end of market, before the board snapshot.
- **Golden Bell:** Self-damage pops Golden Bell like any damage.
- **Deaths:** Units destroyed by a deviation count as friendly deaths, so they feed Demonic growth, Vengeance and Righteous sect brothers.
- **Ascension:** Ascended keeps the same drawback with doubled upside (the drawback does not double).
- **UI:** Cracked-meridian icon in red; drawback text on the card face, preview of who dies when you end market.
- **Bot AI:** Demonic bots accept drawbacks when the board has fodder; others avoid deviation units unless the upside is over a threshold.
- **Balance:** Upside about 1.5x a normal unit of the tier; drawback must hurt non-Demonic boards.

### Sample units for new keywords

| Unit | Tribe | Tier | Atk/HP | Effect (draft) |
| --- | --- | --- | --- | --- |
| Spirit Fox Kit | Spirit Beast | 1 | 1/3 | Breakthrough (2): +2/+2 and Last Breath: summon a 1/1 Cub |
| Wandering Sword Ronin | Rogue | 4 | 3/4 | Breakthrough (3): +4/+4 and Twin Strike |
| Thunder-Tempered Disciple | Righteous | 2 | 3/4 | Tribulation (3) |
| Storm Pill Refiner | Alchemist | 5 | 5/7 | Tribulation (5); if it survives, also 3 damage to all enemies |
| Blood Lotus Devourer | Demonic | 3 | 2/2 | Devour |
| Nine-Coiled Demon Serpent | Demonic | 5 | 4/4 | Devour; then give adjacent Demonic allies +2/+2 |
| Cave Hermit | Rogue | 2 | 2/3 | Seclusion: +3/+3 |
| Closed-Door Elder | Righteous | 4 | 4/6 | Seclusion: +5/+5 and Golden Bell |
| Heart-Devil Acolyte | Demonic | 2 | 5/4 | Qi Deviation: end of market, destroy your lowest-Attack ally |

### Candidate keywords for later

- **Inner Disciple / Master:** Master buffs linked disciples; bond pairs
- **Karmic Debt:** Power now, penalty paid later turns
- **Soul Lamp:** Lamp tracks a unit; when it dies, effect elsewhere or revive

> **Rules calls decided.** Golden Bell absorbs Tribulation lightning (unit doubles, fights unshielded). Seclusion unlocks on turn 4. Dao Heart stays player health only; triple reward is Ascension Gift.

## Talismans (spells)

> **Decided.** Talisman spells are in the MVP Spirit Pavilion, modeled on Battlegrounds Tavern spells, which appear in the tavern beside minions, are gated by tier and have varying Gold costs (average about 1.3 at Tier 1 rising to about 3.1 at Tier 6, per bgknowhow).

- **Shop:** 1 talisman slot to the right of the unit row on every roll (draft); refreshed and sealed with the market
- **Cost:** 0 to 4 Spirit Stones printed per talisman; most early ones 1, late ones 3 to 4
- **Realm gating:** Only talismans of your realm or lower appear
- **Pool:** Talismans have no copy limits in v1 (draft); not shared with bots' units
- **Hand:** Bought talismans go to hand and share the 10-slot hand with units
- **Casting:** Market phase only, drag onto a unit or the board; never during combat. Combat-timed ones say 'next combat' and are applied to the board snapshot
- **Triples:** Talismans do not triple or ascend
- **Pills vs talismans:** Distinct: pills are produced by Alchemist units, patriarchs or refining sold units, are free and permanent stat buffs; talismans are bought from the shop and can do anything (stats, keywords, economy, discover). Effects that say 'talisman' never affect pills and vice versa

| Talisman | Tier | Cost | Effect (draft) |
| --- | --- | --- | --- |
| Qi Gathering Charm | 1 | 1 | Give a unit +1/+2 |
| Spirit Stone Pouch | 1 | 0 | Gain 1 Spirit Stone next turn |
| Fortune Divination | 1 | 1 | Refresh the market and seal one unit |
| Iron Skin Seal | 2 | 1 | Give a unit Iron Body |
| Golden Bell Ward | 2 | 2 | Give a unit Golden Bell |
| Heart-Devil Curse | 2 | 1 | Destroy a friendly unit; give its stats to your Demonic units split evenly (Demonic) |
| Beast Taming Bell | 3 | 2 | Give a Spirit Beast a second tribe tag of your choice (Spirit Beast) |
| Sect Oath | 3 | 2 | Your Righteous units gain +1/+1; double if 4 or more (Righteous) |
| Wandering Path | 3 | 2 | Your only Rogue gains +3/+3 and Rebirth (Rogue) |
| Furnace Fire Talisman | 4 | 2 | Your next 3 pills give +1/+1 extra (Alchemist) |
| Thunder Calling Seal | 4 | 3 | Next combat: start of combat, 3 damage to 2 random enemies |
| Realm Insight | 5 | 3 | Discover a unit from one realm higher |
| Body Refining Scripture | 5 | 3 | Give a unit +4/+4 and Breakthrough (2): +4/+4 |
| Heavenly Decree | 6 | 4 | Give all your units +3/+3 |

- **Bot buying:** Score talisman like a unit: value per Stone vs best unit in shop; tribe talismans only if focus matches; Apprentice buys rarely and casts on a random unit
- **Bot casting:** Disciple+: stats on the unit most likely to survive; keywords to fill gaps (Iron Body if none, Golden Bell on Tribulation units); economy talismans early, Decree late
- **Mobile UI:** Talisman slot drawn as a yellow paper charm with vermilion script; drag to target with valid targets highlighted; long-press for text; cost on a stone token

## Combat rules

1. Side with more units attacks first; tie is a coin flip (our rule; Blizzard does not document theirs)
2. Attackers go left to right, sides alternate; target random among Iron Body units, else random
3. Start of combat: Tribulation, then Qi Deviation, then other effects, each left to right
4. Simultaneous damage; deaths resolve left to right, Last Breath then Rebirth
5. Max 7 units; summons beyond 7 fizzle
6. Damage to loser: winner's realm tier + tiers of surviving units
7. Damage cap: 5 until turn 4, 10 until turn 8, 15 after, removed at final 4
8. Fully deterministic from a seed so ghost fights replay identically

## Sample units

| Unit | Tribe | Tier | Atk/HP | Effect |
| --- | --- | --- | --- | --- |
| Outer Sect Disciple | Righteous | 1 | 2/2 | Golden Bell |
| Jade Gate Guardian | Righteous | 2 | 2/5 | Iron Body; when a Righteous ally dies, gain +1/+1 |
| Sect Master Bai | Righteous | 5 | 5/7 | Master: your Disciples have +2/+2; when one dies, other Righteous gain +2/+2 |
| Blood Cult Initiate | Demonic | 1 | 2/1 | Whenever a friendly unit dies, gain +1 Attack |
| Soul-Eating Patriarch | Demonic | 6 | 6/6 | Whenever a friendly unit dies, gain +2/+2 permanently |
| Lone Blade Wanderer | Rogue | 1 | 3/2 | If it is your only Rogue, +2/+2 in combat |
| Thief of Heaven | Rogue | 3 | 3/3 | Start of combat: steal 2 Attack and a keyword from the enemy opposite |
| Wolf Mother | Spirit Beast | 2 | 3/3 | Last Breath: summon two 1/1 Wolf Cubs |
| Tamed Thunder Ape | Spirit Beast | 4 | 5/5 | Tamed: also counts as Righteous; Breakthrough (2): +3/+3 |
| Pill Furnace Apprentice | Alchemist | 1 | 1/3 | End of market: give a random unit in the shop +1/+1 |
| Pill Refiner Elder | Alchemist | 4 | 3/5 | When you sell a unit, refine it into a pill: +2/+2 to a board unit |
| Golden Core Treasurer | Alchemist | 6 | 5/8 | Start of market: gain 1 Spirit Stone; unspent Stones become +1/+1 each on your leftmost unit |
| Heavenly Dao Arbiter | Neutral | 6 | 8/8 | Start of combat: +2/+2 to all allies for each different tribe on board |

## Heroes (patriarchs) and hero powers

> **Decided.** Battlegrounds-style heroes: each patriarch has exactly one hero power, either passive or active with a Spirit Stone cost, usable once per turn in the shop phase unless stated.

| Hero | Shield | Hero power |
| --- | --- | --- |
| Sword Saint Lan (Righteous) | 5 | 1 Stone: give a unit Golden Bell for next combat |
| Abbot Huiming (Righteous) | 10 | Passive: first Righteous unit bought each turn gains Iron Body |
| Blood Demon Ancestor (Demonic) | 6 | 1 Stone: destroy a friendly unit; give its stats to another |
| Lone Swordsman Ye (Rogue) | 7 | Passive: if you have exactly one Rogue, it gains +1/+1 each end of market |
| Beast Tamer Ling (Spirit Beast) | 8 | 2 Stones: summon a 1/1 Cub that copies a tribe tag of an adjacent unit |
| Pill King Yan (Alchemist) | 8 | 2 Stones: craft a pill giving +2/+2 to a shop or board unit |
| Wandering Immortal (Neutral) | 7 | 2 Stones: Discover a unit of your current realm |

MVP: 16 patriarchs, 3 per tribe plus 1 Neutral; 3 offered per run, only from tribes in that run (or Neutral).

## Ascension (triples) and seasonal layer

Three copies merge into an Ascended unit: double base stats, stronger effect, buffs kept, plus an Ascension Gift to Discover a unit one realm higher. Later seasons can rotate a layer like Battlegrounds' anomalies or buddies: Heavenly Omens (lobby-wide rule), Sect Disciples (buddies). Decided: no Bazaar-style treasures or Spirit Array, in any season.

## Progression, ranking, monetization

- **Rating (v1 local ladder):** Offline, so rank is a local ladder: climb by placing top 4, unlock higher bot difficulties as ranks. Online MMR rank arrives with v2 ghosts
- **Offline implications:** No server in v1; pass purchases and cosmetics need Google Play billing and a light account sync; local saves can be edited, so local rank is not competitive
- **Free track:** Seasonal XP from placements unlocks patriarchs and cosmetics
- **Dao Pass $4.99:** Cosmetic only: board skins, unit art, emotes, combat effects
- **Deliberate difference:** Battlegrounds' pass grants 2 extra hero choices; Dao Chess will not, to stay non-pay-to-win

## Art direction: vivid anime xianxia

> **Decided.** Vivid, high-detail anime xianxia in a donghua and gacha card-game look: saturated colors, glowing VFX, ornate gold filigree frames. Ink-wash was rejected.

- **Units:** Attractive, high-detail anime character and beast art on portrait cards; ornate gold filigree frames, tier shown by frame richness; Ascended units get an animated golden glow frame
- **Patriarchs:** Full splash-art portraits with flowing robes and auras; idle animation (hair, robes, qi particles)
- **Board:** Chinese palace courtyard backdrop with red plum blossoms, clouds and golden roofs; slightly blurred and darkened behind cards so stats stay readable; scenery rises with realm (sect courtyard to heavenly palace)
- **Tribulation VFX:** Purple-gold storm cloud, glowing lightning bolt strikes the card; Golden Bell shatters in a burst of gold shards; on survival a blazing aura and stat numbers pulse x2
- **Devour VFX:** Crimson demonic flames and a spectral maw pull the neighbor in as glowing particles
- **Seclusion VFX:** Unit sits inside a glowing jade lotus barrier with a turn counter; returns with a radiant burst
- **Breakthrough VFX:** Pillar of golden light, rising qi rings and a short realm-glyph flash
- **Tribe accents:** Righteous azure, Demonic crimson, Rogue earth brown, Spirit Beast jade green, Alchemist amber; Neutral pure ink
- **Phone readability:** Busy art must never cover stats: Attack and Health in solid high-contrast badges; keyword icons as clean glyphs; tribe accents always paired with a tribe crest shape for color-blind play; minimum 12 sp text; VFX intensity setting for low-end phones
- **Dao Pass cosmetics:** Alternate character skins and outfits, animated card frames (jade, crimson, celestial), board backdrops (night palace, sakura-plum garden, floating isles), patriarch splash variants, colored lightning and fire VFX, emotes; all cosmetic. Higher art cost per item than a minimal style, so plan a small art pipeline

## Mobile UI notes

- **Landscape 16:9 layout:** Spirit Pavilion shop row at top with refresh, seal and realm-advance buttons; your 7-slot board in the middle; hand tray along the bottom; hero portrait with Dao Heart at bottom-left beside the tray
- **Spirit Stones counter:** Prominent counter next to the hero portrait showing current/max (e.g. 7/10) with a Spirit Stone icon; animates on spend and gain
- **Hero power button:** Round button next to the hero portrait with the power's icon and its Spirit Stone cost; greyed when used this turn or unaffordable; passive powers show a no-cost ring; long-press for text
- **Hand area:** A simple row/tray of cards, no illustrated hand: bottom strip holds up to 10 cards, units and talismans, bought but not yet played; drag a unit from hand to a board slot to play it (Arrival triggers), drag a talisman onto a target to cast; full hand blocks buying with a clear message
- **Attack and Health always visible:** Every creature card in the shop, in hand and on the board shows its Attack (bottom-left sword badge) and Health (bottom-right heart badge); buffed values in green, damaged in red; badges stay on top of all VFX
- **Selling:** Drag a board unit onto the sell zone, or back onto the Spirit Pavilion row, to sell for 1 Spirit Stone. While dragging, the shop row tints red and shows a 'Release +1' label with a Spirit Stone icon; dropping elsewhere cancels. Tap-select then Sell button as fallback
- **Drag and tap:** Drag to buy, place or sell; tap-to-select fallback for accessibility
- **Long-press:** Card details and keyword glossary
- **Tribe badges:** Each card shows a tribe crest (sword, skull, straw hat, paw, cauldron); tamed beasts show two. Run tribes shown at start and in a corner chip
- **Keyword icons:** Breakthrough pips, Tribulation storm cloud with N, Devour maw, Seclusion lotus timer, Qi Deviation red crack
- **Tribe counters:** Demonic death-growth and Righteous brother buffs show a small counter; lone-Rogue bonus lights up when active
- **Combat:** 2x speed and skip; replay from the result screen
- **Resume:** Async runs survive app close; market timer pauses offline
- **Low-end Android:** 2D anime art with downloadable high-res packs, VFX quality setting; base download under 300 MB target

## MVP scope vs later

| MVP | Later |
| --- | --- |
| v1: 7 AI bots, offline, local ladder | v2: async ghost lobbies and online rank |
| Board recording for future ghosts | Later, maybe: live 8-player lobbies |
| 5 tribes, 4 per run, 42 units | More units, more tribes, rotating pools |
| 16 patriarchs | Duos (sect partners) |
| Ascension triples, 14 talismans | Omens, more talismans |
| Ranked rating, free track | Dao Pass, friends, spectate |

## Open questions

1. Decided: v1 vs 7 AI bots offline, v2 async ghosts, live 8 maybe later.
2. Decided: five sect tribes, no elements.
3. Decided: 4 of 5 tribes rotate per run plus Neutral; fixed set allowed for tutorial and Apprentice.
4. Decided: Golden Bell absorbs Tribulation lightning.
5. Decided: Seclusion unlocks on turn 4.
6. Decided: no Bazaar treasures or Spirit Array, not even seasonal.
7. Decided: element positioning question dropped with elements.
8. Rules detail: does a unit killed by its own Tribulation count toward Vengeance and Demonic growth (draft: yes, for its owner)?
9. Rules detail: tripling a secluded unit ends Seclusion with no bonus, or is blocked until it returns?
10. Market timer length, and is a 12 to 15 minute match right?
11. Decided: talisman spells are in the MVP shop.
12. Decided: heroes (patriarchs) with one hero power each.
13. Decided: vivid anime xianxia (donghua / gacha card look), landscape 16:9.
14. Talismans: confirm one dedicated talisman slot per shop roll, and whether Alchemist pills count as talismans.
15. Launch regions and languages (Chinese localization?)

> All numbers on this page are starting drafts for tuning.

## Sources

- [How Battlegrounds Works (patch 37.0 check, cites Blizzard posts)](https://senseos.gg/guides/hearthstone/how-battlegrounds-works/) — senseos.gg, 26 Oct 2026
- [Battlegrounds Know-How: tiers, pool, shop size](http://sat.bgknowhow.com/bgstrategy/general.php) — bgknowhow.com
- [Tavern Tier](https://hearthstone.wiki.gg/wiki/Battlegrounds/Tavern_Tier) — hearthstone.wiki.gg
- [Secrets of Norgannon and Tier 7](https://www.strafe.com/news/read/exploring-the-limitations-and-potential-of-secrets-of-norgannon-in-hearthstone-battlegrounds/) — strafe.com
- [Battlegrounds 2026 tier list (types, HP range)](https://www.playnews.gg/en/guides/hearthstone-battlegrounds-2026-post-patch-35-4-tier-list) — playnews.gg
