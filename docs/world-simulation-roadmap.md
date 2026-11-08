# JMod-WoW World Simulation Roadmap

## Purpose

Expand the private AzerothCore WotLK realm from a mostly human-driven private server into a persistent simulated MMO world where server-side bots provide population, gathering, crafting, economy activity, grouping, PvP participation, and social texture without replacing AzerothCore as the source of truth.

The guiding principle is simple: bots should behave like imperfect players, not resource-printing automation.

## Scope

This roadmap covers five connected areas:

1. Player population simulation
2. Farming and profession behavior
3. Auction-house and economy participation
4. Custom game modes and world events
5. Personality, guild, party, and social behavior

The existing JMod admin platform remains the operator and control surface. AzerothCore remains authoritative for game state. JMod-specific scheduling, simulation metadata, policy, metrics, and orchestration stay outside the AzerothCore core databases wherever practical.

## Target Architecture

```text
Humans                         Simulated Players
  |                                  |
  +---------------+------------------+
                  |
             AzerothCore
                  |
       +----------+----------+
       |                     |
   Playerbots            Existing Mods
       |                 (AutoBalance etc.)
       |
   JMod World Director
       |
   +---+-----------------------------+
   |              |                  |
Population     Economy            Game Modes
Scheduler      Controller         / Events
   |              |                  |
   +--------------+------------------+
                  |
             JMod Admin
          Web UI + /jc console
```

## Design Rules

- Do not use external screen-scraping or client automation.
- Prefer server-side Playerbots or equivalent AzerothCore-native mechanisms.
- Do not spawn materials simply to satisfy the economy unless explicitly configured as a recovery mechanism.
- Preserve human-visible travel, gathering, deaths, repairs, vendors, training, auction listing, and other normal player behavior.
- Bots should not be perfectly efficient.
- Bot activity must be observable and administratively controllable.
- Bot population must be capped and scheduled.
- Every automated economy intervention should be measurable.
- Custom game modes should be modular and independently enabled or disabled.
- Avoid invasive AzerothCore core patches when a module, service, or supported hook can accomplish the same job.

## Phase 0: Baseline and Safety

Before introducing Playerbots, capture the current realm state.

### Inventory

- AzerothCore version and branch
- Enabled modules
- Existing `mod-autobalance` configuration
- Realm and character database sizes
- Active account count
- Current population patterns
- Auction House item counts and price distribution
- Economy baseline: gold created, destroyed, and transferred
- Current `/opt/wow-admin` and JesterConsole integration points

### Backup and rollback

- Database backup before migration
- Current AzerothCore Docker image/tag recorded
- Current module revisions recorded
- Tested rollback procedure
- Staging realm or test database strongly preferred

## Phase 1: Playerbot Foundation

### Goal

Introduce autonomous server-side characters that can populate the world without changing game rules yet.

### Initial target

Start small:

- 20-30 bots for burn-in
- 50 bots after stability testing
- 100 bots after CPU, DB, worldserver, and economy behavior are understood

Do not start at 300 bots merely because the configuration permits it. That is how load testing becomes folklore.

### Desired behavior

Bots should be capable of:

- leveling
- questing
- grinding
- travel and flight paths
- death and corpse recovery
- equipment upgrades
- training
- grouping
- dungeon participation
- battleground participation
- profession learning
- gathering
- basic auction-house interaction

### Population distribution

Maintain believable faction, race, class, and level-bracket distributions.

Suggested initial class-role weighting should ensure:

- sufficient tanks for dungeon formation
- sufficient healers for dungeon formation
- majority DPS, as in a real realm
- no extreme oversupply of farming-optimized classes

## Phase 2: Population Scheduler

Bots should not remain permanently online.

### Goals

- simulate daily population cycles
- create quieter overnight periods
- produce evening peaks
- introduce weekend variation
- rotate characters so the world feels populated by individuals rather than static fixtures

### Example weekday curve

| Local time | Target bots |
|---|---:|
| 03:00 | 25 |
| 06:00 | 35 |
| 09:00 | 55 |
| 12:00 | 80 |
| 15:00 | 110 |
| 18:00 | 150 |
| 21:00 | 180 |
| 00:00 | 90 |

The actual values must be tuned to host capacity.

### JMod controls

Add operator controls for:

- global bot cap
- target online population
- faction ratio
- class ratio
- level distribution
- emergency bot shutdown
- maintenance mode
- schedule presets

## Phase 3: Farming and Professions

