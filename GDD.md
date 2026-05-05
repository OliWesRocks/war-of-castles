# War of Castles — Game Design Document

## 1. Overview
War of Castles is a turn-based medieval territory strategy game for 2-4 players (1 human + up to 3 AI). Players conquer regions, build economies through villages, and produce armies from castles. Every game is played on a procedurally generated map. The game targets CrazyGames.com and is built as a single HTML5 Canvas file.

---

## 2. General Rules
- There is a map with 2 to 4 participants, each starting in their own castle
- The standard game lasts 12 turns (configurable — see Session Structure)
- When time runs out, the player meeting the chosen victory condition wins
- Armies conquer and defend regions
- Regions can be empty land, villages, or castles
- Conquered villages produce coins
- Conquered castles produce armies and coins

---

## 3. Regions

### Castles
- Each player starts with one castle under their control
- After each turn, every owned castle produces 1 new army unit
- Players can spend coins to recruit additional units at owned castles
- Neutral castles can be conquered
- Enemy castles can be taken over
- If a castle is lost, the player loses its production

### Villages
- Players start with no villages
- A conquered village produces coins each turn (tax)
- Neutral and enemy villages can be conquered
- Villages do not produce army units

### Empty Regions
- No production
- No starting defenders
- Used as buffers and chokepoints
- Still count toward region total for Conquest victory

---

## 4. Armies

### Movement
- Each player gets 3 moves per turn
- One move = moving one army to a neighbouring region
- An army that has just conquered a region cannot move again this turn
- Moving into an occupied region triggers combat

### Combat Resolution
- **Deterministic**: attacker wins if attacking units > defending units
- **Ties**: resolved randomly — show UI message *"Evenly matched! Fate decides..."* before revealing result
- **If attacker wins**: defender loses all units, attacker takes the region
- **If defender wins**: attacker retreats, loses all units

### Army Production
- Each owned castle produces 1 free army unit per turn by default
- For every 2 villages a player owns, 1 of their castles produces +1 army unit per turn
- The village bonus applies to castles in order (first castle gets boosted first)
- No cap on this bonus — naturally limited by map scarcity of villages
- Villages do not produce armies

---

## 5. Economy

### Income (generated at end of each player's turn)
| Source | Coins per turn |
|---|---|
| Castle | 1 |
| Village | 2 |
| Empty region | 0 |

### Recruitment Costs (resets each turn)
| Unit recruited that turn | Coin cost |
|---|---|
| 1st | 2 |
| 2nd | 4 |
| 3rd | 7 |
| 4th | 11 |
| 5th and beyond | 16 |

- Recruitment happens during the player's turn at any owned castle
- Cost progression resets at the start of each new turn

---

## 6. Starting Conditions

| Location | Starting Army Units |
|---|---|
| Owned castle (player start) | 3 |
| Neutral castle | 2 |
| Neutral village | 1 |
| Empty region | 0 |

- Ownership is persistent: a conquered region stays yours until an enemy reconquers it
- An owned region with 0 units has no defense — any single unit can walk in

---

## 7. Asymmetric Factions
Each faction has one passive trait applied automatically throughout the game.

| Faction | Passive |
|---|---|
| Merchants | +1 coin per turn |
| Warriors | Attacking armies count as +1 extra unit in combat |
| Stoics | Defending armies count as +1 extra unit in combat |
| Ninjas | +1 extra move every 3rd turn (turns 3, 6, 9...) |

---

