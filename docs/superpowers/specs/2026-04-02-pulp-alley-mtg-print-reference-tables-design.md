# Pulp Alley Forge: MTG Card Print + Complete Reference Tables + Settings

**Date:** 2026-04-02
**Files:** `index.html` (renamed from `PulpAlley_Forge.html`), `PulpAlley_Data.json`

## Overview

Three changes:
1. Replace current PrintableRoster with old MTG card format (63mm x 88mm)
2. Add 19 toggleable reference table sections, configured in Settings, persisted with league
3. Rename main file to `index.html`

## 1. PrintableRoster: MTG Card Format

Replace current simple print cards with the old MTG-sized card layout recovered from commit `6811f61`.

### Card spec (per character)
- **Size**: 63mm x 88mm (standard MTG/poker card)
- **Layout**: 2 cards per row, flex-wrapped with 4mm gap
- **Background**: parchment gradient `linear-gradient(145deg, #d4c5a9, #c4b48e)`
- **Border**: 1.5px solid `#3b2a1a`, 4px border-radius

### Card sections (top to bottom)
1. **Header bar**: Type label (uppercase, crimson) + character name (Playfair Display bold). Background `rgba(59,47,30,0.08)`, bottom border
2. **Body**: Left = character image (22mm x 22mm) or silhouette placeholder. Right = skills in two color-coded boxes:
   - Combat skills (Brawl, Shoot, Dodge) in red/salmon background `rgba(180,80,60,0.15)`
   - Action skills (Might, Finesse, Cunning) in green background `rgba(60,120,60,0.12)`
3. **Gang models**: Below image, "Models: 5" (or 6 with Sixth-Man ability)
4. **Weapon row**: If equipped — bold weapon name (crimson), group label, full description. Background `rgba(139,26,26,0.05)`
5. **Abilities section**: Flex-1 (fills remaining space). Each ability: bold name, flavour name if set, `[free]` tag if free, description. "No abilities" italic if empty.
6. **Health track**: Bottom bar. Shows die progression (e.g., d10 → d8 → d6 → Down → Out). Color-coded: normal black, Down gold, Out crimson with tinted backgrounds. For gangs: KO threshold instead ("Knocked out at ≤2 models" or "≤1 model" with Loyal).

### Health die logic
- Uses template health by default
- Duo perk: sidekick gets d10
- Acting leader badge shown when `league.actingLeaderId` matches

### Roster header
Keep current header with:
- League name + slots used/total
- Genre + modifier descriptions (full text, from our earlier work)
- Optional rule descriptions (full text, from our earlier work)
- Campaign state (reputation, XP, resources, society)

### Associates section
Use old parchment-styled version: gradient background, border `2px solid #3b2a1a`, rounded corners. Each associate: bold name, abilities with descriptions joined by pipes.

## 2. New Data in PulpAlley_Data.json

Add 11 new top-level keys. Data transcribed from Pulp Alley 2E Core Rules PDF.

### `tipsAssets` (array of objects)
Tips spending table (p.93 of rulebook). Fields: `name`, `cost`, `description`.
Assets include: Emergency Exit, Dumb Luck, Dead Drop, Reconnaissance, Deception, Quick Getaway, Rat in the Ranks, Hot Tip, My Lucky Day.

### `contactsAssets` (array of objects)
Contacts spending table (p.94). Fields: `name`, `cost`, `description`.
Assets include: Friendly Local, Good Rumors, Snitch, A Few Maneuvers, Informed, Local Network, Allies Everywhere, Good Friends.

### `gearAssets` (array of objects)
Gear spending table (p.95). Fields: `name`, `cost`, `description`.
Assets include: Diving Suit, Gadget X, Smoke Grenades, Armor Plating, Extra Ammo, Grappling Hook, Medical Kit, etc.

### `gadgets` (array of objects)
Experimental Gadgets table (p.96). Fields: `name`, `cost`, `description`.
Includes Mishap rule text as a preamble note. Assets include: Death Ray, Force Field, Freeze Ray, Jet Pack, Mind Control, Shrink/Growth Ray, Teleporter, etc.

### `backupCharacters` (array of objects)
Backup hired help table (p.97). Fields: `name`, `cost`, `skills`, `health`, `abilities` (optional).
Level 1-3 stat blocks: Brawler, Shooter, Scout, Animal, etc.

### `mounts` (array of objects)
Mount types (p.117-118). Fields: `name`, `description`.
Includes: Skimmer, Basic Mount, Fast Mount, Flying Mount, War Mount.

### `vehicles` (array of objects)
Vehicle types (p.119-122). Fields: `name`, `description`.
Includes: Light Vehicle, Standard Vehicle, Heavy Vehicle, plus modification rules.

### `minions` (array of objects)
Dominion perk minions (p.98). Fields: `name`, `cost`, `health`, `skills`, `abilities` (optional).
Includes: Living Dead, Mindless, Swarm, Dark Minion, etc.