### Goal

Make useful materials enter the economy through normal world interaction rather than artificial injection.

### Bot archetypes

Bots should have weighted preferences rather than hard-exclusive roles.

Suggested distribution:

| Archetype | Initial share | Main behavior |
|---|---:|---|
| Adventurer | 35% | Questing and leveling |
| Gatherer | 15% | Mining, herbalism, skinning |
| Dungeon player | 15% | Group PvE |
| PvP player | 10% | Battlegrounds |
| Crafter/trader | 10% | Professions and AH |
| Casual/RPG | 10% | Travel, social, mixed content |
| Wildcard | 5% | Mixed or unusual activity |

### Farming realism rules

Gatherers should:

- travel between zones normally
- use appropriate gathering skill levels
- visit trainers
- vendor trash
- repair
- sometimes quest while traveling
- sometimes die
- occasionally change activity
- retain some materials for crafting
- auction only a portion of collected materials

Avoid:

- fixed circular node routes
- 24/7 online farmers
- instant teleporting between nodes
- perfect bag management
- infinite vendor cycles
- deterministic respawn exploitation

### Profession balancing

The World Director should track profession supply across the active bot population.

Example target signals:

- miners per level bracket
- herbalists per level bracket
- skinners per level bracket
- alchemists
- blacksmiths
- engineers
- tailors
- leatherworkers
- enchanters
- jewelcrafters

If one profession becomes underrepresented, change weighting for future bot activity rather than immediately rewriting characters.

## Phase 4: Economy Simulation

### Goal

Create a functioning player-driven market with bots providing liquidity without controlling all prices.

### Economic actors

1. Human players
2. Playerbots listing naturally acquired items
3. Playerbots purchasing useful goods
4. Optional AH market-maker logic
5. World Director economy policy

### World Director metrics

Track at minimum:

- item quantity by auction category
- median and weighted-average price
- listing volume
- sale volume
- material availability
- bot-created listings
- bot purchases
- human purchases
- gold entering the economy
- gold leaving through vendors, repairs, training, AH fees, and other sinks

### Supply-pressure behavior

Example:

```text
Copper Ore supply below threshold
        |
        v
Increase mining preference for eligible bots
        |
        v
Bots travel and gather normally
        |
        v
Some ore reaches AH over time
```

When supply is high, reduce gathering preference rather than deleting auctions.

### Safeguards

- daily bot listing cap
- daily bot purchase cap
- per-item intervention limits
- price floor and ceiling guardrails
- maximum bot share of total AH inventory
- operator kill switch
- audit log for every policy adjustment

## Phase 5: Dungeon and Battleground Population

### Goal

Use bots to make multiplayer content available when human population is low.

### Dungeon behavior

Bots can supplement groups but should not automatically replace every missing slot immediately.

Potential policy:

1. Human group forms.
2. Human players get a short priority period.
3. Missing roles may be filled with bots.
4. Bot difficulty or gear can be constrained by dungeon level.

### Battleground behavior

Use bots to fill queues when enough humans show intent to play.

Example:

- 1 human queues: no forced match
- several humans queue: fill missing slots to launch a match
- prefer faction balance
- avoid always creating perfectly symmetrical compositions

## Phase 6: Persistent Bot Identity

Bots should represent persistent characters rather than anonymous processes.

### Identity data

Store JMod metadata for each simulated player:

- character GUID
- personality profile
- preferred activities
- profession interests
- friends
- guild affiliation
- rivals
- usual play window
- recent goals
- activity history

AzerothCore remains authoritative for the actual character.

### Behavioral memory

Examples:

- remembers frequent party members
- prefers known guildmates
- remembers recent dungeon runs
- tracks profession goals
- remembers losing repeated PvP encounters

The first implementation should use deterministic structured state. LLM memory can come later.

## Phase 7: Guild Simulation

Create bot-populated guilds with different identities.

Example concepts:

- progression/PvE guild
- gathering and crafting guild
- PvP guild
- casual/social guild

Guild affiliation can influence:

- preferred group members
- professions
- content selection
- chat frequency
- trade preferences
- raid participation

Human players should be able to join selected simulated guilds if configured.

## Phase 8: Social and Personality Layer

### Goal

Make bots socially recognizable without allowing an LLM to control gameplay.

### Hard separation

Playerbot AI handles:

- combat
- movement
- navigation
- questing
- farming
- inventory
- professions
- grouping

Optional local language model handles:

