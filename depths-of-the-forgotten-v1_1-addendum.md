# Depths of the Forgotten — v1.1 Recommended Additions

This addendum expands the existing technical requirements with higher-priority gameplay, replayability, UX, and balancing improvements while preserving the game's current core identity as a text-based roguelike dungeon crawler. [file:2]

## 21. Run Variety Systems

The current design already includes procedural floors, class-specific abilities, loot tiers, hidden items, shops, and boss floors. [file:2] This section adds systems that make individual runs feel more distinct instead of only becoming numerically stronger over time. [file:2]

### 21.1 Relics / Artifacts

- Add a new item category: `relic`.
- Relics are passive, run-only modifiers that cannot be sold and do not consume gear slots.
- Maximum relic capacity: 5 active relics per run.
- Relics are awarded from boss kills, elite encounters, rare chests, shrines, or secret rooms.
- Relics persist through death/respawn during the run but are lost on full game over.

#### Example relic effects

| Relic Name | Effect | Notes |
|---|---|---|
| Shadow Fang | Backstab applies Bleed for 3 turns | Rogue synergy |
| Saint's Ember | Healing also grants 5 Barrier | Cleric synergy |
| Merchant's Coin | Shop prices -15%, chest gold -25% | Economy tradeoff |
| Blood Sigil | +3 ATK when below 40% HP | Berserker synergy |
| Lantern of Truth | Hidden item detection chance +25 | Exploration synergy |
| Arcane Ash | Fireball gains +50% damage, uses -1 per floor | Mage tradeoff |

### 21.2 Floor Mutators

- Each floor has a 20% chance to roll a mutator.
- Boss floors never roll standard mutators unless specifically scripted.
- Mutators must be displayed in the HUD and announced when entering the floor.

#### Example mutators

- **Darkness**: visibility radius reduced from 7 to 5.
- **Blood Price**: enemy damage +20%, gold drops +30%.
- **Arcane Surge**: ability effects +25%, consumable drop rate -20%.
- **Greed**: shop prices +25%, chest loot quality increased.
- **Rot**: healing items restore 25% less HP.

### 21.3 Special Rooms

Add distinct nonstandard room types to increase floor identity and decision-making.

| Room Type | Spawn Chance | Effect |
|---|---:|---|
| Shrine | 8% | Gain blessing, curse, or stat tradeoff |
| Campfire | 6% | Heal 30% HP or restore 1 ability use |
| Prison Cell | 5% | Rescue NPC for reward or ambush trigger |
| Vault | 4% | Locked treasure room with key requirement or elite guard |
| Lore Chamber | 6% | Provides world text, boss hint, or relic clue |
| Cursed Altar | 4% | Sacrifice HP/life for major reward |

- Special rooms override normal room flavor text. [file:2]
- Shops and boss rooms remain mutually exclusive with most special room types. [file:2]

## 22. Combat Depth Expansion

The current combat system includes attack, defend, item use, running, and class abilities in a turn-based structure. [file:2] This section adds tactical clarity and more decision points without changing the core turn flow. [file:2]

### 22.1 Enemy Intent Preview

- At the start of each enemy turn, display a short intent message.
- Enemy intents are visible before the player selects an action.
- Intent examples:
  - `The Troll is winding up a heavy strike.`
  - `The Dark Wizard is preparing a curse.`
  - `The Bandit looks ready to evade.`
- Bosses must always telegraph special moves at least 1 turn in advance.

### 22.2 Status Effects

Add timed combat effects to both player and enemies.

| Status | Target | Effect | Duration |
|---|---|---|---|
| Bleed | Player / Enemy | Take damage at turn start | 3 turns |
| Poison | Player / Enemy | Fixed damage each turn | 4 turns |
| Burn | Player / Enemy | Damage over time, reduced healing | 3 turns |
| Weaken | Player / Enemy | -25% ATK | 2 turns |
| Barrier | Player / Enemy | Absorbs fixed damage before HP | 2 turns or until broken |
| Regen | Player / Enemy | Heal at turn end | 3 turns |
| Stun | Player / Enemy | Skip next action | 1 turn |
| Curse | Player | -1 to all item healing rolls, +10% incoming boss damage | 3 turns |

Implementation notes:
- Status effects stack only if explicitly flagged as stackable.
- Reapplying a non-stackable status refreshes duration, not potency.
- Status effects must be shown in the combat HUD.