## 8. World Event System
At the start of each round (before the first player's turn), one event card is drawn at random.

**Rule**: The same event cannot occur two rounds in a row. Otherwise events are random.

### Events

**🌊 Flood**
A coastal region becomes impassable for 2 turns. If occupied, the army cannot move in or out. Flooded villages produce no coins.

**🔥 Wildfire**
One unconquered (neutral) region becomes impassable for 1 turn. Villages under wildfire produce no coins. If all regions are already conquered, a wildfire breaks out in a random occupied region — half the occupying army dies (round down). Cannot stack on a region already on fire.

**🌾 Rich Harvest**
All attacking armies count as +1 extra unit in all battles this turn. Stacks with Warriors faction passive.

**☀️ Sunny Days**
No effect.

### Coastal Regions
Land regions adjacent to sea regions are flagged as coastal and are subject to Flood events.

---

## 9. Victory Conditions
The player chooses one victory condition before the game starts.

| Condition | How to win |
|---|---|
| Conquest | Most regions when turns run out |
| Economic Win | First player to accumulate 50 coins (ends game immediately mid-turn) |

### Tiebreakers (Conquest)
Tied regions → most castles → most coins → random

---

## 10. Loss Conditions
- A player is eliminated when they have **0 armies AND 0 castles**
- A player with a castle but 0 armies survives — their castle produces 1 unit next turn
- Eliminated players' regions go **neutral with 0 defending units**

---

## 11. Session Structure

| Mode | Turns | Estimated time |
|---|---|---|
| Quick Skirmish | 9 | ~12 min |
| Standard | 12 | ~20 min |
| Marathon | 15 | ~30 min |
| Endless | Until eliminated | — |

---

## 12. Procedural Map Generation
Every new game generates a unique map according to these rules:

- Total land regions: 28–32 (random)
- Orientation: horizontal (landscape)
- Number of players: 2–4
- Player castles placed in opposite corners/edges — no two player castles start adjacent
- 2–3 neutral castles placed near the center
- 6–8 villages distributed across the map, never more than 3 clustered adjacently
- Remaining regions filled as empty land
- Sea regions used as impassable separators
- Land regions adjacent to sea are flagged as coastal
- Every land region must be reachable from every other land region (fully connected graph)

---

## 13. AI Players

### Difficulty Levels
| Level | Behaviour |
|---|---|
| Nice | Never attacks the human player first |
| Rude | Prioritises attacking the human player's regions |
| Mean | Plays optimally, no restrictions |
| Evil | Plays optimally, specifically targets human player's castles |

### AI Decision Priority (per move)
1. Attack adjacent enemy castle if attackable (more units)
2. Take adjacent neutral castle if attackable
3. Take adjacent neutral village
4. Attack adjacent enemy village if attackable
5. Expand into adjacent empty neutral region
6. Move units toward nearest valuable target (castle or village)

### AI Recruitment
Always spends available coins on units if it can afford the 1st or 2nd unit cost.

### AI Turn Speed
600ms delay between moves so the human player can follow the action.

---

## 14. CrazyGames-Specific Design

| Challenge | Solution |
|---|---|
| Players quit fast | Auto-save to localStorage after every turn; resume button on load |
| Discovery via thumbnail | Vivid faction flags on pale map; animated event card flip |
| No accounts/login | All progress in localStorage, no server needed |
| Mobile traffic (~40%) | Minimum 44×44px tap targets; pinch-to-zoom on map |
| Monetisation | Cosmetic map skins, faction color packs — no pay-to-win; banner ad on game over screen only |
| Orientation | Landscape only — mobile users rotate their phone; portrait shows rotate prompt |

---

## 15. Art Direction

### Map & Regions
- Pale, desaturated land colors: parchment, sage, dusty rose, slate
- Water rendered as light textured pattern (subtle waves or crosshatch)
- Region borders: clean dark lines, slightly hand-drawn feeling
- No gradients — flat fills only

### Factions
- Each faction identified by a colorful flag/banner icon planted in their regions
- Flag colors are the only saturated, vivid colors on screen
- Faction colors: Amber (orange), Crimson (red), Lavender (purple), Emerald (green)

### Structures
- Castle: small stylized tower silhouette icon, no fine detail
- Village: simple cottage or cluster of rooftops icon
- Both readable at small sizes on mobile

### Units
- Simple symbolic icons, similar to Compact Conflict's dome/ball approach
- Grouped visually to show army size: 1–5 shown as stacked symbols, 6+ shown as number

### UI Panel
- Dark sidebar, clean typography
- Always visible: turn counter, coin balance per faction, region count per faction, current event card, moves remaining