- guild chat
- party chat
- whispers
- limited world chat
- personality-consistent responses

The language model must not directly issue combat rotations or unrestricted server commands.

### Privacy and safety

- never transmit account credentials or sensitive server configuration to an external model
- local inference preferred
- allow social AI to be disabled globally
- rate-limit messages
- prevent chat spam
- provide block/mute functionality

## Phase 9: World Director

The World Director is the custom JMod orchestration service.

It does not directly play characters. It adjusts the environment in which Playerbots operate.

### Responsibilities

- active population target
- login/logout scheduling
- faction balancing
- level-bracket balancing
- class-role balancing
- profession weighting
- farming pressure
- AH policy
- dungeon-fill policy
- battleground-fill policy
- guild population
- game-mode participation
- metrics and alerts

### Example decisions

```text
Too few healers online
 -> increase healer login weighting

Low Peacebloom supply
 -> increase herbalism activity weighting

Barrens underpopulated
 -> increase eligible Horde activity there

Four humans queue WSG
 -> permit bot queue fill

Realm CPU high
 -> lower target bot population
```

### Service placement

Proposed repository location:

```text
jmod/world/
  director/
  population/
  economy/
  professions/
  guilds/
  game_modes/
  metrics/
```

## Phase 10: Custom Game Modes

Chris's game-mode ideas should live here once recovered.

Do not mix experimental rules directly into the bot engine.

Each game mode should define:

- purpose
- entry conditions
- player count
- human/bot composition rules
- map or zone scope
- win conditions
- rewards
- cooldowns
- persistence requirements
- admin controls
- rollback/disable behavior

Potential categories:

- survival modes
- faction events
- zone-control events
- dungeon challenges
- permadeath/ironman variants
- seasonal modifiers
- scavenger hunts
- world invasions
- bot-assisted raid events
- alternate leveling rules

## Admin Web Additions

Add a `World` section containing:

### Overview

- humans online
- bots online
- CPU/worldserver load
- active bot target
- faction ratio
- class-role ratio
- bot level distribution

### Population

- schedule editor
- online cap
- force login/logout controls
- archetype distribution

### Economy

- AH inventory metrics
- price movement
- bot listing/purchase volume
- material shortages
- intervention history

### Professions

- active profession distribution
- gathering pressure
- underrepresented professions

### Game Modes

- installed modes
- enable/disable
- scheduling
- current participants

### Social

- simulated guilds
- personality-engine status
- chat limits

## JesterConsole Additions

Possible `/jc` commands:

```text
/jc world status
/jc bots status
/jc bots target 100
/jc bots pause
/jc bots resume
/jc economy status
/jc economy shortages
/jc professions status
/jc modes list
/jc modes start <mode>
/jc modes stop <mode>
```

Every state-changing command must use the existing JMod audit mechanism.

## Metrics

Expose metrics suitable for Prometheus or the existing homelab monitoring stack.

Examples:

- `jmod_bots_online`
- `jmod_bots_target`
- `jmod_bots_online_by_faction`
- `jmod_bots_online_by_class`
- `jmod_bots_online_by_level_bracket`
- `jmod_gather_actions_total`
- `jmod_auction_bot_listings_total`
- `jmod_auction_bot_purchases_total`
- `jmod_world_director_actions_total`
- `jmod_world_director_errors_total`
- `jmod_game_mode_participants`

## Implementation Order

### Milestone A: Discovery

- document current AzerothCore build
- document active modules
- confirm Playerbots-compatible branch strategy
- benchmark current worldserver resource use
- establish backup and rollback

### Milestone B: Bot proof of concept

- enable Playerbots in a test environment
- create/import small bot pool
- run 20-30 bots
- validate questing, travel, combat, death, login rotation

### Milestone C: Population management

- build JMod bot inventory
- build scheduler
- add global caps
- add admin status and emergency stop

### Milestone D: Gathering and professions

- enable gathering behavior
- measure generated materials
- tune profession distribution
- implement farming activity weights

### Milestone E: Economy

- enable controlled bot AH participation
- collect price/supply metrics
- introduce World Director scarcity weighting
- establish intervention limits

### Milestone F: Multiplayer realism

- dungeon assistance
- battleground queue fill
- guild simulation

### Milestone G: Custom game modes

- import Chris's recovered designs
- implement game-mode framework
- ship first mode independently of core bot logic

### Milestone H: Social realism

- deterministic personality profiles
- relationship memory
- optional local LLM chat

## Immediate Next Actions