### 22.3 Ability Cooldowns and Charges

Current class abilities include unlimited-use skills for Knight, Rogue, Ranger, and Berserker, plus per-floor charges for Mage and Cleric. [file:2] To increase tension and reduce repetitive optimal play, add cooldown or charge tuning as follows: [file:2]

| Class | Current | Recommended Change |
|---|---|---|
| Knight | Shield Block unlimited | 2-turn cooldown after use |
| Rogue | Backstab unlimited | Only gains bonus if unused in prior combat round |
| Mage | Fireball 3 uses/floor | Keep as-is, add Burn chance 25% |
| Ranger | Scout unlimited | 3-turn cooldown in combat; free out of combat |
| Cleric | Divine Heal 3 uses/floor | Keep as-is, healing can overheal into Barrier with relic synergy |
| Berserker | Rage unlimited, non-stackable | Add 3-turn cooldown after expiration |

## 23. Enemy Behavior and Roles

The current enemy roster is well-sized and already scales by floor. [file:2] To improve combat variety, each enemy type should also have a behavioral role and optional special trait. [file:2]

### 23.1 Enemy Roles

| Role | Behavior |
|---|---|
| Bruiser | High damage, slow telegraphed attacks |
| Skirmisher | Lower HP, acts fast, may evade |
| Caster | Uses curses, burn, weaken, or ranged effects |
| Tank | High defense, low speed, can gain Barrier |
| Swarm | Low HP, appears in groups, benefits from numbers |
| Ambusher | Gains bonus damage on first turn |

### 23.2 Trait System

- 15% of normal enemies spawn with one trait.
- 35% of elite enemies spawn with one or two traits.
- Bosses use fixed signature traits instead of random ones.

#### Example traits

- **Armored**: -2 incoming damage from attacks.
- **Venomous**: attacks have 30% chance to Poison.
- **Savage**: +20% damage when below 50% HP.
- **Phasing**: first hit each combat deals half damage.
- **Watcher**: detects hidden player advantages; reduces Backstab bonus by 25%.
- **Blessed**: starts combat with Barrier 10.

## 24. Boss Identity Expansion

The current game includes 8 bosses and phase 2 triggers. [file:2] Each boss should be upgraded from a stronger enemy into a distinct encounter with a signature rule, telegraphed attack pattern, and clear counterplay. [file:2]

### 24.1 Boss Design Rules

- Every boss has 1 core mechanic, 1 phase shift behavior, and 1 unique reward.
- Every boss special attack must be telegraphed 1 turn before execution.
- Every boss should alter player priorities, not only increase HP/ATK values.

### 24.2 Example Boss Mechanics

| Boss Archetype | Signature Mechanic | Counterplay |
|---|---|---|
| Mirror Knight | Repeats the last player action against them | Vary action sequence |
| Bone Necromancer | Summons skeleton adds until ward totems are destroyed | Target summons support objects first |
| Iron Warden | Locks item usage for 2 turns | Time consumables before lockout |
| Broodmother | Applies poison eggs that hatch after 2 turns | Burst before hatch or defend through swarm |
| Ash Tyrant | Gains power when player uses fire or rage effects | Shift to defense or items |
| Hollow Saint | Heals whenever player defends twice in a row | Mix attack timing and item usage |

### 24.3 Boss Rewards

Boss kills currently grant scaled rewards and item drops. [file:2] In addition, each boss should grant one of the following: [file:2]

- A guaranteed relic.
- A class-tagged item.
- A permanent run modifier choice between 2 rewards.
- A max HP, ATK, or DEF blessing.

## 25. Equipment and Buildcraft Systems

The current equipment system uses 7 slots and bonus stats. [file:2] To make loot decisions more interesting, add build-focused mechanics beyond raw stat upgrades. [file:2]

### 25.1 Set Bonuses

- Add named gear sets with 2-piece and 3-piece thresholds.
- A set may span weapon, chest, shield, legs, ring, or amulet.
- Set bonuses must be shown in the equipment UI.

| Set Name | 2-Piece Bonus | 3-Piece Bonus |
|---|---|---|
| Warden Set | +2 DEF | First defend each combat grants Barrier 8 |
| Shadow Set | +15% crit-equivalent Backstab damage | Hidden items easier to detect |
| Ember Set | Burn effects deal +2 damage per tick | Fireball gains +1 use per floor |
| Pilgrim Set | +10 max HP | Healing items restore +20% |

