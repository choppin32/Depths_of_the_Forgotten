# DEPTHS OF THE FORGOTTEN
## Technical Requirements Document
### Version 1.0 | April 2026

---

## TABLE OF CONTENTS

1. [Project Overview](#1-project-overview)
2. [Technical Architecture](#2-technical-architecture)
3. [Core Game Constants](#3-core-game-constants)
4. [Character Classes](#4-character-classes)
5. [Enemy System](#5-enemy-system)
6. [Boss System](#6-boss-system)
7. [Item & Loot System](#7-item--loot-system)
8. [Equipment System](#8-equipment-system)
9. [Shop System](#9-shop-system)
10. [Dungeon Generation](#10-dungeon-generation)
11. [Visibility & Fog of War](#11-visibility--fog-of-war)
12. [Combat System](#12-combat-system)
13. [Progression & Economy](#13-progression--economy)
14. [Death & Respawn System](#14-death--respawn-system)
15. [Save/Load System](#15-saveload-system)
16. [Audio System](#16-audio-system)
17. [User Interface](#17-user-interface)
18. [Input & Controls](#18-input--controls)
19. [Responsive Design & Mobile Support](#19-responsive-design--mobile-support)
20. [Deployment & Hosting](#20-deployment--hosting)

---

## 1. PROJECT OVERVIEW

### 1.1 Description
"Depths of the Forgotten" is a text-based roguelike dungeon crawler featuring a hybrid display system combining an ASCII grid map with narrative text descriptions. The player descends 40 procedurally-generated dungeon floors, battling enemies and 8 bosses, collecting loot, and managing equipment across 7 gear slots. The game features 5-life permadeath (not instant), 6 playable character classes, a tiered shop system, and turn-based combat with class-specific abilities.

### 1.2 Target Platforms
| Platform | Implementation | Notes |
|----------|---------------|-------|
| Desktop Browser | Single-file HTML/CSS/JS | Primary platform |
| Mobile Browser | Same file, responsive CSS | Touch controls, landscape support |
| Python Terminal | Separate `game.py` file | ANSI color codes, JSON save |

### 1.3 Hosting
- **Netlify Drop** for browser version deployment
- Single `index.html` file (zero dependencies, no build step)

---

## 2. TECHNICAL ARCHITECTURE

### 2.1 File Structure
```
/Ccode/
  index.html     ~1,845 lines  (HTML + CSS + JavaScript, single file)
  game.py        Python terminal version (legacy, not actively maintained)
```

### 2.2 Technology Stack
| Component | Technology |
|-----------|-----------|
| Rendering | DOM manipulation, innerHTML |
| Styling | Embedded CSS with media queries |
| Game Logic | Vanilla JavaScript (ES6 classes) |
| Audio | Web Audio API (OscillatorNode, square wave) |
| Persistence | localStorage (browser) / JSON file (Python) |
| Map Display | Monospace `<pre>`-style with `<span>` color classes |

### 2.3 Core Classes & Objects
| Class/Object | Purpose |
|-------------|---------|
| `Player` | Player state, stats, equipment, abilities |
| `Room` | Room geometry, contents (enemies, items, chests, hidden items) |
| `GameState` | Master game state (grid, rooms, player, revealed tiles, victory flag) |

### 2.4 Utility Functions
| Function | Signature | Description |
|----------|-----------|-------------|
| `randInt` | `(a, b) -> int` | Random integer in [a, b] inclusive |
| `pick` | `(arr) -> element` | Random element from array |
| `cloneItem` | `(item) -> item` | Deep clone via JSON serialize/deserialize |

---

## 3. CORE GAME CONSTANTS

| Constant | Value | Description |
|----------|-------|-------------|
| `MAP_W` | 50 | Dungeon grid width (columns) |
| `MAP_H` | 25 | Dungeon grid height (rows) |
| `MAX_ROOMS` | 10 | Maximum rooms generated per floor |
| `MAX_FLOOR` | 40 | Total dungeon depth (victory condition) |
| `STARTING_LIVES` | 5 | Lives before permanent death |
| `soundEnabled` | `true` | Global sound toggle (mutable) |

---

## 4. CHARACTER CLASSES

### 4.1 Overview
Six playable classes with distinct stat spreads, abilities, starting items, and exploration modifiers.

### 4.2 Class Definitions

#### Knight
| Stat | Value |
|------|-------|
| HP | 40 |
| ATK | 5 |
| DEF | 3 |
| Ability | **Shield Block** — Halves incoming damage for 2 turns |
| Ability Uses | Unlimited |
| Starting Item | Steel Shield (armor, +1 DEF) |
| Detect Hidden | 10% |
| Disarm Trap | 20% |
| Description | Tanky defender. Beginner-friendly. |

#### Rogue
| Stat | Value |
|------|-------|
| HP | 25 |
| ATK | 7 |
| DEF | 1 |
| Ability | **Backstab** — 2x damage on first attack in combat |
| Ability Uses | Unlimited |
| Starting Item | Lockpick Set (key, +1 key) |
| Detect Hidden | 25% |
| Disarm Trap | 60% |
| Description | High risk, high reward. Finds secrets. |

#### Mage
| Stat | Value |
|------|-------|
| HP | 20 |
| ATK | 4 |
| DEF | 1 |
| Ability | **Fireball** — Deals 15 + (level x 2) damage |
| Ability Uses | 3 per floor |
| Starting Item | Scroll of Fire (scroll, 20 damage) |
| Detect Hidden | 15% |
| Disarm Trap | 10% |
| Description | Glass cannon. Devastating magic. |

#### Ranger
| Stat | Value |
|------|-------|
| HP | 28 |
| ATK | 6 |
| DEF | 2 |
| Ability | **Scout** — Reveals all hidden items in current room |
| Ability Uses | Unlimited |
| Starting Item | Hunter's Bow (weapon, +1 ATK) |
| Detect Hidden | 50% |
| Disarm Trap | 35% |
| Description | Balanced. Best at finding secrets. |

#### Cleric
| Stat | Value |
|------|-------|
| HP | 30 |
| ATK | 4 |
| DEF | 2 |
| Ability | **Divine Heal** — Restores 15 + (level x 2) HP |
| Ability Uses | 3 per floor |
| Starting Item | Holy Symbol (buff_defense, +1 DEF permanent) |
| Detect Hidden | 15% |
| Disarm Trap | 20% |
| Description | Sustain healer for long runs. |

#### Berserker
| Stat | Value |
|------|-------|
| HP | 35 |
| ATK | 8 |
| DEF | 0 |
| Ability | **Rage** — +5 ATK for 3 turns, +50% incoming damage |
| Ability Uses | Unlimited (cannot stack) |
| Starting Item | War Paint (buff_attack, +1 ATK permanent) |
| Detect Hidden | 5% |
| Disarm Trap | 10% |
| Description | All offense, no defense. |

### 4.3 Starting Item Handling Logic
| Item Type | Action |
|-----------|--------|
| `key` | Added to player key count |
| `weapon` | Auto-equipped to weapon slot |
| `armor` | Auto-equipped to chest slot |
| `buff_attack` | Added to bonusAttack, recalcStats called |
| `buff_defense` | Added to bonusDefense, recalcStats called |
| Other | Added to inventory |

---

## 5. ENEMY SYSTEM

### 5.1 Enemy Roster (20 Types)

| # | Name | HP | ATK | XP | Map Char | Floor Range |
|---|------|-----|-----|-----|----------|-------------|
| 0 | Rat Swarm | 5 | 3 | 3 | `r` | 1+ |
| 1 | Goblin | 8 | 4 | 5 | `g` | 1+ |
| 2 | Cave Spider | 7 | 5 | 4 | `s` | 1+ |
| 3 | Skeleton | 12 | 5 | 8 | `S` | 2+ |
| 4 | Zombie | 16 | 4 | 7 | `Z` | 3+ |
| 5 | Bandit | 10 | 6 | 9 | `b` | 3+ |
| 6 | Dark Wizard | 14 | 8 | 15 | `W` | 5+ |
| 7 | Troll | 28 | 7 | 20 | `T` | 7+ |
| 8 | Wraith | 18 | 9 | 18 | `H` | 9+ |
| 9 | Orc Warlord | 32 | 10 | 25 | `O` | 11+ |
| 10 | Demon Imp | 20 | 11 | 22 | `D` | 13+ |
| 11 | Mimic | 24 | 9 | 20 | `M` | 15+ |
| 12 | Gargoyle | 30 | 8 | 18 | `G` | 17+ |
| 13 | Vampire Bat | 14 | 7 | 12 | `v` | 19+ |
| 14 | Fire Elemental | 22 | 12 | 28 | `F` | 21+ |
| 15 | Ice Golem | 38 | 10 | 30 | `I` | 23+ |
| 16 | Shadow Assassin | 16 | 14 | 32 | `A` | 25+ |
| 17 | Necromancer | 20 | 12 | 35 | `N` | 27+ |
| 18 | Death Knight | 42 | 13 | 40 | `K` | 29+ |
| 19 | Elder Demon | 48 | 15 | 50 | `E` | 31+ |

### 5.2 Enemy Scaling
```
scalingFactor = 1 + (floorNumber * 0.12)
scaledHP     = floor(baseHP * scalingFactor)
scaledATK    = floor(baseATK * scalingFactor)
scaledXP     = floor(baseXP * scalingFactor)
```

### 5.3 Enemy Pool Selection
```
maxEnemyIndex = min(ENEMY_TYPES.length - 1, 2 + floor(floorNumber / 2))
pool = ENEMY_TYPES.slice(0, maxEnemyIndex + 1)
```

### 5.4 Enemy Spawn Count Per Room
```
count = randInt(0, 1 + floor(floorNumber / 3))
```
Enemies only spawn in rooms that are not: the starting room, a shop room, or a boss room.

---

## 6. BOSS SYSTEM

### 6.1 Boss Schedule
Bosses appear every 5 floors starting at floor 5 (floors 5, 10, 15, 20, 25, 30, 35, 40).

### 6.2 Boss Scaling
```
bossScaling = 1 + (floorNumber - 5) * 0.05
```

### 6.3 Boss Definitions

| Floor | Name | HP | ATK | XP | Phase 2 HP Trigger | Phase 2 ATK Bonus | Drop |
|-------|------|-----|-----|------|---------------------|-------------------|------|
| 5 | The Rat King | 60 | 6 | 50 | <= 30 | +3 | Rat King's Crown (head, +3 DEF) |
| 10 | The Bone Colossus | 100 | 8 | 100 | <= 50 | +4 | Colossus Femur (weapon, +5 ATK) |
| 15 | The Plague Witch | 80 | 10 | 150 | <= 40 | +5 | Witch's Grimoire (scroll, 30 dmg) |
| 20 | The Iron Golem | 150 | 12 | 200 | <= 75 | +6 | Golem Core (amulet, +25 HP) |
| 25 | The Shadow Dragon | 180 | 15 | 300 | <= 90 | +7 | Dragon Scale Mail (chest, +5 DEF) |
| 30 | The Lich Lord | 200 | 18 | 400 | <= 100 | +8 | Lich's Phylactery (ring, +6 ATK) |
| 35 | The Abyssal Hydra | 250 | 20 | 500 | <= 125 | +10 | Hydra Fang Blade (weapon, +8 ATK) |
| 40 | The Forgotten King | 300 | 25 | 1000 | <= 150 | +12 | Crown of the Forgotten (head, +10 DEF) |

### 6.4 Phase 2 Mechanics
- Phase 2 triggers when boss HP drops to or below the threshold.
- One-time trigger: `phase2Triggered` flag prevents re-triggering.
- Boss ATK permanently increases by the phase 2 bonus for the rest of the fight.
- A narrative message is displayed in the combat log.

### 6.5 Boss Floor Rules
- The boss spawns in the **last generated room** (designated boss room).
- Boss room description is overridden: *"The air crackles with dark energy..."*
- **No stairs** appear initially on boss floors.
- Stairs (`>`) are placed at the boss's position upon defeat.
- **Cannot flee** from boss fights (Run button hidden).

---

## 7. ITEM & LOOT SYSTEM

### 7.1 Item Types

| Type Key | Effect | Auto-Use | Equippable |
|----------|--------|----------|------------|
| `heal` | Restores HP | No | No |
| `weapon` | Equips to weapon slot (+ATK) | Yes | Yes |
| `armor` | Equips to gear slot (+DEF) | Yes | Yes |
| `ring_atk` | Equips to ring slot (+ATK) | Yes | Yes |
| `ring_def` | Equips to ring slot (+DEF) | Yes | Yes |
| `amulet_hp` | Equips to amulet slot (+MaxHP) | Yes | Yes |
| `buff_attack` | Permanent ATK bonus | Yes | No |
| `buff_defense` | Permanent DEF bonus | Yes | No |
| `buff_hp` | Permanent Max HP bonus | Yes | No |
| `scroll` | Deals damage to current combat target | No | No |
| `gold` | Adds to gold currency | Yes | No |
| `key` | Adds to key count | Yes | No |
| `life` | Adds extra lives | Yes | No |

### 7.2 Floor Item Drops (35% chance per eligible room)

| Item | Type | Value | Slot |
|------|------|-------|------|
| Health Potion | heal | 10 | — |
| Big Health Potion | heal | 25 | — |
| Rusty Sword | weapon | 2 | weapon |
| Iron Shield | armor | 2 | shield |
| Leather Cap | armor | 1 | head |
| Worn Greaves | armor | 1 | legs |
| Copper Ring | ring_atk | 1 | ring |
| Bone Amulet | amulet_hp | 8 | amulet |
| Elixir of Strength | buff_attack | 3 | — |
| Scroll of Protection | buff_defense | 2 | — |
| Scroll of Lightning | scroll | 15 | — |
| Scroll of Obliteration | scroll | 40 | — |

### 7.3 Chest Items (30% chance per eligible room)

| Item | Type | Value | Slot |
|------|------|-------|------|
| Gold Coins | gold | 25 | — |
| Gold Hoard | gold | 75 | — |
| Ruby Necklace | gold | 50 | — |
| Health Potion | heal | 15 | — |
| Fine Sword | weapon | 3 | weapon |
| Reinforced Armor | armor | 3 | chest |
| Iron Helm | armor | 2 | head |
| Chain Leggings | armor | 2 | legs |
| Buckler | armor | 2 | shield |
| Silver Ring | ring_def | 2 | ring |
| Jade Amulet | amulet_hp | 12 | amulet |
| Elixir of Vitality | buff_hp | 10 | — |
| Skeleton Key | key | 1 | — |

#### Chest Types (Weighted Distribution)
| Type | Probability | Behavior |
|------|------------|----------|
| Normal | 60% | Opens freely |
| Locked | 20% | Requires 1 key (consumed), or Rogue 70% lockpick chance |
| Trapped | 20% | Player disarmTrap% to disarm; failure deals `randInt(3, 8 + floor)` damage |

### 7.4 Hidden Items (20% chance per room)

| Item | Type | Value | Slot |
|------|------|-------|------|
| Ancient Coin | gold | 40 | — |
| Hidden Dagger | weapon | 4 | weapon |
| Secret Scroll | scroll | 25 | — |
| Dusty Potion | heal | 20 | — |
| Forgotten Ring | ring_atk | 2 | ring |
| Buried Gem | gold | 60 | — |
| Shadow Amulet | amulet_hp | 15 | amulet |

#### Hidden Item Discovery
- **Passive detection**: When player moves within Manhattan distance 1, roll `Math.random() < player.detectHidden`
- **Active search** (Search button / F key): Roll `Math.random() < player.detectHidden + 0.3`
- **Ranger Scout ability**: Auto-discovers all hidden items in current room
- Must be discovered before pickup (Grab button / G key)

---

## 8. EQUIPMENT SYSTEM

### 8.1 Equipment Slots (7 Total)

| Slot | Stat Bonus | Display Position |
|------|-----------|-----------------|
| Weapon | +ATK | Left column |
| Head | +DEF | Left column |
| Chest | +DEF | Left column |
| Shield | +DEF | Right column |
| Legs | +DEF | Right column |
| Ring | +ATK or +DEF | Right column |
| Amulet | +Max HP | Centered below |

### 8.2 Stat Calculation Formula
```
ATK = baseAttack + bonusAttack + equipmentATK + (level - 1)
DEF = baseDefense + bonusDefense + equipmentDEF + floor((level - 1) / 3)
MaxHP = classBaseHP + (level - 1) * 5 + bonusMaxHP + equipmentHP
```

Where:
- `equipmentATK` = sum of all weapon + ring_atk equipped values
- `equipmentDEF` = sum of all armor + ring_def equipped values
- `equipmentHP` = sum of all amulet_hp equipped values

### 8.3 Equip/Unequip Behavior
- **Equipping**: If slot is occupied, old item returns to inventory. New item removed from inventory.
- **Unequipping**: Item moves from slot back to inventory. Slot becomes null.
- **Auto-equip on pickup**: Weapon, armor, ring, and amulet items auto-equip when picked up from ground/chest.
- Stats are recalculated via `recalcStats()` after every equip/unequip action.
- If maxHP increases, current HP increases by the difference.

### 8.4 Equipment Screen UI
- WoW-style layout with 3-column grid: left slots | character silhouette | right slots
- Amulet slot centered below the grid
- Equipped items show name and stat bonus; empty slots show "- empty -"
- Tap/click equipped slot to unequip
- Equippable inventory items listed below with "tap to equip" prompt
- Stats summary bar at bottom showing current ATK / DEF / HP

---

## 9. SHOP SYSTEM

### 9.1 Shop Tier Calculation
```
tier = min(floor(currentFloor / 5), 4)
```

### 9.2 Shop Spawn Rules
- 40% chance per floor
- Only spawns if `rooms.length > 2`
- Placed in a random middle room (not first or last room)
- Shop tile (`$`) placed at `(room.centerX + 1, room.centerY)`

### 9.3 Shop Inventory by Tier

#### Tier 0 (Floors 1–4)
| Item | Type | Value | Cost | Slot |
|------|------|-------|------|------|
| Health Potion | heal | 15 | 15g | — |
| Iron Dagger | weapon | 2 | 25g | weapon |
| Leather Armor | armor | 1 | 20g | chest |
| Skeleton Key | key | 1 | 20g | — |
| Leather Cap | armor | 1 | 18g | head |

#### Tier 1 (Floors 5–9)
| Item | Type | Value | Cost | Slot |
|------|------|-------|------|------|
| Big Health Potion | heal | 30 | 30g | — |
| Steel Sword | weapon | 3 | 50g | weapon |
| Chain Mail | armor | 2 | 45g | chest |
| Iron Shield | armor | 1 | 35g | shield |
| Scroll of Lightning | scroll | 20 | 35g | — |
| Skeleton Key | key | 1 | 20g | — |

#### Tier 2 (Floors 10–14)
| Item | Type | Value | Cost | Slot |
|------|------|-------|------|------|
| Greater Health Potion | heal | 50 | 50g | — |
| Enchanted Blade | weapon | 5 | 100g | weapon |
| Plate Armor | armor | 4 | 90g | chest |
| Steel Greaves | armor | 2 | 60g | legs |
| Ruby Ring | ring_atk | 2 | 80g | ring |
| Extra Life | life | 1 | 200g | — |

#### Tier 3 (Floors 15–19)
| Item | Type | Value | Cost | Slot |
|------|------|-------|------|------|
| Supreme Health Potion | heal | 80 | 80g | — |
| Mithril Sword | weapon | 7 | 160g | weapon |
| Mithril Armor | armor | 6 | 150g | chest |
| Amulet of Vitality | amulet_hp | 15 | 120g | amulet |
| Extra Life | life | 1 | 200g | — |

#### Tier 4 (Floors 20+)
| Item | Type | Value | Cost | Slot |
|------|------|-------|------|------|
| Elixir of the Gods | heal | 999 | 120g | — |
| Legendary Blade | weapon | 10 | 300g | weapon |
| Dragon Armor | armor | 8 | 280g | chest |
| Dragon Shield | armor | 5 | 220g | shield |
| Sapphire Ring | ring_def | 3 | 180g | ring |
| Extra Life | life | 1 | 250g | — |

---

## 10. DUNGEON GENERATION

### 10.1 Grid Initialization
- 2D array of size `MAP_H (25) x MAP_W (50)`
- All cells initialized to `'#'` (wall)

### 10.2 Room Generation
| Parameter | Value |
|-----------|-------|
| Room width | `randInt(4, 8)` |
| Room height | `randInt(4, 8)` |
| Position X | `randInt(1, MAP_W - width - 1)` |
| Position Y | `randInt(1, MAP_H - height - 1)` |
| Max attempts | `MAX_ROOMS * 3 = 30` |
| Max rooms | 10 |

- Rooms rejected if they overlap any existing room (AABB intersection test)
- Floor tiles (`.`) carved for entire room area

### 10.3 Corridor Generation
Each new room connects to the previous room via L-shaped corridors:
- 50% chance: horizontal first → then vertical
- 50% chance: vertical first → then horizontal
- Corridors carved as single-tile-wide paths

### 10.4 Entity Placement Per Room

| Entity | Condition | Spawn Rule |
|--------|-----------|------------|
| Enemies | Not start/shop/boss room | `randInt(0, 1 + floor(floorNum/3))` enemies |
| Floor Items | Not shop room | 35% chance of 1 random item |
| Chests | Not shop room | 30% chance of 1 chest |
| Hidden Items | Not shop room | 20% chance of 1 hidden item |

### 10.5 Special Tile Placement
| Tile | Character | Placement Rule |
|------|-----------|---------------|
| Stairs | `>` | Center of last room (non-boss floors only) |
| Shop | `$` | Random middle room, `(cx+1, cy)`, 40% chance |
| Boss | Enemy char `B` | Center of last room (boss floors only) |

### 10.6 Room Descriptions
18 atmospheric room descriptions, randomly assigned. Boss rooms override to: *"The air crackles with dark energy..."*

| # | Description |
|---|-------------|
| 1 | A damp chamber with moss creeping up the walls. |
| 2 | Torchlight flickers across ancient stone carvings. |
| 3 | The air is thick with dust and decay. |
| 4 | Bones litter the floor. |
| 5 | Water drips from cracks in the ceiling. |
| 6 | A cold draft blows through this passage. |
| 7 | Faded murals depict a forgotten battle. |
| 8 | The room hums with a strange vibration. |
| 9 | Broken furniture scattered about. |
| 10 | Cobwebs hang thick — nothing passed here in ages. |
| 11 | A faint glow from cracks in the floor. |
| 12 | The walls are scorched black by dragon fire. |
| 13 | Crystals cast prismatic light. |
| 14 | An underground stream trickles through. |
| 15 | Sulfur hangs heavy in the air. |
| 16 | Mushrooms glow with eerie bioluminescence. |
| 17 | Chains sway in unseen wind. |
| 18 | Strange runes pulse faintly on the walls. |

---

## 11. VISIBILITY & FOG OF WAR

### 11.1 Raycasting System
| Parameter | Value |
|-----------|-------|
| Rays cast | 360 (one per degree, 0°–359°) |
| Default radius | 7 tiles |
| Step method | `cos(angle)` / `sin(angle)` per step |
| Rounding | `Math.round()` |

### 11.2 Rules
- Each visible tile is added to a `Set` as key `y * MAP_W + x`
- Rays terminate upon hitting a wall tile, but **the wall itself IS visible**
- Rays terminate at grid boundaries
- All currently visible tiles are merged into the persistent `game.revealed` set

### 11.3 Rendering States
| State | Appearance |
|-------|-----------|
| Currently visible | Full color, entities shown |
| Previously revealed | Dark gray (`#222`), no entities |
| Never seen | Empty space character |

---

## 12. COMBAT SYSTEM

### 12.1 Combat Initialization
- `inCombat` flag set to `true`
- `backstabReady` reset to `true`
- Boss fights play `sfxBoss()`, normal enemies play `sfxHit()`

### 12.2 Player Damage Formula
```
playerDamage = max(1, player.attack + randInt(-1, 3))
```
- If Backstab active: `playerDamage * 2`

### 12.3 Enemy Damage Formula
```
enemyDamage = max(1, enemy.attack + randInt(-1, 2) - player.defense)
```
- If Shield Block active (`blockTurns > 0`): `damage = floor(damage / 2)`
- If Rage active (`rageTurns > 0`): `damage = floor(damage * 1.5)`

### 12.4 Defend Action
```
defendDamage = max(0, floor((enemy.attack + randInt(-1, 2)) / 2) - player.defense)
```
- Defend **skips enemy retaliation** (returns immediately after)
- If Shield Block also active: damage halved again

### 12.5 Combat Actions

| Action | Effect | Enemy Retaliates? |
|--------|--------|-------------------|
| Attack | Deal player damage to enemy | Yes |
| Defend | Take reduced damage | No (damage applied in defend calc) |
| Item | Use consumable from inventory | Yes |
| Run | 40% success chance (not available vs bosses) | Yes (on failure) |
| Ability | Class-specific special action | Yes |

### 12.6 Class Abilities in Combat

| Class | Ability | Effect |
|-------|---------|--------|
| Knight | Shield Block | `blockTurns = 2`, halves incoming damage |
| Rogue | Backstab | `max(1, attack * 2 + randInt(0, 4))` damage |
| Mage | Fireball | `15 + level * 2` damage, 3 uses/floor |
| Ranger | Scout | Reveals all hidden items in current room |
| Cleric | Divine Heal | Heals `15 + level * 2` HP, 3 uses/floor |
| Berserker | Rage | +5 ATK for 3 turns, +50% incoming damage, cannot stack |

### 12.7 Turn Sequence
1. Decrement `blockTurns` and `rageTurns` if active
2. Check boss Phase 2 trigger
3. Execute player's chosen action
4. If enemy alive and action allows retaliation → enemy attacks
5. Check for enemy defeat → rewards
6. Check for player death → respawn or game over

### 12.8 Reward on Enemy Defeat
```
goldReward = randInt(1, 5 + floor * 2)
bossGoldReward = goldReward * 5
```
- XP granted (scaled by floor)
- Boss kills: increment `bossesKilled`, apply drop item, place stairs

---

## 13. PROGRESSION & ECONOMY

### 13.1 Leveling System
| Metric | Formula |
|--------|---------|
| Starting XP to level | 20 |
| XP scaling | `xpToLevel = floor(xpToLevel * 1.5)` per level |
| ATK per level | +1 per level (`level - 1`) |
| DEF per level | +1 per 3 levels (`floor((level - 1) / 3)`) |
| Max HP per level | +5 per level (`(level - 1) * 5`) |
| On level up | Full HP restore, recalcStats |

### 13.2 XP Progression Table (First 10 Levels)
| Level | XP Required | Cumulative XP |
|-------|-------------|---------------|
| 1→2 | 20 | 20 |
| 2→3 | 30 | 50 |
| 3→4 | 45 | 95 |
| 4→5 | 67 | 162 |
| 5→6 | 100 | 262 |
| 6→7 | 150 | 412 |
| 7→8 | 225 | 637 |
| 8→9 | 337 | 974 |
| 9→10 | 505 | 1,479 |
| 10→11 | 757 | 2,236 |

### 13.3 Gold Economy
| Source | Amount |
|--------|--------|
| Enemy kill | `randInt(1, 5 + floor * 2)` |
| Boss kill | Enemy gold × 5 |
| Gold Coins (chest) | 25 |
| Ruby Necklace (chest) | 50 |
| Gold Hoard (chest) | 75 |
| Ancient Coin (hidden) | 40 |
| Buried Gem (hidden) | 60 |

### 13.4 Ability Reset
- Abilities with limited uses (Mage Fireball, Cleric Heal) reset to max when entering a new floor.
- Rogue's `backstabReady` resets at new floor and at start of each combat.

---

## 14. DEATH & RESPAWN SYSTEM

### 14.1 Respawn Behavior
- When HP reaches 0: decrement `lives` by 1
- If `lives >= 0`: restore full HP, reset blockTurns and rageTurns, teleport to start room center
- If `lives < 0`: permanent game over

### 14.2 Life Economy
| Source | Lives |
|--------|-------|
| Starting lives | 5 |
| Extra Life (shop, Tier 2+) | +1 |
| Extra Life (max cost) | 200–250g |

### 14.3 Game End Conditions
| Condition | Trigger |
|-----------|---------|
| **Victory** | Step on stairs when `floor >= MAX_FLOOR (40)` |
| **Game Over** | `player.lives < 0` (all lives exhausted) |

### 14.4 End Screen
- Displays: class, level, kills, bosses killed, gold, lives remaining (victory) or floor reached (game over)
- Deletes save file
- "Play Again" / "Try Again" button reloads page

---

## 15. SAVE/LOAD SYSTEM

### 15.1 Storage
- **Key**: `'dotf_save'` in `localStorage`
- **Format**: JSON string

### 15.2 Saved Data Fields
```
className, classIdx, hp, maxHp, attack, defense,
xp, level, xpToLevel, lives, gold, keys,
inventory (array), floor, kills, bossesKilled,
abilityUsesLeft, detectHidden, disarmTrap,
equipment (object with 7 slots), bonusAttack,
bonusDefense, bonusMaxHp
```

### 15.3 Load Behavior
- Creates new GameState with `skipFloor = true`
- Applies saved properties via `Object.assign`
- **Dungeon is regenerated** (not saved — procedural each load)
- Player placed at first room center
- Equipment and bonus stats restored
- `recalcStats()` called to recalculate derived stats

### 15.4 Limitations
- Dungeon layout, enemy positions, and chest states are NOT persisted
- Loading places the player in a freshly generated dungeon for their current floor
- Save is deleted upon victory or game over

---

## 16. AUDIO SYSTEM

### 16.1 Implementation
- Web Audio API (`AudioContext` / `webkitAudioContext`)
- Square wave oscillator with exponential gain ramp to 0.001
- Default volume: 0.08
- Global `soundEnabled` toggle

### 16.2 Sound Effects

| Function | Frequencies (Hz) | Duration (ms) | Volume | Trigger |
|----------|------------------|---------------|--------|---------|
| `sfxStep` | 200 | 30 | 0.02 | Player moves |
| `sfxHit` | 400 | 80 | 0.08 | Player attacks enemy |
| `sfxPlayerHit` | 150 | 120 | 0.08 | Enemy hits player |
| `sfxKill` | 600→800 | 60→80 | 0.08 | Enemy defeated |
| `sfxLevelUp` | 400→500→600→800 | 100 each | 0.08 | Level up |
| `sfxItem` | 500→700 | 50 each | 0.08 | Item pickup |
| `sfxChest` | 300→500→700 | 60→60→80 | 0.08 | Chest opened |
| `sfxTrap` | 100 | 200 | 0.08 | Trap triggered |
| `sfxShop` | 500→600 | 50 each | 0.08 | Shop entered |
| `sfxBoss` | 200→180→160→300 | 150 each | 0.08 | Boss encounter |
| `sfxDeath` | 400→300→200→100 | 200 each | 0.08 | Player death |
| `sfxVictory` | 523→587→659→698→784→880→988→1047 | 120 each | 0.08 | Game victory |
| `sfxSave` | 600→800 | 50→80 | 0.08 | Game saved |
| `sfxBlock` | 250 | 80 | 0.08 | Shield block |
| `sfxFlee` | 300→250 | 50 each | 0.08 | Flee attempt |
| `sfxStairs` | 300→400→500→600 | 80 each | 0.08 | Descend stairs |

---

## 17. USER INTERFACE

### 17.1 Screen Inventory

| Screen ID | Purpose | Z-Index | Border Color |
|-----------|---------|---------|-------------|
| `#menu-screen` | Main menu / title | 100 | Cyan (#5ff) |
| `#class-screen` | Character selection | 100 | Cyan (#5ff) |
| `#game-container` | Main game (map + HUD + controls) | — | — |
| `#combat-screen` | Turn-based combat | 90 | Red (#f55) |
| `#shop-screen` | Merchant shop | 90 | Green (#5f5) |
| `#item-screen` | Inventory / item use | 95 | Gold (#fc0) |
| `#equip-screen` | Equipment / gear slots | 100 | Orange (#f93) |
| `#legend-screen` | Controls & symbol legend | 100 | Cyan (#5ff) |
| `#end-screen` | Victory / Game Over | 100 | Cyan (#5ff) |

### 17.2 HUD Elements (Portrait Mode)
- Class name & Level
- HP bar (green) with numeric display
- XP bar (cyan) with numeric display
- ATK (orange) & DEF (blue)
- Lives (filled ♥ / empty ♡ hearts)
- Gold (yellow) & Keys
- Kills & Bosses killed
- Floor number / MAX_FLOOR with boss indicator star

### 17.3 Landscape HUD (Compact Sidebar)
- Same data as portrait HUD but condensed
- Displayed inside `#controls` sidebar
- Hidden in portrait mode, shown in landscape via `!important`

### 17.4 Message Log
- Displays last 8 messages
- Auto-scrolls to bottom
- Color-coded by message type (hp, enemy, item, gold, boss, xp, etc.)

### 17.5 Map Display
| Symbol | CSS Class | Color | Glow | Meaning |
|--------|-----------|-------|------|---------|
| `@` | player-color | #4f4 | Green 6px | Player |
| `█` | wall-color | #6a6a8a | — | Wall |
| `·` | floor-color | #444 | — | Floor |
| `>` | stairs-color | #5ff | Cyan 8px | Stairs |
| `$` | shop-color | #5f5 | Green 5px | Shop |
| `C` | chest-color | #fc0 | Orange 5px | Chest |
| `!` | item-color | #ff0 | Yellow 5px | Item |
| `?` | hidden-color | #f5f | Magenta 5px | Hidden item |
| `B` | boss-color | #f5f | Magenta 8px | Boss |
| `r,g,s...` | enemy-color | #f55 | Red 5px | Enemies |

### 17.6 Action Buttons
| Button | Function | Keyboard |
|--------|----------|----------|
| Bag | `openInventory()` | I |
| Equip | `openEquipment()` | E |
| Look | `lookAround()` | L |
| Search | `searchRoom()` | F |
| Grab | `grabHidden()` | G |
| Save | `saveGame()` | P |
| ? | `openLegend()` | ? |

### 17.7 Legend Screen Contents
1. **Map Symbols** — 10 entries with colored symbol previews
2. **Enemy Symbols** — 16 enemy type entries (all lowercase/uppercase chars)
3. **Keyboard Shortcuts** — 11 exploration controls
4. **Combat Keys** — 5 combat action bindings

---

## 18. INPUT & CONTROLS

### 18.1 Keyboard Controls (Exploration)
| Key | Action |
|-----|--------|
| W / ArrowUp | Move up |
| S / ArrowDown | Move down |
| A / ArrowLeft | Move left |
| D / ArrowRight | Move right |
| I | Open inventory |
| E | Open equipment |
| L | Look around |
| F | Search room for secrets |
| G | Grab discovered hidden item |
| P | Save game |
| ? | Open legend / help |

### 18.2 Keyboard Controls (Combat)
| Key | Action |
|-----|--------|
| A / 1 | Attack |
| D / 2 | Defend |
| I / 3 | Use item |
| R / 4 | Run away |
| E / 5 | Use class ability |

### 18.3 Keyboard Guards
Input is blocked when any of these overlays are visible:
- Menu screen, Class select, Shop, Inventory, Equipment, Legend, End screen

### 18.4 Touch Controls (Mobile)
| Gesture | Action |
|---------|--------|
| D-pad buttons | Directional movement (touchstart, instant) |
| Action buttons | touchend → btn.click() (instant) |
| Swipe (>30px, <500ms) | Directional movement |

### 18.5 Touch Interference Prevention
| Issue | Solution |
|-------|----------|
| Double-tap zoom | `dblclick` event prevented |
| Tap highlight | `-webkit-tap-highlight-color: transparent` |
| Text selection | `user-select: none` |
| Pull-to-refresh | `touchmove` prevented on body (except scrollable areas) |
| Context menu | `contextmenu` event prevented |
| 300ms click delay | D-pad uses `touchstart`; other buttons use `touchend` → `.click()` |

---

## 19. RESPONSIVE DESIGN & MOBILE SUPPORT

### 19.1 Breakpoints

| Breakpoint | Target |
|------------|--------|
| Default | Desktop (max-width: 1000px container) |
| `max-width: 600px` | Mobile portrait |
| `max-width: 380px` | Small mobile portrait |
| `orientation: landscape, max-height: 500px` | Mobile landscape |
| `orientation: landscape, max-height: 380px` | Small mobile landscape |

### 19.2 Mobile Portrait (≤600px)
| Element | Change |
|---------|--------|
| Body | padding: 2px |
| Map | font-size: 10px, letter-spacing: 0, line-height: 1.15 |
| HUD | font-size: 11px |
| HP/XP bars | width: 80px, height: 10px |
| Messages | font-size: 11px, max-height: 60px |
| D-pad | 48px cells |
| Action buttons | 9-10px padding, 11px font |
| Class grid | Single column |

### 19.3 Small Mobile Portrait (≤380px)
| Element | Change |
|---------|--------|
| Map | font-size: 8px |
| D-pad | 42px cells |
| Action buttons | 7-8px padding, 10px font |

### 19.4 Mobile Landscape (≤500px height)
| Element | Change |
|---------|--------|
| Layout | CSS Grid: `"map sidebar" / "msg sidebar"` |
| Columns | `1fr 180px` |
| Map | font-size: 10px, line-height: 1.1 |
| Portrait HUD | Hidden |
| Landscape HUD | Shown (compact, inside sidebar) |
| Messages | max-height: 45px |
| D-pad | 40px cells |

### 19.5 Small Landscape (≤380px height)
| Element | Change |
|---------|--------|
| Layout | `"map sidebar"` (single row) |
| Columns | `1fr 170px` |
| Messages | Hidden |
| D-pad | 36px cells |

---

## 20. DEPLOYMENT & HOSTING

### 20.1 Build Requirements
- **None** — single HTML file with no dependencies or build step
- No npm, no bundler, no framework

### 20.2 Deployment Method
- **Netlify Drop**: Drag-and-drop `index.html` to [app.netlify.com/drop](https://app.netlify.com/drop)
- Produces shareable public URL for testers

### 20.3 Browser Compatibility
| Feature | Requirement |
|---------|------------|
| CSS Grid | Modern browsers (Chrome 57+, Firefox 52+, Safari 10.1+) |
| Web Audio API | All modern browsers |
| localStorage | All modern browsers |
| ES6 Classes | Chrome 49+, Firefox 45+, Safari 10+ |
| Touch events | Mobile browsers |
| `touch-action: manipulation` | iOS Safari 9.3+, Chrome 36+ |

### 20.4 Performance Considerations
- Map rendering: Full DOM rewrite per frame (`innerHTML`) — acceptable for 50×25 grid
- Visibility: 360 raycasts × 7 steps = 2,520 tile checks per render
- No requestAnimationFrame loop — renders only on player action (turn-based)
- Audio: Oscillator nodes created and destroyed per sound (no pooling)

---

## APPENDIX A: COLOR PALETTE

| CSS Class | Hex Color | Usage |
|-----------|-----------|-------|
| `.hp-color` | #5f5 | HP, healing, positive effects |
| `.xp-color` | #5ff | XP, cyan highlights |
| `.atk-color` | #f93 | Attack stat, orange UI |
| `.def-color` | #99f | Defense stat, blue UI |
| `.gold-color` | #ff0 | Gold, currency |
| `.enemy-color` | #f55 | Enemies (red glow) |
| `.boss-color` | #f5f | Bosses (magenta glow) |
| `.item-color` | #ff0 | Items (yellow glow) |
| `.chest-color` | #fc0 | Chests (orange glow) |
| `.stairs-color` | #5ff | Stairs (cyan glow) |
| `.shop-color` | #5f5 | Shop (green glow) |
| `.hidden-color` | #f5f | Hidden items (magenta glow) |
| `.player-color` | #4f4 | Player character (green glow) |
| `.wall-color` | #6a6a8a | Dungeon walls |
| `.floor-color` | #444 | Dungeon floors |
| `.life-full` | #f55 | Filled heart (lives) |
| `.life-empty` | #555 | Empty heart (lives) |
| Body background | #0a0a0f | Page background |
| UI panels | #111118 | HUD, controls, messages |
| Map background | #08080e | Map display area |

---

## APPENDIX B: FULL FUNCTION INDEX

| Function | Category | Description |
|----------|----------|-------------|
| `randInt(a,b)` | Utility | Random int [a,b] |
| `pick(arr)` | Utility | Random array element |
| `cloneItem(t)` | Utility | Deep clone item object |
| `generateDungeon(floor)` | Dungeon | Generate grid, rooms, entities |
| `carveH(grid,x1,x2,y)` | Dungeon | Carve horizontal corridor |
| `carveV(grid,y1,y2,x)` | Dungeon | Carve vertical corridor |
| `computeVisibility(grid,px,py,r)` | Rendering | Raycast fog of war |
| `renderMap()` | Rendering | Render ASCII map to DOM |
| `renderHud()` | Rendering | Update both HUDs |
| `renderMessages()` | Rendering | Render message log |
| `renderAll()` | Rendering | Combined render call |
| `addMsg(text,cls)` | Messages | Add message to log |
| `applyItem(item,enemy)` | Items | Apply item effect |
| `movePlayer(dx,dy)` | Movement | Handle player movement |
| `openChest(chest)` | Interaction | Open chest (locked/trapped/normal) |
| `searchRoom()` | Interaction | Search for hidden items |
| `grabHidden()` | Interaction | Pick up discovered hidden item |
| `lookAround()` | Interaction | Describe current room |
| `startCombat(enemy,room)` | Combat | Initialize combat |
| `combatAction(action)` | Combat | Process combat turn |
| `useAbility(enemy)` | Combat | Execute class ability |
| `handleEnemyDefeat(enemy,room)` | Combat | Process enemy death rewards |
| `endCombat(resolved)` | Combat | Clean up combat state |
| `openInventory()` | UI | Show inventory screen |
| `useInventoryItem(idx)` | UI | Use item from inventory |
| `closeInventory()` | UI | Hide inventory screen |
| `openEquipment()` | UI | Show equipment screen |
| `renderEquipment()` | UI | Render equipment slots |
| `closeEquipment()` | UI | Hide equipment screen |
| `unequipSlot(slot)` | Equipment | Unequip item from slot |
| `equipFromInventory(idx)` | Equipment | Equip item from inventory |
| `openLegend()` | UI | Show legend screen |
| `closeLegend()` | UI | Hide legend screen |
| `openShop()` | UI | Show shop screen |
| `buyItem(idx)` | Shop | Purchase shop item |
| `closeShop()` | UI | Hide shop screen |
| `saveGame()` | Persistence | Save to localStorage |
| `loadGame()` | Persistence | Load from localStorage |
| `deleteSave()` | Persistence | Remove save data |
| `hasSave()` | Persistence | Check for existing save |
| `handleDeath()` | Death | Process player death/respawn |
| `checkEndState()` | Game Flow | Check victory/game over |
| `initClassSelect()` | Init | Populate class selection |
| `selectClass(idx)` | Init | Start game with chosen class |
| `startNewGame()` | Init | Transition to class select |
| `toggleSound()` | Settings | Toggle audio on/off |
| `beep(freq,dur,vol)` | Audio | Play oscillator tone |
| `sfxStep/Hit/Kill/...` | Audio | Specific sound effects |

---

*Document generated from source code analysis of `index.html` (v1.0, ~1,845 lines)*
*Game: Depths of the Forgotten — A Roguelike Adventure*