### `cultAssets` (array of objects)
Dominion perk cult assets (p.99). Fields: `name`, `cost`, `description`.
Includes: Eyes and Ears, The Following, Receive Dreams, Gathering, Sacrifice, Dark Lair.

### `gifts` (array of objects)
Dominion perk gifts (p.100). Fields: `name`, `cost`, `description`.
Includes: Aura of Night, Glyph of Knowing, Shadow Step, Dark Speech, Unnatural Endurance, Cursed Idol, etc.

### `quickReference` (object)
Action Sequence quick reference (p.2). Fields: `phases` (array of `{ name, description }` objects covering Direct, Act, actions, end of turn flow).

## 3. Print Settings in League State

### EMPTY_LEAGUE changes

Add `printSettings` object to league state:

```js
printSettings: {
    weapons: false,
    harrowingEscape: false,
    associateAbilities: false,
    reputationTable: false,
    secretSociety: false,
    characterTemplates: true,
    genres: false,
    perks: false,
    tipsAssets: false,
    contactsAssets: false,
    gearAssets: false,
    gadgets: false,
    backupCharacters: false,
    mounts: false,
    vehicles: false,
    minions: false,
    cultAssets: false,
    gifts: false,
    quickReference: true,
}
```

Only `characterTemplates` and `quickReference` default to ON.

### Backwards compatibility

When loading a league that has no `printSettings` key, default to the `EMPTY_LEAGUE` printSettings. In the league load path, add:

```js
if (!league.printSettings) league.printSettings = EMPTY_LEAGUE().printSettings;
```

## 4. Settings Panel: Print References Section

Add a new section to `SettingsPanel` below the existing "Advanced Settings" section.

### Section header
"Print Reference Tables" — `typewriter text-xs uppercase tracking-wider`, crimson color

### Grouped toggles (same toggle switch UI as existing settings)

**Core Reference**
- Quick Reference — Action sequence summary
- Character Templates — Stat templates for all character types
- Weapons Table — Ranged and melee weapons
- Perks Reference — All 40 league perks
- Genre Reference — Selected genre description and modifiers

**Campaign**
- Harrowing Escape — Full Recovery & Critical Injury tables
- Reputation Table — 22 reputation milestones
- Secret Society Perks — Selected society's ranks and perks
- Associate Abilities — All 15 associate abilities

**Resources & Assets**
- Tips Assets — Tips spending table
- Contacts Assets — Contacts spending table
- Gear Assets — Gear spending table
- Gadgets — Experimental gadgets with Mishap rules
- Backup Characters — Hired help stat blocks

**Mounts & Vehicles**
- Mounts — Mount types and rules
- Vehicles — Vehicle types and rules

**Dominion**
- Minions — Dominion perk minion stat blocks
- Cult Assets — Dominion perk cult assets
- Gifts — Dominion perk gift assets

Each toggle updates `league.printSettings[key]` via `updateLeague`.

## 5. Print Button Behavior

Print button (desktop TopBar and mobile More panel) calls `window.print()` directly. No modal.

`PrintableRoster` reads `league.printSettings` to determine which reference table sections to render after the character cards and associates.

## 6. Remove PrintModal

- Delete the `PrintModal` component
- Delete `showPrintModal` state from App
- Remove `onPrint` prop from TopBar and MorePanel
- Revert Print buttons to `onClick={() => window.print()}`

## 7. File Rename

- `PulpAlley_Forge.html` → `index.html`
- `deploy/index.html` stays as `index.html` (already named correctly)
- Update any internal references if needed (there shouldn't be any since it's self-contained)

## 8. PrintableRoster Reference Sections

Each enabled reference table renders as a print-card styled section after the Associates block. Consistent styling:
- `breakInside: avoid`, `pageBreakInside: avoid`
- Border: `2px solid #000`, border-radius 4px, padding 12px
- Header: Playfair Display 14px bold, bottom border
- Content: Special Elite 10px (9px for dense tables)
- Tables use collapsed borders, alternating subtle backgrounds optional

### Section rendering order
1. Quick Reference
2. Character Templates
3. Weapons
4. Perks
5. Genre
6. Harrowing Escape
7. Reputation Table
8. Secret Society
9. Associate Abilities
10. Tips Assets
11. Contacts Assets
12. Gear Assets
13. Gadgets
14. Backup Characters
15. Mounts
16. Vehicles
17. Minions
18. Cult Assets
19. Gifts

## What stays unchanged

- Mobile layout (bottom tab bar, responsive TopBar)
- Desktop layout (sidebar + center panel)
- All editor components (CharacterEditor, PerkBrowser, AssociateBuilder, CampaignPanel)
- Data model for characters, abilities, perks, associates
- Existing JSON data structure (only additions, no changes to existing keys)