1. Inspect the live WoW host and capture the exact AzerothCore/container/module state.
2. Confirm how the existing AzerothCore deployment is built and upgraded.
3. Determine the least disruptive Playerbots migration path.
4. Build a test deployment before touching the current realm.
5. Recover Chris's game-mode concepts and add them under Phase 10.
6. Add World Director data models and admin status pages only after the Playerbots proof of concept works.

## Definition of Success

The finished system should feel like a lightly populated persistent WotLK realm rather than a single-player server with background NPCs.

A human player should be able to log in and observe simulated players leveling, gathering, traveling, grouping, participating in the economy, joining events, and appearing repeatedly over time, while the operator retains clear limits, metrics, auditability, and an immediate off switch.

## Upstream Playerbot and City Life integration baseline (reviewed 2026-11-07)

Reuse the existing server-side bot projects; do not implement bot navigation, combat AI, questing, or gathering from scratch.

### Pinned candidate components

- Core: [mod-playerbots AzerothCore fork](https://github.com/mod-playerbots/azerothcore-wotlk/tree/Playerbot), branch `Playerbot`, reviewed at `e7e2e288e7d9ec30e2dfe6f417178a95986fcbb2`.
- Bot engine: [mod-playerbots](https://github.com/mod-playerbots/mod-playerbots), reviewed at `405f81196161017eebae3ee7a612d786a3ed2a9d`. Its random bots support population, leveling, questing, grinding, gear upgrades, groups/raids/battlegrounds, and profession gathering.
- City residents: [mod-playerbots-city-life](https://github.com/chenyifeiai/mod-playerbots-city-life), branch `main`, reviewed at `ce6a05df450f04c4961d09254ad73cf11f1afee2`. It distributes already-online random bots among configured hubs, with weighted hotspots, gradual/time-based population changes, roaming and limited social emotes. It does not create bots or operate the auction-house economy.
- Existing family module source remains in [JMod-WoW](https://github.com/Jestel2/JMod-WoW/tree/feature/jmod-family-server/server/modules/mod-jmod-family) at branch head `3f43e1df56c5de0ff86680ef13854452fbc256ed`. Preserve and rebuild it with the existing `mod-autobalance` module; record and verify the exact AutoBalance revision from the live build before compiling.

These revisions are research pins, not production approvals. Refresh them and review upstream changes before each candidate build.

### Core compatibility boundary

The Playerbots project states that its module requires the Playerbot core fork; the standard AzerothCore repository is unsupported. The current JMod build notes record core pin `9d139488`. GitHub currently reports the Playerbot branch and that pin as diverged (632 commits exclusive to the Playerbot side and 21 commits exclusive to the current pin): [compare](https://github.com/mod-playerbots/azerothcore-wotlk/compare/9d139488...Playerbot).

Therefore build a separate candidate from the Playerbot fork, then port and compile the JMod family and AutoBalance modules against it. Do not try to load Playerbots into the existing production core image or merge the core histories blindly.

### City Life dependency and installation

City Life requires both the Playerbot core fork and `mod-playerbots`. Its version 0.2.0 includes seeded city/town/social hubs, scheduled population scaling, weighted hotspots, a global population cap, and reserved Wintergrasp residents. It borrows bots from the Playerbots online random-bot pool; it cannot create that pool. Its WORLD SQL and module configuration belong only in the isolated candidate database/build until validated.

### Staged rollout gates

1. Record the live core image, module revisions, database sizes, resource headroom and backups; use an isolated staging database and candidate images.
2. Build the Playerbot fork with Playerbots, the current JMod family module and AutoBalance. Explicitly override upstream population defaults: the sample Playerbots config defaults to 500 random bots. Use the roadmap's 20–30 bot burn-in only after checking host capacity.
3. Verify population rotation, movement, travel, questing/grinding, death/recovery, repairs/training and gathering. Start by measuring mining/herbalism behavior; do not enable an auction-house market maker or inject items as part of the first farming test.
4. Test human login, parties, family travel and AutoBalance with bots present. Record worldserver CPU/memory, database load, errors and material/gold changes.
5. Add City Life to that candidate, apply its WORLD SQL to the staging copy, and validate hub caps, faction/level filters, scheduled arrivals/departures and no duplicate bot reservations.
6. Expand the economy, dungeon/battleground filling, guilds, and social layers as separate feature flags with observable limits and a global stop control. Promote the candidate only after staging acceptance and a separately approved production cutover/rollback plan.
