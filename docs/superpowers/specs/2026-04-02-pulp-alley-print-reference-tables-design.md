# Pulp Alley Forge: Print Reference Tables with Modal

**Date:** 2026-04-02
**File:** `PulpAlley/PulpAlley_Forge.html`
**Data:** `PulpAlley/PulpAlley_Data.json` (read-only)

## Overview

Replace direct `window.print()` calls with a Print Settings Modal that lets users toggle which reference tables to include alongside their roster. Tables are smart-defaulted based on league state.

## Print Settings Modal

When the user clicks Print (desktop TopBar button or mobile More panel button), a modal opens instead of printing immediately.

### Modal layout

- **Header**: "Print Roster" with league name
- **Reference Tables section**: 6 checkboxes with labels and brief descriptions
- **Footer**: "Print" button (calls `window.print()`) and "Cancel" button (closes modal)
- **Responsive**: Centered card on desktop (`max-w-lg`), near-full-screen on mobile with padding

### Smart defaults

Each toggle is pre-checked based on current league state:

| Toggle | Label | Default ON when... |
|---|---|---|
| `weapons` | Weapons Table | `league.optionalRules.weapons === true` |
| `harrowingEscape` | Harrowing Escape | `league.campaignState.reputation > 0 \|\| league.campaignState.xp > 0` |
| `associateAbilities` | Associate Abilities | `league.associates.length > 0` |
| `reputationTable` | Reputation Table | `league.campaignState.reputation > 0 \|\| league.campaignState.xp > 0` |
| `secretSociety` | Secret Society Perks | `league.campaignState.secretSociety` is truthy |
| `characterTemplates` | Character Templates | Always off |

All toggles are always available regardless of defaults — user can check/uncheck any.

## PrintableRoster Changes

### New prop

`PrintableRoster` receives `printOptions` prop — an object with boolean keys matching the toggle names above. Defaults to all-false if not provided (backwards compatible).

### Reference appendix sections

After the existing Associates section (end of current PrintableRoster), render enabled reference tables. Each table:

- Wrapped in a `print-card` div with `break-inside: avoid`
- Bold section header using `Playfair Display` at 14px
- Content in `Special Elite` at 10px
- Consistent with existing print styling (black on white, borders)

### Table formats

**Weapons Table** (`printOptions.weapons`):
- Two sub-sections: "Ranged Weapons" and "Melee Weapons"
- Each weapon: bold name, then description
- Data source: `gameData.weapons.ranged` and `gameData.weapons.melee`

**Harrowing Escape** (`printOptions.harrowingEscape`):
- Two sub-tables: "Full Recovery" and "Critical Injury"
- Each row: roll range, bold name, description
- Rendered as a compact HTML table with borders
- Data source: `gameData.harrowingEscape.fullRecovery` and `gameData.harrowingEscape.criticalInjury`

**Associate Abilities** (`printOptions.associateAbilities`):
- Simple list: bold name — description for each of the 15 abilities
- Data source: `gameData.associateAbilities`

**Reputation Table** (`printOptions.reputationTable`):
- HTML table with columns: Threshold, Name, Description
- 22 rows from reputation 10 to 175
- Data source: `gameData.reputationTable`

**Secret Society Perks** (`printOptions.secretSociety`):
- Only prints the league's selected society (not all three)
- Header: society name
- Grouped by rank (1-4), each rank shows its perks: bold name — description
- Data source: `gameData.secretSocieties`, filtered by `league.campaignState.secretSociety`

**Character Templates** (`printOptions.characterTemplates`):
- HTML table with columns: Type, Level, Health, Skill Dice Pools, Skill Dice Types, Ability Slots, Max Ability Level, Roster Slot Cost
- 6 rows (leader, sidekick, ally, follower, gang, epic)
- Data source: `gameData.characterTemplates`

## State Management

### New state in App

```
const [showPrintModal, setShowPrintModal] = useState(false);
const [printOptions, setPrintOptions] = useState(null);
```

When modal opens: compute smart defaults from league state, set `printOptions` to that object.
When modal closes: set `showPrintModal` to false.

### Print flow

1. User clicks Print button → `setShowPrintModal(true)`, compute defaults into `setPrintOptions`
2. Modal renders with checkboxes bound to `printOptions`
3. User toggles checkboxes → updates `printOptions` state
4. User clicks "Print" → `window.print()` is called (PrintableRoster already has options via props)
5. Modal closes after print

### PrintableRoster receives options

`<PrintableRoster league={league} gameData={gameData} printOptions={printOptions || {}} />`

## Cleanup

### Remove dead `harrowing` key

In `EMPTY_LEAGUE` (line 199), remove `harrowing: false` from the `optionalRules` object. Harrowing Escape is a core campaign mechanic, not an optional rule. It's accessible as a print reference table instead.

## What stays unchanged

- Existing print output (roster header, character cards, associates) — untouched
- Desktop and mobile layouts — no changes beyond replacing `window.print()` with modal opener
- Data model / JSON format — no changes
- All other components — no changes