### 25.2 Affix System

- Rare and higher-tier items may spawn with 1 prefix or suffix affix.
- Affixes can alter stats, status application, economy, or utility.

Example affixes:
- `Sturdy`: +1 DEF.
- `Jagged`: attacks have 15% chance to Bleed.
- `Prospector's`: +10% gold from kills.
- `Blessed`: +5 max HP.
- `Swift`: improves run chance by 10%.

## 26. Meta Progression and Unlocks

The game currently supports class selection, persistent saves, and end-state tracking such as kills, bosses killed, and floor reached. [file:2] Add light meta progression that unlocks options and knowledge without undermining roguelike tension. [file:2]

### 26.1 Unlock Philosophy

- Permanent unlocks should expand variety, not give large direct power increases.
- Avoid flat permanent stat bonuses like +5 base ATK for all future runs.
- Preferred unlock categories: starting loadouts, relic pool expansion, bestiary info, cosmetic map themes, alternate boss variants.

### 26.2 Suggested Unlocks

| Unlock Trigger | Reward |
|---|---|
| Reach floor 10 with Knight | Alternate Knight starter shield |
| Kill 25 skeleton-type enemies | Skeleton bestiary page reveals weakness |
| Defeat first boss with Rogue | Unlock Shadow relic pool |
| Win a run | Unlock hard mode mutators |
| Discover 20 hidden items total | Unlock Lantern-themed dungeon palette |

### 26.3 Bestiary

- Add a bestiary screen accessible from the main menu.
- Bestiary tracks enemy kills, observed traits, common drops, and discovered special behaviors.
- Unknown fields remain hidden until enough encounters are completed.

## 27. UX and Accessibility Improvements

The game already includes multiple UI screens, keyboard shortcuts, touch controls, and responsive support. [file:2] This section improves usability, readability, and fairness of information presentation. [file:2]

### 27.1 Gear Comparison

- When selecting an equippable item, show side-by-side comparison versus the currently equipped slot item.
- Highlight stat changes using `+` green and `-` red indicators.
- If an item activates or breaks a set bonus, show that result before confirmation.

### 27.2 Combat Log Prioritization

- Split message classes into: `critical`, `combat`, `loot`, `system`, `warning`, `boss`.
- Pin the latest critical event at the top of the combat log area.
- Boss telegraphs and low-HP warnings must override generic hit messages visually.

### 27.3 Accessibility Settings

Add an Accessibility menu with the following options:

- Colorblind-safe palette toggle.
- Reduced flash / reduced screen shake mode.
- Larger UI text mode (+2px base body size).
- High-contrast ASCII map symbols.
- Persistent keybinding legend overlay option.
- Touch control size scaling on mobile.

### 27.4 Danger and Information Indicators

- Add low HP warning at 30% or less.
- Add floor mutator banner on floor entry.
- Add boss room proximity hint if the player enters the final room cluster on boss floors.
- Add hidden-item probability feedback when Search is used unsuccessfully.

## 28. Save / Load Model Revision

The current save/load behavior restores the player state but regenerates the dungeon layout instead of preserving exact room, enemy, and chest state. [file:2] This is functional but may create fairness, exploitation, or continuity issues depending on player expectations. [file:2]

### 28.1 Recommended Save Model

Adopt a **suspend save** system:

- Saving stores complete game state including current floor grid, revealed tiles, room states, enemy states, chest states, shop inventory, mutator, and special room outcomes.
- Loading restores the exact saved state.
- After a successful load, the save file is immediately deleted.
- Suspend save prevents save-scumming while preserving session continuity.

### 28.2 Minimum Saved Fields

In addition to current saved data fields, persist the following: [file:2]

- Dungeon grid tiles.
- Revealed tile set.
- Room list and special room type data.
- Enemy HP, traits, statuses, and positions.
- Chest state (opened/closed/locked/trapped).
- Hidden item discovery state.
- Current floor mutator.
- Active relics.
- Active status effects.
- Boss phase state.
- Shop stock state.

## 29. Analytics and Balance Targets

The current document contains formulas for XP, combat scaling, rewards, and leveling. [file:2] Add explicit balancing and telemetry targets so future tuning can be data-driven rather than only intuition-based. [file:2]

