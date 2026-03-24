# Pulp Alley League Forge — Design Spec

## Overview

A web-based league builder for Pulp Alley 2nd Edition that enforces all character creation rules and constraints. Players create complete league rosters with characters, perks, and associates, then print clean reference cards for tabletop play.

**Reference:** Similar in spirit to One Page Rules Army Forge — wizard-style creation flow, persistent roster sidebar, printable output.

## Architecture

**Approach:** Single HTML file (`PulpAlley_Forge.html`) + external JSON data file (`PulpAlley_Data.json`).

- HTML: React 18 + Tailwind CSS via CDN, Babel for in-browser JSX. Follows the existing Forge pattern used by BlitzTactics, NexusStrike, VoidbreakRaids, etc.
- JSON: Contains all abilities, perks, associate abilities, gang abilities, epic abilities, weapons, genres, secret societies, and reputation table. Loaded via FileReader on first use, cached in LocalStorage.
- No build step. Open HTML in browser, load JSON once, start building.

## Data Model (PulpAlley_Data.json)

**Note on naming:** This file is called `PulpAlley_Data.json` rather than `Universal_Units_PulpAlley.json` because it is a rules-constraint database (abilities, perks, validation rules), not a unit database. Pulp Alley characters are user-created from templates, not selected from a pre-made unit list.

### Top-Level JSON Schema

```json
{
  "characterTemplates": [ ... ],
  "abilities": {
    "level1": [ ... ],
    "level2": [ ... ],
    "level3": [ ... ],
    "level4": [ ... ],
    "epic": [ ... ],
    "gang": [ ... ]
  },
  "perks": [ ... ],
  "associateAbilities": [ ... ],
  "weapons": {
    "ranged": [ ... ],
    "melee": [ ... ]
  },
  "genres": [ ... ],
  "secretSocieties": [ ... ],
  "reputationTable": [ ... ]
}
```

### Character Templates

Each character level defines starting constraints. The "Roster Slot Cost" column is the number of league roster slots consumed by adding this character type.

| Level | Type | Health | # Dice (high/low) | Dice Type (high/low) | Abilities | Roster Slot Cost |
|---|---|---|---|---|---|---|
| 4 | Leader | d10 | 4 skills at 3d / 2 skills at 2d | 4 skills at d10 / 2 skills at d8 | 3 (L1-L4) | 0 |
| 3 | Sidekick | d8 | 3 skills at 3d / 3 skills at 2d | 3 skills at d8 / 3 skills at d6 | 2 (L1-L3) | 3 |
| 2 | Ally | d6 | 2 skills at 2d / 4 skills at 1d | all 6 skills at d6 | 1 (L1-L2) | 2 |
| 1 | Follower | d6* | all 6 skills at 1d | all 6 skills at d6 | 1 (L1) | 1 |
| 2 | Gang | none | special (see Gang Rules) | special | 1 (gang list) | 2 |
| 4 | Epic | d10 | 4 skills at 3d / 2 skills at 2d | 4 skills at d10 / 2 skills at d8 | 3 + 1 epic + Monster free | 5 |

*d6\* = knocked out on failed Health check instead of going down.*

**Worked example — Leader:** The user must assign which 4 of the 6 skills get 3 dice and which 2 get 2 dice, AND which 4 skills get d10 and which 2 get d8. These are independent assignments. E.g., Raven: Brawl 2d8, Shoot 3d10, Dodge 3d10, Might 2d8, Finesse 3d10, Cunning 3d10. Note Shoot later becomes 4d10 after the Marksman ability adds +1 die.

Template entry in JSON:
```json
{
  "type": "leader",
  "level": 4,
  "health": "d10",
  "skillDicePools": { "high": 3, "low": 2, "highCount": 4, "lowCount": 2 },
  "skillDiceTypes": { "high": "d10", "low": "d8", "highCount": 4, "lowCount": 2 },
  "abilitySlots": 3,
  "maxAbilityLevel": 4,
  "rosterSlotCost": 0
}
```

### Skills

Six skills: `Brawl`, `Shoot`, `Dodge`, `Might`, `Finesse`, `Cunning`.

Dice types in order: `d6 → d8 → d10 → d12`. Below d6 = no-dice (cannot roll).

### Abilities

