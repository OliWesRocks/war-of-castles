# War of Castles — Claude Code Build Plan

## Tech Stack Recommendation
- **Engine:** Plain HTML5 Canvas + JavaScript (no framework) — fastest to iterate, no build step, runs directly on CrazyGames
- **State:** Single JS object, serialized to localStorage after every turn
- **AI:** Rule-based (no ML needed for v1)
- **Maps:** Procedural graph generation (regions as nodes, adjacency as edges)

---

## Iteration 1 — Static Map & Basic Rendering

**Goal:** A visible, clickable map with regions, factions, and unit counts. No gameplay yet.

**Prompt for Claude Code:**
```
Build an HTML5 Canvas game shell for a turn-based strategy game called War of Castles.

Requirements:
- Single HTML file with embedded CSS and JavaScript
- Canvas fills the browser window in landscape orientation
- Hardcode a static test map with exactly 20 regions as an adjacency graph. Each region is a polygon with:
  - A type: "castle", "village", or "empty"
  - An owner: null (neutral) or a faction index (0-3)
  - A unit count (integer)
- Render each region as a filled polygon with:
  - Pale desaturated fill colors for land (parchment, sage, dusty rose, slate tones)
  - Dark hand-drawn-style borders
  - A small icon inside indicating type: tower silhouette for castle, cottage for village, nothing for empty
  - A colored flag/banner showing faction ownership (vivid faction colors: amber, crimson, lavender, emerald)
  - Unit count shown as small stacked circle symbols (up to 5 visible, then show a number)
- Include 2-3 sea regions rendered as light blue with a subtle wave pattern, which are impassable
- Coastal land regions (adjacent to sea) should be visually distinguishable
- Dark sidebar panel on the left showing: Turn number, faction list with region count and coin count, current event card slot (empty for now), moves remaining
- No gameplay logic yet — just rendering
```

---

## Iteration 2 — Procedural Map Generation

**Goal:** Replace the hardcoded map with a procedurally generated one that follows the GDD rules.

**Prompt for Claude Code:**
```
Replace the hardcoded map in War of Castles with a procedural map generator.

The generator must follow these rules:
- Total land regions: 20-24 (random each game)
- Place sea regions as impassable separators along edges
- Land regions adjacent to sea are flagged as "coastal"
- Place 2-4 player castle regions in opposite corners/edges — no two player castles adjacent
- Place 2-3 neutral castle regions near the center
- Place 6-8 village regions distributed across the map, never more than 3 clustered adjacently
- Fill remaining regions as empty land
- Every land region must be reachable from every other land region (connected graph)
- Store the map as a graph: regions[] with polygon vertices, type, owner, units, isCoastal flag, and adjacency list

The generator should run on page load and produce a different map every time.
Keep all rendering from Iteration 1 working with the new dynamic map.
```

---

## Iteration 3 — Core Turn & Movement System

**Goal:** A human player can take a turn — select armies and move them to adjacent regions.

**Prompt for Claude Code:**
```
Add turn-based gameplay to War of Castles. Implement the following:

Turn structure:
- Each turn a player gets 3 moves
- A move = selecting one of your regions with units, then clicking an adjacent region to move all units there
- After moving into a region, that army cannot move again this turn
- Highlight valid move targets when a region is selected (adjacent regions only)
- Show moves remaining in the sidebar
- "End Turn" button ends the player's turn early

Movement rules:
- Moving into an empty unowned region: player takes ownership, units move in
- Moving into an owned region of yours: units merge
- Moving into an enemy or neutral region: trigger combat (placeholder for now — attacker wins automatically)
- An army that just conquered a region cannot move again this turn

State:
- Save full game state to localStorage after every turn
- On page load, check localStorage and offer a "Resume game" button if a saved state exists

Starting conditions:
- Player castle: 3 units
- Neutral castle: 2 units
- Neutral village: 1 unit
- Empty region: 0 units

Support 1 human player (faction 0) and 3 AI players (factions 1-3). AI takes no action yet — just ends its turn immediately.
```

---

## Iteration 4 — Combat & Economy

**Goal:** Real combat resolution, coin income, and army recruitment.

**Prompt for Claude Code:**
```
Add combat resolution and economy to War of Castles.

Combat (deterministic):
- Attacker wins if attacking units > defending units
- Ties are resolved randomly — show UI message "Evenly matched! Fate decides..." before revealing result
- Loser loses all units. Winner keeps their units and takes the region (if attacker wins) or retreats (if defender wins)
- Warriors faction passive: attacking armies count as +1 unit in battle
- Stoics faction passive: defending armies count as +1 unit in battle

Economy (runs at end of each player's turn):
- Each owned castle produces: 1 coin + 1 free army unit
- Each owned village produces: 2 coins
- Village bonus: for every 2 villages owned, one castle produces +1 army unit (apply to castles in order)
- Merchants faction passive: +1 coin per turn

Recruitment (during player's turn, at any owned castle):
- Player can spend coins to recruit extra units at owned castles
- Progressive cost: 1st=2 coins, 2nd=4 coins, 3rd=7 coins, 4th=11 coins, 5th+=16 coins
- Cost resets each turn
- Show recruitment UI when player clicks their own castle: display cost of next unit and current coin balance

Update sidebar to show each faction's coin balance and region count.
```

---

## Iteration 5 — World Events System

**Goal:** One event card drawn per round, affecting the map.