### 29.1 Core Metrics to Track

| Metric | Purpose |
|---|---|
| Death floor by class | Detect weak or overloaded classes |
| Win rate by class | Measure balance across full runs |
| Boss attempt / success rate | Find overtuned encounters |
| Gold earned vs spent | Validate economy pacing |
| Potion use frequency | Measure sustain pressure |
| Item pickup vs equip rate | Detect uninteresting loot |
| Save/load abandonment point | Find friction spikes |
| Hidden item discovery rate | Evaluate exploration value |

### 29.2 Balance Targets

Set initial targets for playtesting and rebalance against observed data.

| Target Area | Suggested Baseline |
|---|---|
| First boss reach rate | 70%+ of new players |
| Run win rate | 5% to 15% for first 20 runs |
| Class spread | No class exceeds others by more than 10 percentage points win rate |
| Shop purchase cadence | 1 meaningful purchase every 2 to 3 floors |
| Hidden item relevance | At least 50% of discovered hidden items should feel useful |
| Death causes | No single generic enemy should account for over 20% of total deaths |

### 29.3 Floor Pacing Matrix

Add a balancing appendix that defines expected player power by floor band.

| Floor Band | Expected Level | Expected Gear Tier | Encounter Goal |
|---|---:|---|---|
| 1-5 | 1-3 | Starter / Tier 1 | Learn loop, low punishment |
| 6-10 | 3-5 | Tier 1 / Tier 2 | Introduce harder roles and first relics |
| 11-20 | 5-8 | Tier 2 / Tier 3 | Build identity solidifies |
| 21-30 | 8-11 | Tier 3 / Tier 4 | Strong synergy checks |
| 31-40 | 11-14 | Tier 4 / Legendary | Endgame pressure and boss mastery |

## 30. Content Pacing and Presentation

The game already uses atmospheric room descriptions, map symbols, and screen overlays. [file:2] Add stronger presentation rules so the growing feature set stays readable and cohesive. [file:2]

### 30.1 Floor Introduction Format

Each new floor should display:
- Floor number.
- Floor mutator, if any.
- Threat level label.
- One short atmospheric line.
- Special note if boss floor, elite floor, or shrine-rich floor.

### 30.2 Elite Enemies

- Add elite variants beginning on floor 6.
- Elite enemies have +25% HP, +15% damage, 1 guaranteed trait, and improved drop quality.
- Elite spawn rate starts at 5% and scales to 15% by floor 30.

### 30.3 Encounter Variety Limits

- No room should contain more than one elite unless scripted.
- Identical enemy types should not compose more than 60% of a floor's encounter population.
- Avoid placing hard counters against a specific class on more than 2 consecutive floors.

## 31. New UI Screens

The current screen inventory includes menu, class select, game, combat, shop, inventory, equipment, legend, and end screen. [file:2] Add the following screens or panels: [file:2]

- Relics panel.
- Accessibility settings panel.
- Bestiary screen.
- Run summary panel after death or victory.
- Floor mutator information popover.

### 31.1 Run Summary Data

The end screen should additionally display:
- Total relics found.
- Special rooms visited.
- Elite kills.
- Status effects applied / suffered.
- Gold spent vs saved.
- Favorite action frequency.

## 32. Implementation Priority

Recommended implementation order:

1. Enemy intents and status effects.
2. Relics / artifacts.
3. Special rooms and floor mutators.
4. Boss identity mechanics.
5. Gear comparison and accessibility menu.
6. Set bonuses and affixes.
7. Suspend-save model.
8. Bestiary and light meta unlocks.
9. Analytics instrumentation and balance matrix.

## 33. Acceptance Criteria

The following acceptance criteria should be added for v1.1 scope completion:

- At least 20 relics implemented.
- At least 6 floor mutators implemented.
- At least 6 special room types implemented.
- All bosses have 1 telegraphed signature mechanic.
- At least 8 status effects implemented and surfaced in UI.
- At least 12 enemy traits implemented.
- At least 4 item sets implemented.
- Gear comparison UI functional on desktop and mobile.
- Accessibility menu functional on desktop and mobile.
- Suspend-save fully restores exact floor state.
- Bestiary unlock system operational.
- End-run analytics summary visible to player.
