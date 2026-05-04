# War of Castles — Claude Code Context

## What this is
A turn-based medieval territory strategy game built as a single HTML5 Canvas file, 
targeting CrazyGames.com. Inspired by Compact Conflict.

## Tech stack
- Single file: `index.html` (embedded CSS + JS, no frameworks, no build step)
- HTML5 Canvas for map rendering
- Vanilla JavaScript for all game logic
- localStorage for save state (auto-save after every turn)
- CrazyGames SDK (Iteration 8 only)

## Core design rules — never break these
- Map is procedurally generated every new game (20-24 regions)
- Landscape orientation only — mobile users rotate their phone
- Combat is deterministic: attacker wins if units > defenders, ties are random
- Economy: castles produce 1 coin + 1 unit/turn; villages produce 2 coins/turn
- 3 moves per turn (Ninjas get +1 every 3rd turn)
- Same world event cannot repeat two rounds in a row

## Factions
- Merchants: +2 coins/turn
- Warriors: +1 attacking unit in combat
- Stoics: +1 defending unit in combat  
- Ninjas: +1 move every 3rd turn

## Victory conditions
- Conquest: most regions when turns run out
- Economic Win: first to 50 coins (ends game immediately)

## Loss condition
- Eliminated when 0 armies AND 0 castles
- Eliminated players' regions go neutral with 0 units

## Full design reference
See `GDD.md` for complete rules, economy tables, and starting conditions.

## Build status
- [ ] Iteration 1 — Static map & rendering
- [ ] Iteration 2 — Procedural map generation
- [ ] Iteration 3 — Turn & movement system
- [ ] Iteration 4 — Combat & economy
- [ ] Iteration 5 — World events
- [ ] Iteration 6 — AI players
- [ ] Iteration 7 — Factions, victory & loss
- [ ] Iteration 8 — Polish & CrazyGames