**Prompt for Claude Code:**
```
Add the World Event System to War of Castles.

At the start of each round (before the first player's turn), draw one event card at random:

Events:
1. Flood 🌊 — randomly select one coastal region. Mark it impassable for 2 turns. Units inside cannot move in or out. Flooded villages produce no coins.
2. Wildfire 🔥 — target one unconquered (neutral) region, mark impassable for 1 turn, villages produce no coins. If no neutral regions exist, a random occupied region loses half its units (round down). Cannot stack on a region already on fire.
3. Rich Harvest 🌾 — this turn only, all attacking armies count as +1 extra unit in battle (stacks with Warriors passive)
4. Sunny Days ☀️ — no effect

Rules:
- The same event cannot occur two rounds in a row
- Display the drawn event card prominently at the start of the round with its icon and description — show for 2 seconds then dismiss
- Show active flood/wildfire regions with a visual overlay on the map (blue tint for flood, orange tint for wildfire)
- Show the current active event in the sidebar
```

---

## Iteration 6 — AI Players

**Goal:** Rule-based AI that plays a reasonable game.

**Prompt for Claude Code:**
```
Implement rule-based AI for factions 1-3 in War of Castles.

AI decision priority (evaluate in order each move):
1. If an enemy castle is adjacent and attackable (AI has more units), attack it
2. If a neutral castle is adjacent and attackable, take it
3. If a neutral village is adjacent, take it
4. If an enemy village is adjacent and attackable, take it
5. If an empty neutral region is adjacent, expand into it
6. Otherwise, move units toward the nearest valuable target (castle or village)

AI personality modifiers (set per faction at game start, selectable by human player in setup):
- Nice: never attacks the human player first
- Rude: prioritizes attacking the human player's regions
- Mean: plays optimally, no restrictions
- Evil: plays optimally AND targets the human player's castles specifically

AI recruitment: always spend available coins on units if it can afford the 1st or 2nd unit cost.

AI takes its turn automatically with a 600ms delay between moves so the human can follow the action.
```

---

## Iteration 7 — Factions, Victory & Loss Conditions

**Goal:** Full game loop with selectable factions, win/loss detection, and game over screen.

**Prompt for Claude Code:**
```
Complete the game loop in War of Castles.

Pre-game setup screen (shown before map generation):
- Player selects their faction: Merchants, Warriors, Stoics, or Ninjas
- Player selects victory condition: Conquest (most regions when time runs out) or Economic Win (first to 50 coins)
- Player selects session mode: Quick Skirmish (9 turns), Standard (12 turns), Marathon (15 turns), Endless
- Player selects AI difficulty: Nice, Rude, Mean, Evil
- "Start Game" button generates map and begins

Faction passives (apply automatically throughout game):
- Merchants: +1 coin per turn
- Warriors: +1 attacking unit in combat
- Stoics: +1 defending unit in combat
- Ninjas: +1 move every 3rd turn (turn 3, 6, 9...)

Loss condition:
- A player is eliminated when they have 0 armies AND 0 castles
- A player with a castle but 0 armies survives (castle produces 1 unit next turn)
- Eliminated players' regions go neutral with 0 units

Victory conditions:
- Conquest: when turns run out, player with most regions wins
- Economic Win: first player to reach 50 coins wins immediately, game ends mid-turn

Game over screen:
- Show winner with faction color
- Show final region counts and coin counts for all factions
- "Play Again" button (new map, same settings) and "Change Settings" button
```

---

## Iteration 8 — Polish & CrazyGames Readiness

**Goal:** Production-ready, performant, CrazyGames-compliant build.

**Prompt for Claude Code:**
```
Polish War of Castles for CrazyGames publication.

UI & UX:
- Add animated event card flip when a new event is drawn
- Add smooth unit movement animation between regions (200ms slide)
- Add combat flash animation (red pulse on losing region)
- Improve sidebar: show faction flags, coin icons, region counts clearly
- Add a "?" help button that shows a rules summary overlay
- Add a mute button for any audio (no audio needed yet, just the button)
- Ensure all tap targets are minimum 44x44px for mobile

Performance:
- Render only dirty regions on canvas (not full redraw every frame)
- Cap to 30fps when idle, 60fps during animations

Mobile:
- Detect landscape orientation; if portrait, show "Please rotate your device" overlay
- Pinch-to-zoom on the map canvas
- Double-tap a region to see its details in a popup

localStorage:
- Auto-save after every turn
- On load, if a saved game exists, show "Resume your game?" prompt
- "New Game" clears saved state

CrazyGames SDK:
- Add CrazyGames SDK script tag (https://sdk.crazygames.com/crazygames-sdk-v3.js)
- Call CrazyGames.SDK.init() on load
- Show a banner ad between games (on the game over screen) using CrazyGames ad API
- Do not show ads mid-game

Final check:
- Game must run in a single HTML file with no external dependencies except the CrazyGames SDK
- Test that localStorage save/resume works correctly
- Ensure no console errors on load
```

---

## Summary

| Iteration | What you get | Estimated sessions |
|---|---|---|
| 1 | Visible map, rendering pipeline | 1 |
| 2 | Procedural map generation | 1-2 |
| 3 | Human turn & movement | 1-2 |
| 4 | Combat & economy | 1-2 |
| 5 | World events | 1 |
| 6 | AI players | 2-3 |
| 7 | Full game loop | 1-2 |
| 8 | Polish & CrazyGames | 1-2 |

**Total: ~8 Claude Code sessions to a publishable game.**

After Iteration 3 you have something playable. After Iteration 7 you have a complete game. Iteration 8 is what you submit to CrazyGames.