Each ability entry:
```json
{
  "name": "Animal",
  "level": 1,
  "description": "Add +1 die to two skills. Reduce Shoot to no-dice.",
  "effects": [
    { "type": "addDice", "count": 1, "target": "pick2" },
    { "type": "setNoDice", "target": ["Shoot"] }
  ],
  "restrictions": [],
  "incompatibleWith": []
}
```

Categories: Level 1 (~25), Level 2 (~18), Level 3 (~22), Level 4 (~28), Epic (~18), Gang-specific (6).

### Effect Types Reference

All ability effects are expressed as objects in the `effects` array. The full vocabulary:

| Effect Type | Parameters | Description | Example |
|---|---|---|---|
| `addDice` | `count`, `target` | Add dice to skill(s). `target` is a skill name, array of skills, or `"pick1"`/`"pick2"` for user-chosen. | Agile: `{type:"addDice", count:1, target:"Dodge"}` |
| `setNoDice` | `target` (skill array) | Reduce skill(s) to no-dice (cannot roll). | Animal: `{type:"setNoDice", target:["Shoot"]}` |
| `shiftUp` | `target` (skill array) | Raise dice type one step (d6→d8, etc.). Max d12. | Mindless: `{type:"shiftUp", target:["Brawl","Might"]}` |
| `shiftDown` | `target` (skill array) | Lower dice type one step. Below d6 = no-dice. | Mindless: `{type:"shiftDown", target:["Cunning","Finesse"]}` |
| `shiftHealthUp` | — | Raise Health dice type one step. | Big: `{type:"shiftHealthUp"}` |
| `preventAction` | — | Character cannot perform actions. | Beast: `{type:"preventAction"}` |
| `grantFreeAbilities` | `count`, `fromList` (array of ability names) | Select N free abilities from a restricted list. | Beast: `{type:"grantFreeAbilities", count:2, fromList:["Animal","Aquatic",...]}` |
| `movementRestrict` | `maxInches` | Cap movement distance. | Armored: `{type:"movementRestrict", maxInches:6}` |
| `shootingRestrict` | `maxRange` | Cap shooting range. | Close Combat: `{type:"shootingRestrict", maxRange:12}` |
| `coverAlways` | — | Always counts as in cover. | Untouchable: `{type:"coverAlways"}` |
| `rerollSkill` | `skills` (array), `perTurn` | May re-roll one die of the listed skill(s) per turn. | Brute: `{type:"rerollSkill", skills:["Brawl","Might"], perTurn:1}` |
| `healthBonus` | `count` | Roll bonus dice on Health checks. | Armored: `{type:"healthBonus", count:1}` |
| `modifyRosterSlots` | `count`, `restriction` | Add/remove league roster slots with optional type restriction. | Commander: `{type:"modifyRosterSlots", count:4, restriction:"L1L2only"}` |
| `narrative` | — | Effect is purely narrative/gameplay and not computable (e.g., Monster's "horrific" tag). Displayed as description only. | Monster: `{type:"narrative"}` |

Effects that require user choice (like `pick2` targets) are resolved in the UI during character creation, not at data-load time. The chosen targets are stored in the character's saved state.

### Perks

Each perk entry:
```json
{
  "name": "Well Armed",
  "slotCost": 1,
  "category": "background",
  "description": "Once per turn, before you roll Shoot or Brawl for one of your characters, you may cancel a -1 penalty.",
  "prerequisites": [],
  "restrictions": [],
  "leagueStructureOverride": null
}
```

Perks range from 0 to 10 roster slots. Some require other perks (e.g., Ancient Sect requires Dominion). Some restructure the league — these use `leagueStructureOverride` to specify the structural change:

- **Mastermind** (perk, 1 slot): `leagueStructureOverride: "mastermind"`
- **Duo** (perk, 4 slots): `leagueStructureOverride: "duo"`
- **League of Legends** (perk, 10 slots): `leagueStructureOverride: "leagueOfLegends"`
- **Companions** (perk, 1 slot): `leagueStructureOverride: "companions"`
- **Specialists** (perk, 1 slot): `leagueStructureOverride: "specialists"`
- **Supers** is a setting/optional rule, not a perk. It allows upgrading the Leader to Epic at 5 roster slot cost.

### Associates

Each associate has 2 abilities selected from a shared pool of ~16 associate abilities. Max 2 associates at league creation (expandable via Reputation). No duplicate associate abilities across the league.

```json
{
  "name": "Lucky Charm",
  "description": "Your Leader gains Lucky Devil or Danger Sense (pick one)."
}
```

### Secret Societies (Extended)

Three societies (The Society/IRES, The Order/Oginali, The Guild/G13), each with Ranks 1-4, each rank offering 3 perk choices. Unlocked at Reputation 30/55/80/125.

```json
{
  "name": "The Society (I.R.E.S.)",
  "ranks": [
    { "rank": 1, "reputationRequired": 30, "perks": [
      { "name": "The First Cup", "description": "Draw one Fortune Card each time your Leader passes a Recovery check." },
      { "name": "The First Blade", "description": "Draw two Fortune Cards each time your Leader injures an enemy Leader." },
      { "name": "The First Coin", "description": "Draw two Fortune Cards each time your Leader passes a plot point challenge." }
    ]},
    ...
  ]
}
```

### Weapons (Extended)

Ranged: Pistol, SMG, Rifle, Shotgun, Primitive, Thrown.
Melee: Light, Basic, Heavy, Long, Grappling, Shield.
Each selected as a Level 1 ability or free if no other kit.

```json
{
  "name": "Pistol",
  "category": "ranged",
  "description": "You may re-roll one Shoot die during your own activation. Maximum range is 12\"."
}
```

### Genre Presets (Extended)

Fantasy, Western, War, Horror, Sci-Fi, Lost World. Each modifies constraints (e.g., Western: gritty caps dice at d8, free Pistol/Rifle; Fantasy: primitive shooting limits).

```json
{
  "name": "Pulp Westerns",
  "modifiers": [
    { "type": "maxDiceType", "value": "d8" },
    { "type": "freeWeapon", "options": ["Pistol", "Rifle"] },
    { "type": "freeMount", "description": "All characters may begin on a free basic mount." }
  ]
}
```

### Reputation Table (Extended)

Threshold-to-perk mapping from 10 to 175+ Reputation. Awards include extra slots, XP, resources, secret society ranks.

```json
{ "threshold": 25, "name": "Regional Influence", "description": "Your league roster is expanded by +1 slot.", "effect": { "type": "modifyRosterSlots", "count": 1 } }
```

## Validation Engine

### League-Level Constraints

- 10 base roster slots (expandable by Reputation perks, Commander ability)
- Exactly 1 Leader (0 slots) — always present
- Max 1 Sidekick (overridden by Company of Heroes perk → 2)
- Unlimited Allies, Followers, Gangs within slot limits
- Max 2 Associates at start (expandable at 50 and 75 Reputation)
- Perks permanently consume slots

### Character-Level Constraints

- Skill dice count and type must match level template — user assigns which skills get which values
- No duplicate abilities on the same character
- Ability level ≤ character level
- Max 1 ability reducing the same skill to no-dice
- Max 1 ability preventing actions (no-action)
- Specific incompatibility checks (e.g., Dithering incompatible with no-dice/no-action abilities; True Believer prevents all other abilities)

### Ability Effect Tracking

When abilities modify skills:
- `+1 die` effects (Agile, Fierce, Marksman, etc.) add to the dice count
- `shift up/down` effects change the dice type
- `reduce to no-dice` effects override the skill entirely
- Display both base template values and computed final values
- Annotate which abilities contributed to modifications

### Special League Structures

- **Mastermind:** No Leader character. Free Sidekick fills Leader role. +6 extra slots for L1/L2 only.
- **Duo:** Sidekick gets +1 ability and d10 Health. Roster limited to Leader + Sidekick + perks + Associates. No L1/L2 characters.
- **League of Legends:** 4 Sidekicks (included in 10-slot perk cost), no Leader character. One Sidekick fills Leader role.
- **Companions:** Each Ally gets +1 ability. No Sidekick allowed.
- **Supers (setting):** Leader upgrades to Epic at cost of 5 slots.
- **Specialists:** Leader reduces 3 skills one dice-type; all other colleagues raise 1 skill one dice-type.

### Sub-Ability Selection

- **Beast:** Select 2 free abilities from: Animal, Aquatic, Big, Fierce, Mindless, Reanimated, Speedy, Swarm, Winged. Cannot perform actions.
- **Goon:** Select 2 free abilities from: Aquatic, Brute, Fierce, Marksman, Sharp, Slam, Trick, Winged. Cannot perform actions.
- **Shapeshifter:** Create alternate profile of same character type. Action to transform.

### Gang Rules

- 5 models (6 with Sixth-Man)
- Brawl/Shoot/Might: 1d6 per 2 models (rounded up). Dodge/Cunning/Finesse: always 1d6
- 1 gang ability from: Armed, Dangerous, Disciplined, Loyal, Mob, Sixth-Man, plus allowed L1/L2 abilities
- Knocked out at ≤2 models (or ≤1 with Loyal)

### Campaign Tracking (Extended)

- **XP:** Cost to learn = character level. Max new abilities = character level. Max 1 new ability per scenario.
- **Reputation:** Running total, auto-flags unlocked perks from the table.
- **Resources:** Tips, Backup, Gear, Contacts balances. Dominion perk restricts to Minions/Gifts/Cult instead of normal assets.

## UI Layout

### Three-Zone Design

1. **Left sidebar** — League roster overview (always visible)
   - Compact character cards: name, level badge (color-coded), health, slot cost, ability count
   - Running slot counter (e.g., "7/10 slots used") with visual bar
   - Perks section below characters
   - Associates section
   - "+ Add Character / Perk" button

2. **Center panel** — Active workspace (context-sensitive)
   - Character editor (skill assignment, ability picker, image upload)
   - Perk browser with filter/search
   - Associate builder
   - Campaign panel (when enabled)

3. **Top bar** — League name (editable), genre selector, optional rules toggles, Save/Load/Export/Print buttons

### User Flow

1. **Create League** — name it, optionally select genre preset
2. **Leader first** — mandatory starting point, guided character creation
3. **Add colleagues** — pick type, build profile
4. **Add perks** — browse/filter, see slot impact before committing
5. **Add associates** — pick 2 abilities each
6. **Optional: Campaign panel** — toggle on for Reputation/XP/Resources/Secret Society

### Character Creation Sub-Flow

1. Pick character type → template auto-populates
2. **Name** — text input with optional mini image upload (local file, stored as base64)
3. **Assign skill dice** — a 6-row grid (one per skill). Each row has two dropdown selectors:
   - **Dice count dropdown:** selects from the level's available pool (e.g., Leader: four "3" and two "2" slots). Selecting a value removes it from the pool for other skills. Remaining unassigned counts shown above the grid.
   - **Dice type dropdown:** selects from the level's available pool (e.g., Leader: four "d10" and two "d8" slots). Same pool-depletion mechanic.
   - **Final value column:** auto-computed (base + ability modifiers), e.g., "4d10" with annotation if abilities contributed.
   - For Ally (all d6) and Follower (all 1d6), these dropdowns are pre-filled and read-only since there's no choice to make.
4. **Pick abilities** — filtered list by level, search/filter by name or effect. Live incompatibility warnings (greyed out with reason tooltip). Each ability card has editable "flavor name" field.
5. **Review** — final profile card with all computed values

### Mini Image Upload

- Optional per character. Click thumbnail area → file picker → local image loaded via FileReader
- On upload, image is resized to max 128×128px using an offscreen `<canvas>` to keep base64 size under ~25KB per image. This prevents LocalStorage quota issues (5-10MB limit shared across all leagues).
- Stored as base64 data URL in save data (both LocalStorage and JSON export)
- Displayed as small thumbnail on roster sidebar card and character editor header
- Included in print output on the character card
- No server required — purely client-side

## States and Error Handling

### Initial State (No Data Loaded)

If `pulpAlley_dataCache` is not in LocalStorage, the app shows a full-screen prompt to load `PulpAlley_Data.json` via file picker. Same FileReader pattern as the existing Forges. No league editing is possible until data is loaded.

### Empty League

After creating a new league, the center panel shows a guided prompt: "Start by creating your Leader" with a single button. The sidebar shows the empty roster with "0/10 slots used."

### Validation Errors

Displayed as inline warnings on the affected element:
- **Ability conflicts:** Incompatible abilities are greyed out in the picker with a tooltip explaining why (e.g., "Incompatible: character already has a no-action ability").
- **Slot overflow:** If adding a character/perk would exceed roster slots, the "+ Add" button is disabled with a count showing how many slots are available.
- **Incomplete characters:** Characters missing required assignments (e.g., unassigned skill dice) show a warning badge on their sidebar card. A global "Roster has issues" banner appears if any character is incomplete.

### Storage Quota Exceeded

If LocalStorage write fails, show a toast notification: "Storage full — export your leagues as JSON files to free space." The JSON export always works regardless of LocalStorage state.

## Visual Theme: Pulp Dossier

Aged case-file aesthetic — worn manila folders, typewriter text, coffee-stained paper, crimson stamps.

### Color Palette

| Token | Hex | Use |
|---|---|---|
| `--paper` | `#d4c5a9` | Main background, card backgrounds |
| `--paper-dark` | `#c4b48e` | Sidebar, secondary surfaces |
| `--ink` | `#2a1f0e` | Primary text |
| `--ink-light` | `#5a4a2e` | Secondary text, labels |
| `--ink-faded` | `#7a6a4e` | Tertiary text, hints |
| `--leather` | `#3b2f1e` | Top bar, dark surfaces |
| `--crimson` | `#8b1a1a` | Accents, leader highlights, stamps |
| `--border` | `#a89060` | Card borders, dividers |

### Typography

- **Labels/stats:** `Special Elite` or `Courier Prime` (Google Fonts) — typewriter feel
- **Names/headings:** Serif font (e.g., `Playfair Display` or `EB Garamond`) — period-appropriate
- **Body text:** `Roboto Slab` — readable serif for descriptions

### UI Details

- Character cards have subtle drop shadows and a left-border accent (crimson for Leader, muted for others)
- Skill grid styled as a typewriter-output table
- Ability cards have left-border color coding by level
- Slot counter styled as a stamped label
- Subtle paper texture via CSS gradient (no image dependency)
- Print output on white background with period-appropriate borders

## Persistence

### LocalStorage

- Auto-saves current league on every change
- Each league is stored under `pulpAlley_league_{uuid}` where the UUID is generated at creation time. League names are display-only and do not affect storage keys.
- A `pulpAlley_leagueIndex` key stores an array of `{ id, name, lastModified }` for the league list view.
- Data JSON cached under `pulpAlley_dataCache` after first FileReader load
- League list view for switching between saved leagues

### JSON Export/Import

- Full league state as downloadable `.json` file
- Includes: characters (with skill assignments, abilities, flavor names, mini images as base64), perks, associates, campaign state
- Import via FileReader to restore a league

### Print

- `@media print` stylesheet hides all UI chrome
- Renders:
  - One-page roster summary (league name, perks, slot usage, genre)
  - Individual character cards: name, mini image (if set), health, all 6 skills with final dice values, abilities with flavor names and descriptions
  - Associates with abilities
  - Campaign summary (if active)
- Uses `.no-print` / `.print-only` classes matching existing Forge pattern

## Scope Tiers

### Core (Must-Have)

- Full league builder: Leader + Sidekick + Allies + Followers + Gangs
- All abilities (L1-L4) with full constraint validation
- All background perks with slot costs and prerequisites
- Associates with ability selection
- Skill assignment with live ability effect calculation
- Flavor renaming on all abilities
- Optional mini image upload per character
- Save/Load (LocalStorage) + JSON Export/Import
- Print-ready character cards
- Pulp Dossier theme

### Extended (Should-Have)

- Epic character support with epic abilities
- Campaign panel (Reputation tracking, XP spending, Resource management)
- Secret Societies (3 societies, rank progression, perk selection)
- Optional weapons (ranged + melee) as Level 1 ability or free kit
- Genre presets that modify constraints

### Nice-to-Have

- Shapeshifter alternate profile editor
- Harrowing Escape roller
- Pre-made example leagues from the rulebook
- Asset/Gear selector for scenario preparation
- Gun Teams (Level 0 optional characters)

## Tech Stack

- **React 18** + **ReactDOM** via unpkg CDN (UMD builds)
- **Babel standalone** for in-browser JSX
- **Tailwind CSS** via CDN
- **Google Fonts:** Special Elite (typewriter), Playfair Display (headings), Roboto Slab (body)
- `window.define = null` before CDN scripts (existing pattern)
- No build step, no npm, no compilation
