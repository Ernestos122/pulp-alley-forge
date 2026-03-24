# Pulp Alley League Forge Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a web-based Pulp Alley 2nd Edition league builder that enforces all character creation rules and produces printable roster cards.

**Architecture:** Single HTML file (`PulpAlley_Forge.html`) with React 18 + Tailwind CSS via CDN, plus an external JSON data file (`PulpAlley_Data.json`) containing all abilities, perks, and constraints. Follows the existing Forge pattern in this project — no build step, open-in-browser.

**Tech Stack:** React 18 (UMD/CDN), Babel standalone, Tailwind CSS (CDN), Google Fonts (Special Elite, Playfair Display, Roboto Slab)

**Spec:** `docs/superpowers/specs/2026-03-23-pulp-alley-league-forge-design.md`

**Existing pattern reference:** `BlitzTacticsBundle_1/BlitzTactics/BlitzTactics_Forge.html` — study its CDN imports, `window.define = null` pattern, `createIcon()` utility, print stylesheet, FileReader JSON loading, and LocalStorage save/load pattern.

**Testing approach:** This is a no-build-step single HTML file with no test runner. Each task produces a browser-verifiable increment. "Verify" steps mean open/refresh the HTML in a browser and confirm the described behavior. For agentic workers without a browser, verify by reading the generated code for correctness against the spec.

---

## File Structure

```
BlitzTacticsBundle_1/PulpAlley/
├── PulpAlley_Forge.html    # Main app (single-file React 18 + Tailwind)
├── PulpAlley_Data.json     # All abilities, perks, associates, gangs, constraints
└── index.html              # (optional) Hub page linking to Forge
```

The HTML file is organized into numbered sections matching the existing Forge pattern:

```
1. ICONS & CONSTANTS        — SVG icon factory, CSS classes, dice type constants
2. DATA & PERSISTENCE       — LocalStorage helpers, JSON load/cache, UUID gen
3. VALIDATION ENGINE         — Constraint checking functions (pure logic, no UI)
4. SKILL COMPUTATION         — Compute final skill values from base + ability effects
5. REACT COMPONENTS          — All UI components
   5a. App shell & routing
   5b. TopBar
   5c. RosterSidebar
   5d. CharacterEditor (skill grid, ability picker, image upload)
   5e. PerkBrowser
   5f. AssociateBrowser
   5g. GangEditor
   5h. AddCharacterModal
6. PRINT OUTPUT              — Print-only character cards and roster summary
7. APP INIT                  — ReactDOM.createRoot, render
```

---

## Task 1: Create PulpAlley_Data.json — Character Templates + Level 1 Abilities

**Files:**
- Create: `BlitzTacticsBundle_1/PulpAlley/PulpAlley_Data.json`

This is the foundation. Start with character templates and Level 1 abilities only. We will add more data in subsequent tasks.

- [ ] **Step 1: Create the JSON file with character templates**

Create `PulpAlley_Data.json` with this structure. All 6 character templates (leader, sidekick, ally, follower, gang, epic):

```json
{
  "characterTemplates": [
    {
      "type": "leader",
      "level": 4,
      "health": "d10",
      "skillDicePools": { "high": 3, "low": 2, "highCount": 4, "lowCount": 2 },
      "skillDiceTypes": { "high": "d10", "low": "d8", "highCount": 4, "lowCount": 2 },
      "abilitySlots": 3,
      "maxAbilityLevel": 4,
      "rosterSlotCost": 0
    },
    {
      "type": "sidekick",
      "level": 3,
      "health": "d8",
      "skillDicePools": { "high": 3, "low": 2, "highCount": 3, "lowCount": 3 },
      "skillDiceTypes": { "high": "d8", "low": "d6", "highCount": 3, "lowCount": 3 },
      "abilitySlots": 2,
      "maxAbilityLevel": 3,
      "rosterSlotCost": 3
    },
    {
      "type": "ally",
      "level": 2,
      "health": "d6",
      "skillDicePools": { "high": 2, "low": 1, "highCount": 2, "lowCount": 4 },
      "skillDiceTypes": { "high": "d6", "low": "d6", "highCount": 6, "lowCount": 0 },
      "abilitySlots": 1,
      "maxAbilityLevel": 2,
      "rosterSlotCost": 2
    },
    {
      "type": "follower",
      "level": 1,
      "health": "d6*",
      "skillDicePools": { "high": 1, "low": 1, "highCount": 6, "lowCount": 0 },
      "skillDiceTypes": { "high": "d6", "low": "d6", "highCount": 6, "lowCount": 0 },
      "abilitySlots": 1,
      "maxAbilityLevel": 1,
      "rosterSlotCost": 1
    },
    {
      "type": "gang",
      "level": 2,
      "health": "none",
      "skillDicePools": null,
      "skillDiceTypes": null,
      "abilitySlots": 1,
      "maxAbilityLevel": 2,
      "rosterSlotCost": 2,
      "models": 5,
      "gangSkillRules": "Brawl/Shoot/Might: 1d6 per 2 models (rounded up). Dodge/Cunning/Finesse: always 1d6."
    },
    {
      "type": "epic",
      "level": 4,
      "health": "d10",
      "skillDicePools": { "high": 3, "low": 2, "highCount": 4, "lowCount": 2 },
      "skillDiceTypes": { "high": "d10", "low": "d8", "highCount": 4, "lowCount": 2 },
      "abilitySlots": 3,
      "maxAbilityLevel": 4,
      "rosterSlotCost": 5,
      "freeAbilities": ["Monster"],
      "epicAbilitySlots": 1,
      "extraAbilitySlotsFromRoster": 5
    }
  ],
  "abilities": {
    "level1": [],
    "level2": [],
    "level3": [],
    "level4": [],
    "epic": [],
    "gang": []
  },
  "perks": [],
  "associateAbilities": [],
  "weapons": { "ranged": [], "melee": [] },
  "genres": [],
  "secretSocieties": [],
  "reputationTable": []
}
```

- [ ] **Step 2: Populate all Level 1 abilities**

Add every Level 1 ability from the rulebook (pages 13-14 of the PDF). Each ability uses the effect types from the spec. Here is the complete list — add all of these to `abilities.level1`:

```json
[
  { "name": "Agile", "level": 1, "description": "Add +1 die to Dodge.", "effects": [{"type":"addDice","count":1,"target":"Dodge"}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Animal", "level": 1, "description": "Add +1 die to two skills. Reduce Shoot to no-dice.", "effects": [{"type":"addDice","count":1,"target":"pick2"},{"type":"setNoDice","target":["Shoot"]}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Aquatic", "level": 1, "description": "You automatically pass the perils for difficult and perilous areas in deep water. In the deep, your line-of-sight is up to 12\" and you ignore the -1 fight penalties.", "effects": [{"type":"narrative"}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Armored", "level": 1, "description": "You cannot move over 6\" except to rush. You roll one bonus die for all Health checks.", "effects": [{"type":"movementRestrict","maxInches":6},{"type":"healthBonus","count":1}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Beast", "level": 1, "description": "You cannot perform actions. Select two free abilities from: Animal, Aquatic, Big, Fierce, Mindless, Reanimated, Speedy, Swarm, Winged.", "effects": [{"type":"preventAction"},{"type":"grantFreeAbilities","count":2,"fromList":["Animal","Aquatic","Big","Fierce","Mindless","Reanimated","Speedy","Swarm","Winged"]}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Brainy", "level": 1, "description": "Add +1 die to Dodge and Cunning. Reduce Brawl to no-dice.", "effects": [{"type":"addDice","count":1,"target":"Dodge"},{"type":"addDice","count":1,"target":"Cunning"},{"type":"setNoDice","target":["Brawl"]}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Brute", "level": 1, "description": "Once per turn, you may re-roll one Brawl or Might die.", "effects": [{"type":"rerollSkill","skills":["Brawl","Might"],"perTurn":1}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Clever", "level": 1, "description": "Add +1 die to Cunning.", "effects": [{"type":"addDice","count":1,"target":"Cunning"}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Covert", "level": 1, "description": "When you are hidden, an opponent must win the opposed spotting check to spot you.", "effects": [{"type":"narrative"}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Crafty", "level": 1, "description": "Once per turn, you may re-roll one Dodge or Cunning die.", "effects": [{"type":"rerollSkill","skills":["Dodge","Cunning"],"perTurn":1}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Fierce", "level": 1, "description": "Add +1 die to Brawl.", "effects": [{"type":"addDice","count":1,"target":"Brawl"}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Goon", "level": 1, "description": "You cannot perform actions. Select two free abilities from: Aquatic, Brute, Fierce, Marksman, Sharp, Slam, Trick, Winged.", "effects": [{"type":"preventAction"},{"type":"grantFreeAbilities","count":2,"fromList":["Aquatic","Brute","Fierce","Marksman","Sharp","Slam","Trick","Winged"]}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Hard-Nosed", "level": 1, "description": "Once per turn, you may re-roll one Might, Finesse, or Cunning die.", "effects": [{"type":"rerollSkill","skills":["Might","Finesse","Cunning"],"perTurn":1}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Marksman", "level": 1, "description": "Add +1 die to Shoot.", "effects": [{"type":"addDice","count":1,"target":"Shoot"}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Mighty", "level": 1, "description": "Add +1 die to Might.", "effects": [{"type":"addDice","count":1,"target":"Might"}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Mindless", "level": 1, "description": "Raise Brawl and Might one dice-type. Reduce Cunning and Finesse one dice-type. If a d6 skill is reduced it drops to no-dice.", "effects": [{"type":"shiftUp","target":["Brawl","Might"]},{"type":"shiftDown","target":["Cunning","Finesse"]}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Monster", "level": 1, "description": "You are horrific (see Horror).", "effects": [{"type":"narrative"}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Mounted", "level": 1, "description": "You may start each scenario riding a basic mount.", "effects": [{"type":"narrative"}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Plan", "level": 1, "description": "Once per turn, you may discard to gain a +1 bonus to Dodge or Cunning.", "effects": [{"type":"narrative"}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Reanimated", "level": 1, "description": "You automatically pass all 1d Health checks. Reduce Dodge to no-dice. You cannot block. You cannot move over 6\" except to rush.", "effects": [{"type":"setNoDice","target":["Dodge"]},{"type":"movementRestrict","maxInches":6},{"type":"narrative"}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Savvy", "level": 1, "description": "Add +1 die to Finesse.", "effects": [{"type":"addDice","count":1,"target":"Finesse"}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Shadowy", "level": 1, "description": "You gain a +1 bonus to all opposed spotting checks.", "effects": [{"type":"narrative"}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Sharp", "level": 1, "description": "Once per turn, you may re-roll one Shoot or Finesse die.", "effects": [{"type":"rerollSkill","skills":["Shoot","Finesse"],"perTurn":1}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Short Burst", "level": 1, "description": "Action: Place the Short Burst (see Burst rules).", "effects": [{"type":"narrative"}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Slam", "level": 1, "description": "Once per turn, you may discard to gain a +1 bonus to Brawl or Might.", "effects": [{"type":"narrative"}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Sly", "level": 1, "description": "Add +1 die to Dodge and Finesse. Reduce Brawl to no-dice.", "effects": [{"type":"addDice","count":1,"target":"Dodge"},{"type":"addDice","count":1,"target":"Finesse"},{"type":"setNoDice","target":["Brawl"]}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Speedy", "level": 1, "description": "You may move up to 16\" instead of 12\".", "effects": [{"type":"narrative"}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Swarm", "level": 1, "description": "You may play a peril from your Fortune hand on an enemy when they come into contact or activate in contact with you.", "effects": [{"type":"narrative"}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Trick", "level": 1, "description": "Once per turn, you may discard to gain a +1 bonus to Shoot or Finesse.", "effects": [{"type":"narrative"}], "restrictions": [], "incompatibleWith": [] },
  { "name": "True Believer", "level": 1, "description": "(Followers, Allies, and Sidekicks only) Reduce Shoot to no-dice. When rolling for any other skill, you roll a number of dice equal to your character level. You ignore all modifiers except Fortune card effects. You can never have any other abilities.", "effects": [{"type":"setNoDice","target":["Shoot"]},{"type":"narrative"}], "restrictions": ["notLeader"], "incompatibleWith": ["all"] },
  { "name": "Two-Fisted", "level": 1, "description": "Add +1 die to two skills. Reduce Shoot to no-dice.", "effects": [{"type":"addDice","count":1,"target":"pick2"},{"type":"setNoDice","target":["Shoot"]}], "restrictions": [], "incompatibleWith": [] },
  { "name": "Winged", "level": 1, "description": "You may fly up to 16\" and ignore the intervening terrain and characters. You cannot fly and shoot during the same activation when you are at ground level.", "effects": [{"type":"narrative"}], "restrictions": [], "incompatibleWith": [] }
]
```

- [ ] **Step 3: Verify JSON is valid**

Run: `python -c "import json; json.load(open('BlitzTacticsBundle_1/PulpAlley/PulpAlley_Data.json')); print('Valid JSON')"`
Expected: "Valid JSON"

- [ ] **Step 4: Commit**

```bash
git add BlitzTacticsBundle_1/PulpAlley/PulpAlley_Data.json
git commit -m "feat(pulp-alley): add data JSON with character templates and Level 1 abilities"
```

---

## Task 2: Complete PulpAlley_Data.json — All Remaining Abilities

**Files:**
- Modify: `BlitzTacticsBundle_1/PulpAlley/PulpAlley_Data.json`

Add Level 2, 3, 4, Epic, and Gang abilities. Reference: spec pages 15-20 (Level 2-4), pages 31-32 (Epic), page 22 (Gang).

**Important:** The implementing agent MUST read the PDF rulebook (`BlitzTacticsBundle_1/Pulp_Alley_2E_Core_Rules_PDF.pdf`) pages 15-22 and 31-32 to get the exact description and mechanical effects for each ability. Use the Effect Types Reference table in the spec to determine the correct `effects` array for each ability. For abilities whose effects are purely gameplay/narrative (cannot be computed as stat modifications), use `{"type":"narrative"}`.

- [ ] **Step 1: Add all Level 2 abilities**

Add to `abilities.level2`. Complete list from the rulebook (pages 15-16):

Big, Long Burst, Burst Fire, Close Combat, Daredevil, Dithering, Doc, Drag, Eagle-Eyed, Faint of Heart, Harmless, Hindrance, Insight, Intrepid, Impetuous, Inventor, Noblesse, Relentless, Shapeshifter, Shock, Unearthly, Unlucky.

Follow the same JSON structure as Level 1. Key abilities with complex effects:
- **Dithering**: `"effects": [{"type":"grantFreeAbilities","count":2,"fromList":"level1"}], "incompatibleWith": ["preventAction","setNoDice"]`
- **Big**: `"effects": [{"type":"shiftHealthUp"},{"type":"shiftUp","target":["Might"]},{"type":"shiftDown","target":["Dodge","Finesse"]}]`
- **Harmless**: `"effects": [{"type":"setNoDice","target":["Brawl","Shoot"]},{"type":"narrative"}]`

- [ ] **Step 2: Add all Level 3 abilities**

Add to `abilities.level3`. Complete list from pages 17-18:

Aid, Bodyguard, Brash, Captain, Crackshot, Dashing, Deadeye, Deductive, Dread Gaze, Gadgeteer, Gearhead, Indomitable, Moxie, Muscles of Steel, Paralyzer, Quick-Dodge, Quick-Shot, Quick-Strike, Quick-Witted, Reach, Ruthless, Savage, Shrewd, Two-Guns, Veteran.

- [ ] **Step 3: Add all Level 4 abilities**

Add to `abilities.level4`. Complete list from pages 18-20:

Blaggard, Cloak & Dagger, Cloud Minds, Commander, Cursed Presence, Danger Sense, Dark Presence, Disarming, Drain Life, Extraordinary, Flying Tackle, Foul, Grappler, Hardboiled, Impervious, Inhuman, Inspiring, Intimidating, Iron Will, Lucky Devil, Loyal Following, Master of Disguise, Mindblast, Nerves of Steel, Rally, Regenerate, Rugged, Scoundrel, Summon, Tactician, Untouchable, Wealthy.

Key effects:
- **Commander**: `"effects": [{"type":"modifyRosterSlots","count":4,"restriction":"L1L2only"}]`
- **Extraordinary**: `"effects": [{"type":"narrative"}]` (raise d8 to d10 — user picks which skill)
- **Inhuman**: `"effects": [{"type":"narrative"}]` (raise d10 to d12 — user picks which skill)

- [ ] **Step 4: Add all Epic abilities**

Add to `abilities.epic`. Complete list from pages 31-32:

Blood Frenzy, Concentrate, Consume, Creeper, Dauntless, Double-Dealing, Immolate, Giant, Insidious, Juggernaut, Long Reach, Mend, Mesmerize, Monstrous, Predator, Protection, Rampage, Revive, Unspeakable.

- [ ] **Step 5: Add all Gang abilities**

Add to `abilities.gang`. Complete list from page 22:

Armed, Dangerous, Disciplined, Loyal, Mob, Sixth-Man.

Also note the "Other Abilities" allowed for Gangs (from page 22):
- Level 1: Animal, Aquatic, Beast, Brute, Crafty, Fierce, Goon, Marksman, Mindless, Reanimated, Sharp, Slam, Speedy, Swarm, Trick, Winged
- Level 2: Big, Eagle-Eyed, Relentless, True Believer, Unearthly

Store these as `gangAllowedAbilities` arrays in the gang ability entries or as a separate field.

- [ ] **Step 6: Verify JSON is valid**

Run: `python -c "import json; d=json.load(open('BlitzTacticsBundle_1/PulpAlley/PulpAlley_Data.json')); print(f'L1:{len(d[\"abilities\"][\"level1\"])} L2:{len(d[\"abilities\"][\"level2\"])} L3:{len(d[\"abilities\"][\"level3\"])} L4:{len(d[\"abilities\"][\"level4\"])} Epic:{len(d[\"abilities\"][\"epic\"])} Gang:{len(d[\"abilities\"][\"gang\"])}')"`

Expected: Counts matching the rulebook (~32 L1, ~22 L2, ~25 L3, ~32 L4, ~19 Epic, ~6 Gang).

- [ ] **Step 7: Commit**

```bash
git add BlitzTacticsBundle_1/PulpAlley/PulpAlley_Data.json
git commit -m "feat(pulp-alley): add Level 2-4, Epic, and Gang abilities to data JSON"
```

---

## Task 3: Complete PulpAlley_Data.json — Perks, Associates, Weapons

**Files:**
- Modify: `BlitzTacticsBundle_1/PulpAlley/PulpAlley_Data.json`

- [ ] **Step 1: Add all background perks**

Add to `perks` array. Complete list from pages 23-26. Each perk entry:
```json
{
  "name": "Dominion", "slotCost": 0, "category": "background",
  "description": "This perk grants access to the Minions, Gifts, and Cult assets. You cannot use your Resource points to select Backup, Contacts, or Gear assets.",
  "prerequisites": [], "restrictions": [],
  "leagueStructureOverride": null
}
```

Full list: Dominion (0), Altar (1), Amphibians (1), Base (1), Companions (1), Garage (1), Jack of All Trades (1), Keen Senses (1), Long Range (1), Mastermind (1), On the Run (1), Reanimated (1), Resourceful (1), Short Range (1), Specialists (1), Stable (1), Well Armed (1), Workshop (1), Ancient Sect (2), Animals (2), Bastion of Science (2), Call to Arms (2), Company of Heroes (2), Dark Pact (2), Flyers (2), Greater Purpose (2), Nefarious (2), Network of Supporters (2), Noblesse (2), Overlord (2), Riders (2), Shapeshifters (2), Skilled (2), Tenacious (2), Daredevil Adventurers (3), Eagle-Eyed Troopers (3), Intrepid Explorers (3), Shadowy Agents (3), Duo (4), League of Legends (10).

Mark league-restructuring perks with `leagueStructureOverride`: Mastermind, Companions, Specialists, Duo, League of Legends.

- [ ] **Step 2: Add all associate abilities**

Add to `associateAbilities` array. Complete list from page 28:

Exploit Weakness, Fortune's Favor, Friend of a Friend, Got the Goods, Got Your Back, In the Know, Infiltrate, Lucky Charm, Misdirection, Phantom Hand, Research & Rumors, Supplies, Tinker, Trickery & Tactics, Twist of Fate.

- [ ] **Step 3: Add weapons (Extended scope — include data now for completeness)**

Add to `weapons.ranged`: Pistol, SMG, Rifle, Shotgun, Primitive, Thrown.
Add to `weapons.melee`: Light, Basic, Heavy, Long, Grappling, Shield.

- [ ] **Step 4: Add secret societies, genres, reputation table**

Add to `secretSocieties`: The Society (I.R.E.S.), The Order (Oginali), The Guild (G13) — each with 4 ranks of 3 perk choices.

Add to `genres`: Pulp Fantasy, Pulp Westerns, Pulp War, Pulp Horror, Pulp Sci-Fi, Pulp Lost World.

Add to `reputationTable`: All entries from pages 85-86 (thresholds 10 through 175+).

- [ ] **Step 5: Verify JSON**

Run: `python -c "import json; d=json.load(open('BlitzTacticsBundle_1/PulpAlley/PulpAlley_Data.json')); print(f'Perks:{len(d[\"perks\"])} Assoc:{len(d[\"associateAbilities\"])} Societies:{len(d[\"secretSocieties\"])} Genres:{len(d[\"genres\"])} Reputation:{len(d[\"reputationTable\"])}')"`

- [ ] **Step 6: Commit**

```bash
git add BlitzTacticsBundle_1/PulpAlley/PulpAlley_Data.json
git commit -m "feat(pulp-alley): add perks, associates, weapons, societies, genres, reputation"
```

---

## Task 4: HTML Scaffold — CDN Imports, Theme CSS, App Shell

**Files:**
- Create: `BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html`

- [ ] **Step 1: Create the HTML boilerplate with CDN imports**

Follow the exact pattern from `BlitzTactics_Forge.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pulp Alley League Forge</title>
    <script>window.define = null;</script>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <!-- Pulp Dossier Theme Fonts -->
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Special+Elite&family=Playfair+Display:wght@400;700;900&family=Roboto+Slab:wght@400;700&display=swap');
    </style>
</head>
```

- [ ] **Step 2: Add the Pulp Dossier theme CSS**

Add the complete theme CSS using CSS custom properties. Include the full color palette from the spec, typography rules, print overrides, scrollbar styling, and toast animations. Reference the spec's "Visual Theme: Pulp Dossier" section for exact hex values.

Key CSS:
```css
:root {
    --paper: #d4c5a9;
    --paper-dark: #c4b48e;
    --ink: #2a1f0e;
    --ink-light: #5a4a2e;
    --ink-faded: #7a6a4e;
    --leather: #3b2f1e;
    --crimson: #8b1a1a;
    --border: #a89060;
}
body { font-family: 'Roboto Slab', serif; background: var(--leather); color: var(--ink); }
h1, h2, h3, .brand-font { font-family: 'Playfair Display', serif; }
.mono-font, .typewriter { font-family: 'Special Elite', cursive; }
/* Subtle paper texture via CSS gradient */
body::before {
    content: ''; position: fixed; inset: 0; z-index: -1; pointer-events: none;
    background: repeating-linear-gradient(
        0deg, transparent, transparent 2px, rgba(139,115,85,0.03) 2px, rgba(139,115,85,0.03) 4px
    );
}
/* Character level badge colors — extend palette for sidebar */
.badge-leader { color: var(--crimson); }
.badge-sidekick { color: #2a6e5a; }
.badge-ally { color: #8b6914; }
.badge-follower { color: var(--ink-faded); }
.badge-gang { color: var(--ink-light); }
/* Print overrides matching existing Forge pattern */
@media print {
    @page { margin: 0.5cm; size: auto; }
    html, body { background: white !important; color: black !important; }
    .no-print { display: none !important; }
    .print-only { display: block !important; }
    .print-card { break-inside: avoid; page-break-inside: avoid; }
}
.print-only { display: none; }
```

- [ ] **Step 3: Add the body, root div, and minimal React app**

```html
<body class="h-screen overflow-hidden">
    <div id="root" class="h-full"></div>
    <script type="text/babel">
        const { useState, useRef, useEffect, useCallback } = React;

        // Section 1: ICONS & CONSTANTS
        const createIcon = (path) => ({ size = 24, className = "", strokeWidth = 2, ...props }) => (
            <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24"
                fill="none" stroke="currentColor" strokeWidth={strokeWidth}
                strokeLinecap="round" strokeLinejoin="round" className={className} {...props}>
                {path}
            </svg>
        );
        // Add icons: Plus, Trash2, Save, Printer, Upload, Download, X, Edit2, Search, ChevronDown, AlertTriangle, Image, Users, Shield, Star
        const Icons = {
            Plus: createIcon(<><path d="M5 12h14"/><path d="M12 5v14"/></>),
            Trash2: createIcon(<><path d="M3 6h18"/><path d="M19 6v14c0 1-1 2-2 2H7c-1 0-2-1-2-2V6"/><path d="M8 6V4c0-1 1-2 2-2h4c1 0 2 1 2 2v2"/></>),
            Save: createIcon(<><path d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11l5 5v11a2 2 0 0 1-2 2z"/><polyline points="17 21 17 13 7 13 7 21"/><polyline points="7 3 7 8 15 8"/></>),
            Printer: createIcon(<><polyline points="6 9 6 2 18 2 18 9"/><path d="M6 18H4a2 2 0 0 1-2-2v-5a2 2 0 0 1 2-2h16a2 2 0 0 1 2 2v5a2 2 0 0 1-2 2h-2"/><rect width="12" height="8" x="6" y="14"/></>),
            Upload: createIcon(<><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" x2="12" y1="3" y2="15"/></>),
            Download: createIcon(<><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" x2="12" y1="15" y2="3"/></>),
            X: createIcon(<><path d="M18 6 6 18"/><path d="m6 6 12 12"/></>),
            Edit2: createIcon(<path d="M17 3a2.83 2.83 0 1 1 4 4L7.5 20.5 2 22l1.5-5.5L17 3z"/>),
            Search: createIcon(<><circle cx="11" cy="11" r="8"/><path d="m21 21-4.3-4.3"/></>),
            AlertTriangle: createIcon(<><path d="m21.73 18-8-14a2 2 0 0 0-3.48 0l-8 14A2 2 0 0 0 4 21h16a2 2 0 0 0 1.73-3Z"/><line x1="12" x2="12" y1="9" y2="13"/><line x1="12" x2="12.01" y1="17" y2="17"/></>),
            Image: createIcon(<><rect width="18" height="18" x="3" y="3" rx="2" ry="2"/><circle cx="9" cy="9" r="2"/><path d="m21 15-3.09-3.09a2 2 0 0 0-2.82 0L6 21"/></>),
            Users: createIcon(<><path d="M16 21v-2a4 4 0 0 0-4-4H6a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M22 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/></>),
        };

        const SKILLS = ['Brawl', 'Shoot', 'Dodge', 'Might', 'Finesse', 'Cunning'];
        const DICE_TYPES = ['d6', 'd8', 'd10', 'd12'];
        const DICE_ORDER = { 'd6': 0, 'd8': 1, 'd10': 2, 'd12': 3 };

        // Section 7: APP INIT — minimal for now
        const App = () => {
            const [gameData, setGameData] = useState(null);
            return gameData ? (
                <div className="h-full" style={{background:'var(--paper)'}}>
                    <div className="text-center p-8 typewriter" style={{color:'var(--ink)'}}>
                        Data loaded! {gameData.abilities.level1.length} Level 1 abilities found.
                    </div>
                </div>
            ) : (
                <DataLoader onLoad={setGameData} />
            );
        };

        // Minimal DataLoader — will be expanded in Task 5
        const DataLoader = ({ onLoad }) => {
            const fileRef = useRef(null);
            const handleFile = (e) => {
                const file = e.target.files[0];
                if (!file) return;
                const reader = new FileReader();
                reader.onload = (ev) => {
                    try {
                        const data = JSON.parse(ev.target.result);
                        try { localStorage.setItem('pulpAlley_dataCache', ev.target.result); }
                        catch (storageErr) { console.warn('Could not cache data in LocalStorage:', storageErr); }
                        onLoad(data);
                    } catch (err) { alert('Invalid JSON file'); }
                };
                reader.readAsText(file);
            };
            // Check cache on mount
            useEffect(() => {
                const cached = localStorage.getItem('pulpAlley_dataCache');
                if (cached) { try { onLoad(JSON.parse(cached)); } catch(e) {} }
            }, []);
            return (
                <div className="h-full flex items-center justify-center" style={{background:'var(--leather)'}}>
                    <div className="text-center p-12 rounded-lg" style={{background:'var(--paper)',border:'2px solid var(--border)'}}>
                        <h1 className="text-3xl font-bold brand-font mb-4" style={{color:'var(--crimson)'}}>PULP ALLEY FORGE</h1>
                        <p className="typewriter mb-6" style={{color:'var(--ink-light)'}}>Load your PulpAlley_Data.json to begin</p>
                        <input ref={fileRef} type="file" accept=".json" onChange={handleFile} className="hidden" />
                        <button onClick={() => fileRef.current.click()}
                            className="px-6 py-3 rounded typewriter text-lg cursor-pointer"
                            style={{background:'var(--crimson)',color:'var(--paper)',border:'none'}}>
                            Load Data File
                        </button>
                    </div>
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
```

- [ ] **Step 4: Verify in browser**

Open `BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html` in a browser. Expected:
1. See the "PULP ALLEY FORGE" loading screen with Pulp Dossier theme
2. Click "Load Data File" and select `PulpAlley_Data.json`
3. See "Data loaded! 32 Level 1 abilities found." (or similar count)
4. Refresh — data should auto-load from cache

- [ ] **Step 5: Commit**

```bash
git add BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add HTML scaffold with CDN imports, Pulp Dossier theme, data loader"
```

---

## Task 5: Persistence Layer — LocalStorage Save/Load, UUID, League State

**Files:**
- Modify: `BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html` (Section 2: DATA & PERSISTENCE)

- [ ] **Step 1: Add persistence utility functions**

Insert after the ICONS & CONSTANTS section. These are pure functions with no UI:

```javascript
// Section 2: DATA & PERSISTENCE
const generateUUID = () => crypto.randomUUID ? crypto.randomUUID() :
    'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, c => {
        const r = Math.random() * 16 | 0;
        return (c === 'x' ? r : (r & 0x3 | 0x8)).toString(16);
    });

const EMPTY_CHARACTER = (type, template) => ({
    id: generateUUID(),
    name: '',
    type: type,
    level: template.level,
    image: null,
    skillAssignments: { Brawl: {count:null,diceType:null}, Shoot: {count:null,diceType:null}, Dodge: {count:null,diceType:null}, Might: {count:null,diceType:null}, Finesse: {count:null,diceType:null}, Cunning: {count:null,diceType:null} },
    abilities: [],
    abilityFlavourNames: {},
    abilityChoices: {},  // for pick2 targets etc
    freeAbilities: [],   // from Beast/Goon/Dithering
    gangModels: type === 'gang' ? 5 : null,
});

const EMPTY_ASSOCIATE = () => ({
    id: generateUUID(),
    name: '',
    abilities: [],  // array of 2 associate ability name strings
});

const EMPTY_LEAGUE = () => ({
    id: generateUUID(),
    name: 'New League',
    characters: [],
    perks: [],
    associates: [],  // array of EMPTY_ASSOCIATE objects — each costs 1 roster slot per rulebook p.27
    campaignState: { reputation: 0, xp: 0, resources: { tips: 0, backup: 0, gear: 0, contacts: 0 }, secretSociety: null, secretSocietyPerks: [] },
    genre: null,
    optionalRules: { weapons: false, harrowing: false, supers: false },
});

const LEAGUE_INDEX_KEY = 'pulpAlley_leagueIndex';
const LEAGUE_PREFIX = 'pulpAlley_league_';
const DATA_CACHE_KEY = 'pulpAlley_dataCache';

const loadLeagueIndex = () => {
    try { return JSON.parse(localStorage.getItem(LEAGUE_INDEX_KEY)) || []; }
    catch { return []; }
};
const saveLeagueIndex = (index) => localStorage.setItem(LEAGUE_INDEX_KEY, JSON.stringify(index));

const loadLeague = (id) => {
    try { return JSON.parse(localStorage.getItem(LEAGUE_PREFIX + id)); }
    catch { return null; }
};

const saveLeague = (league) => {
    try {
        localStorage.setItem(LEAGUE_PREFIX + league.id, JSON.stringify(league));
        const index = loadLeagueIndex();
        const existing = index.findIndex(e => e.id === league.id);
        const entry = { id: league.id, name: league.name, lastModified: Date.now() };
        if (existing >= 0) index[existing] = entry; else index.push(entry);
        saveLeagueIndex(index);
        return true;
    } catch (e) {
        console.error('Storage save failed:', e);
        return false;
    }
};

const deleteLeague = (id) => {
    localStorage.removeItem(LEAGUE_PREFIX + id);
    const index = loadLeagueIndex().filter(e => e.id !== id);
    saveLeagueIndex(index);
};

const exportLeagueJSON = (league) => {
    const blob = new Blob([JSON.stringify(league, null, 2)], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url; a.download = `${league.name.replace(/[^a-zA-Z0-9]/g, '_')}_league.json`;
    a.click(); URL.revokeObjectURL(url);
};

const importLeagueJSON = (file) => new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = (e) => { try { resolve(JSON.parse(e.target.result)); } catch { reject('Invalid JSON'); } };
    reader.readAsText(file);
});

const resizeImage = (file, maxSize = 128) => new Promise((resolve) => {
    const reader = new FileReader();
    reader.onload = (e) => {
        const img = new Image();
        img.onload = () => {
            const canvas = document.createElement('canvas');
            const scale = Math.min(maxSize / img.width, maxSize / img.height, 1);
            canvas.width = img.width * scale;
            canvas.height = img.height * scale;
            canvas.getContext('2d').drawImage(img, 0, 0, canvas.width, canvas.height);
            resolve(canvas.toDataURL('image/jpeg', 0.7));
        };
        img.src = e.target.result;
    };
    reader.readAsDataURL(file);
});
```

- [ ] **Step 2: Update App component to manage league state**

Replace the minimal `App` component with one that manages league state, auto-saves, and shows a league selector or the main editor:

```javascript
const App = () => {
    const [gameData, setGameData] = useState(null);
    const [league, setLeague] = useState(null);
    const [activeView, setActiveView] = useState('roster'); // roster, character, perks, associates
    const [selectedCharId, setSelectedCharId] = useState(null);
    const [toast, setToast] = useState(null);

    const showToast = (msg, type='info') => { setToast({msg,type}); setTimeout(() => setToast(null), 3000); };

    // Auto-save on league change
    useEffect(() => {
        if (league) {
            if (!saveLeague(league)) showToast('Storage full — export as JSON to free space', 'error');
        }
    }, [league]);

    const updateLeague = useCallback((updater) => {
        setLeague(prev => typeof updater === 'function' ? updater(prev) : updater);
    }, []);

    if (!gameData) return <DataLoader onLoad={setGameData} />;
    if (!league) return <LeagueSelector gameData={gameData} onSelect={setLeague} onCreate={() => { const l = EMPTY_LEAGUE(); setLeague(l); }} />;

    return (
        <div className="h-full flex flex-col" style={{background:'var(--paper)'}}>
            {/* TopBar placeholder */}
            <div className="p-2 text-center typewriter" style={{background:'var(--leather)',color:'var(--paper)'}}>
                {league.name} — {league.characters.length} characters
            </div>
            <div className="flex-1 overflow-hidden flex">
                {/* Sidebar placeholder */}
                <div className="w-60 overflow-y-auto p-3" style={{background:'var(--paper-dark)',borderRight:'2px solid var(--border)'}}>
                    <div className="typewriter text-xs uppercase tracking-wider mb-2" style={{color:'var(--crimson)'}}>Roster</div>
                    <p className="text-sm" style={{color:'var(--ink-faded)'}}>Sidebar will go here</p>
                </div>
                {/* Center panel placeholder */}
                <div className="flex-1 overflow-y-auto p-6">
                    <p style={{color:'var(--ink)'}}>Center panel — {activeView}</p>
                </div>
            </div>
            {toast && <div className="fixed top-4 right-4 px-4 py-2 rounded shadow-lg typewriter text-sm" style={{background: toast.type==='error'?'var(--crimson)':'var(--leather)', color:'var(--paper)'}}>{toast.msg}</div>}
        </div>
    );
};
```

- [ ] **Step 3: Add LeagueSelector component**

A simple screen that lists saved leagues and offers "New League" / "Import" options:

```javascript
const LeagueSelector = ({ gameData, onSelect, onCreate }) => {
    const [leagues] = useState(loadLeagueIndex);
    const importRef = useRef(null);
    const handleImport = async (e) => {
        const file = e.target.files[0]; if (!file) return;
        try {
            const league = await importLeagueJSON(file);
            if (!league.id) league.id = generateUUID();
            saveLeague(league);
            onSelect(league);
        } catch { alert('Invalid league file'); }
    };
    return (
        <div className="h-full flex items-center justify-center" style={{background:'var(--leather)'}}>
            <div className="p-8 rounded-lg max-w-md w-full" style={{background:'var(--paper)',border:'2px solid var(--border)'}}>
                <h1 className="text-2xl font-bold brand-font mb-6 text-center" style={{color:'var(--crimson)'}}>LEAGUE DOSSIERS</h1>
                {leagues.length > 0 && (
                    <div className="mb-6">
                        {leagues.sort((a,b) => b.lastModified - a.lastModified).map(entry => (
                            <button key={entry.id} onClick={() => onSelect(loadLeague(entry.id))}
                                className="w-full text-left p-3 mb-2 rounded cursor-pointer"
                                style={{background:'var(--paper-dark)',border:'1px solid var(--border)'}}>
                                <div className="font-bold brand-font" style={{color:'var(--ink)'}}>{entry.name}</div>
                                <div className="text-xs typewriter" style={{color:'var(--ink-faded)'}}>
                                    Last modified: {new Date(entry.lastModified).toLocaleDateString()}
                                </div>
                            </button>
                        ))}
                    </div>
                )}
                <div className="flex gap-3">
                    <button onClick={onCreate} className="flex-1 px-4 py-3 rounded typewriter cursor-pointer"
                        style={{background:'var(--crimson)',color:'var(--paper)',border:'none'}}>New League</button>
                    <input ref={importRef} type="file" accept=".json" onChange={handleImport} className="hidden" />
                    <button onClick={() => importRef.current.click()} className="flex-1 px-4 py-3 rounded typewriter cursor-pointer"
                        style={{background:'var(--leather)',color:'var(--paper)',border:'none'}}>Import</button>
                </div>
            </div>
        </div>
    );
};
```

- [ ] **Step 4: Verify in browser**

Open the HTML. Expected:
1. Data loads from cache (or load it)
2. See the "LEAGUE DOSSIERS" screen
3. Click "New League" — see the basic three-zone layout with placeholder sidebar and center panel
4. Refresh — league reloads from LocalStorage automatically

- [ ] **Step 5: Commit**

```bash
git add BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add persistence layer, league state management, league selector"
```

---

## Task 6: Validation Engine — Pure Logic Functions

**Files:**
- Modify: `BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html` (Section 3: VALIDATION ENGINE)

- [ ] **Step 1: Add core validation functions**

Insert after the persistence section. These are pure functions that take data and return validation results — no UI:

```javascript
// Section 3: VALIDATION ENGINE

const getTemplate = (gameData, type) => gameData.characterTemplates.find(t => t.type === type);

const getAllAbilities = (gameData) => [
    ...gameData.abilities.level1,
    ...gameData.abilities.level2,
    ...gameData.abilities.level3,
    ...gameData.abilities.level4,
    ...(gameData.abilities.gang || []),
];

const getAbilitiesForLevel = (gameData, maxLevel) => {
    const all = [];
    if (maxLevel >= 1) all.push(...gameData.abilities.level1);
    if (maxLevel >= 2) all.push(...gameData.abilities.level2);
    if (maxLevel >= 3) all.push(...gameData.abilities.level3);
    if (maxLevel >= 4) all.push(...gameData.abilities.level4);
    return all;
};

const getTotalSlots = (league, gameData) => {
    let slots = 10;
    // Commander ability adds +4 slots
    league.characters.forEach(c => {
        const allAbilities = [...c.abilities, ...c.freeAbilities];
        allAbilities.forEach(aName => {
            const ability = getAllAbilities(gameData).find(a => a.name === aName);
            if (ability) ability.effects.forEach(eff => {
                if (eff.type === 'modifyRosterSlots') slots += eff.count;
            });
        });
    });
    // Reputation perks that add slots
    if (league.campaignState) {
        const rep = league.campaignState.reputation || 0;
        gameData.reputationTable.forEach(entry => {
            if (rep >= entry.threshold && entry.effect && entry.effect.type === 'modifyRosterSlots') {
                slots += entry.effect.count;
            }
        });
    }
    return slots;
};

const getUsedSlots = (league, gameData) => {
    let used = 0;
    league.characters.forEach(c => {
        const tmpl = getTemplate(gameData, c.type);
        if (tmpl) used += tmpl.rosterSlotCost;
    });
    league.perks.forEach(pName => {
        const perk = gameData.perks.find(p => p.name === pName);
        if (perk) used += perk.slotCost;
    });
    used += league.associates.length; // 1 slot each
    return used;
};

const getMaxSidekicks = (league) => {
    if (league.perks.includes('Company of Heroes')) return 2;
    return 1;
};

const getMaxAssociates = (league) => {
    let max = 2;
    const rep = league.campaignState?.reputation || 0;
    if (rep >= 50) max = 3;
    if (rep >= 75) max = 4;
    return max;
};

const validateAbilityForCharacter = (abilityName, character, gameData) => {
    const errors = [];
    const ability = getAllAbilities(gameData).find(a => a.name === abilityName);
    if (!ability) return ['Ability not found'];

    // Level restriction
    if (ability.level > character.level) errors.push(`Requires level ${ability.level}+`);

    // No duplicates
    if (character.abilities.includes(abilityName)) errors.push('Already selected');

    // True Believer blocks all others
    if (character.abilities.includes('True Believer') && abilityName !== 'True Believer')
        errors.push('True Believer prevents other abilities');
    if (abilityName === 'True Believer' && character.abilities.length > 0)
        errors.push('True Believer must be the only ability');
    if (abilityName === 'True Believer' && character.type === 'leader')
        errors.push('Not available to Leaders');

    // Check no-dice conflicts: max 1 ability reducing same skill to no-dice
    const existingNoDice = new Set();
    character.abilities.forEach(aName => {
        const a = getAllAbilities(gameData).find(ab => ab.name === aName);
        if (a) a.effects.filter(e => e.type === 'setNoDice').forEach(e => e.target.forEach(s => existingNoDice.add(s)));
    });
    const newNoDice = ability.effects.filter(e => e.type === 'setNoDice').flatMap(e => e.target);
    newNoDice.forEach(skill => {
        if (existingNoDice.has(skill)) errors.push(`Already has an ability reducing ${skill} to no-dice`);
    });

    // Check no-action conflicts: max 1 preventAction ability
    const hasPreventAction = character.abilities.some(aName => {
        const a = getAllAbilities(gameData).find(ab => ab.name === aName);
        return a && a.effects.some(e => e.type === 'preventAction');
    });
    if (hasPreventAction && ability.effects.some(e => e.type === 'preventAction'))
        errors.push('Already has an ability preventing actions');

    // Incompatibility checks
    if (ability.incompatibleWith && ability.incompatibleWith.length > 0) {
        if (ability.incompatibleWith.includes('all') && character.abilities.length > 0)
            errors.push('Incompatible with all other abilities');
        // Check specific named incompatibilities
        ability.incompatibleWith.filter(x => x !== 'all' && x !== 'preventAction' && x !== 'setNoDice').forEach(inc => {
            if (character.abilities.includes(inc)) errors.push(`Incompatible with ${inc}`);
        });
        // Dithering/Faint of Heart/Hindrance/Impetuous/Unlucky: incompatible with preventAction and setNoDice abilities
        if (ability.incompatibleWith.includes('preventAction') && hasPreventAction)
            errors.push('Incompatible with no-action abilities');
        if (ability.incompatibleWith.includes('setNoDice') && existingNoDice.size > 0)
            errors.push('Incompatible with no-dice abilities');
    }

    return errors;
};

const validateLeague = (league, gameData) => {
    const errors = [];
    const totalSlots = getTotalSlots(league, gameData);
    const usedSlots = getUsedSlots(league, gameData);
    if (usedSlots > totalSlots) errors.push(`Over slot limit: ${usedSlots}/${totalSlots}`);

    // Must have exactly 1 leader (unless Mastermind or League of Legends)
    const leaders = league.characters.filter(c => c.type === 'leader');
    const hasMastermind = league.perks.includes('Mastermind');
    const hasLoL = league.perks.includes('League of Legends');
    if (!hasMastermind && !hasLoL && leaders.length !== 1) errors.push('League must have exactly 1 Leader');

    // Sidekick limit
    const sidekicks = league.characters.filter(c => c.type === 'sidekick');
    const maxSK = getMaxSidekicks(league);
    if (sidekicks.length > maxSK) errors.push(`Max ${maxSK} Sidekick(s)`);

    // Associate limit
    if (league.associates.length > getMaxAssociates(league))
        errors.push(`Max ${getMaxAssociates(league)} Associates`);

    // Duo restrictions
    if (league.perks.includes('Duo')) {
        const nonLeaderSidekick = league.characters.filter(c => c.type !== 'leader' && c.type !== 'sidekick');
        if (nonLeaderSidekick.length > 0) errors.push('Duo: only Leader and Sidekick allowed');
    }

    // Companions: no sidekick
    if (league.perks.includes('Companions')) {
        if (sidekicks.length > 0) errors.push('Companions: cannot include a Sidekick');
    }

    // Mastermind: +6 slots for L1/L2 only, no Leader character
    if (hasMastermind) {
        if (leaders.length > 0) errors.push('Mastermind: league has no Leader character');
        const nonL1L2 = league.characters.filter(c => c.type !== 'sidekick' && c.level > 2);
        // Mastermind gets a free Sidekick (fills Leader role) + 6 extra slots for L1/L2
        // Note: the free sidekick and +6 slots are handled in getTotalSlots below
    }

    // League of Legends: 4 sidekicks, no leader
    if (hasLoL) {
        if (leaders.length > 0) errors.push('League of Legends: no Leader character');
        if (sidekicks.length > 4) errors.push('League of Legends: max 4 Sidekicks');
    }

    // Specialists: validated in skill computation (shift dice types), not roster structure

    return errors;
};

// NOTE: League structure overrides (Mastermind free Sidekick, Mastermind +6 L1/L2 slots,
// League of Legends 4 included Sidekicks, Specialists dice-type shifts, Duo Health/ability
// upgrades) are partially implemented in validation. Full enforcement of their slot math
// and character template modifications will be refined during Task 14 (polish) based on
// integration testing in Task 15. The perks are selectable and their basic constraints
// (no Leader, no Sidekick, etc.) are enforced, but edge cases in slot computation for
// these special structures may need per-structure logic in getTotalSlots.
```

- [ ] **Step 2: Commit**

```bash
git add BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add validation engine with league and ability constraint checking"
```

---

## Task 7: Skill Computation Engine

**Files:**
- Modify: `BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html` (Section 4: SKILL COMPUTATION)

- [ ] **Step 1: Add skill computation functions**

These compute the final skill values for a character after applying all ability effects:

```javascript
// Section 4: SKILL COMPUTATION

const shiftDiceType = (diceType, direction) => {
    const idx = DICE_ORDER[diceType];
    if (idx === undefined) return diceType;
    const newIdx = idx + direction;
    if (newIdx < 0) return 'no-dice';
    if (newIdx >= DICE_TYPES.length) return DICE_TYPES[DICE_TYPES.length - 1];
    return DICE_TYPES[newIdx];
};

const computeCharacterSkills = (character, gameData) => {
    const template = getTemplate(gameData, character.type);
    if (!template) return null;

    // For gangs, use special rules
    if (character.type === 'gang') return computeGangSkills(character, gameData);

    // Start with base assignments
    const skills = {};
    SKILLS.forEach(skill => {
        const assignment = character.skillAssignments[skill];
        skills[skill] = {
            baseCount: assignment?.count || 0,
            baseDiceType: assignment?.diceType || 'd6',
            bonusCount: 0,
            diceTypeShift: 0,
            noDice: false,
            annotations: [],
        };
    });

    // Apply ability effects
    const allAbilityNames = [...character.abilities, ...character.freeAbilities];
    allAbilityNames.forEach(abilityName => {
        const ability = [...getAllAbilities(gameData), ...(gameData.abilities.epic || [])].find(a => a.name === abilityName);
        if (!ability) return;

        ability.effects.forEach(effect => {
            if (effect.type === 'addDice') {
                const targets = resolveTargets(effect.target, character, abilityName);
                targets.forEach(skill => {
                    if (skills[skill]) {
                        skills[skill].bonusCount += effect.count;
                        skills[skill].annotations.push(`+${effect.count} from ${abilityName}`);
                    }
                });
            }
            if (effect.type === 'setNoDice') {
                effect.target.forEach(skill => {
                    if (skills[skill]) {
                        skills[skill].noDice = true;
                        skills[skill].annotations.push(`No-dice from ${abilityName}`);
                    }
                });
            }
            if (effect.type === 'shiftUp') {
                effect.target.forEach(skill => {
                    if (skills[skill]) {
                        skills[skill].diceTypeShift += 1;
                        skills[skill].annotations.push(`Shift up from ${abilityName}`);
                    }
                });
            }
            if (effect.type === 'shiftDown') {
                effect.target.forEach(skill => {
                    if (skills[skill]) {
                        skills[skill].diceTypeShift -= 1;
                        skills[skill].annotations.push(`Shift down from ${abilityName}`);
                    }
                });
            }
        });
    });

    // Compute finals
    SKILLS.forEach(skill => {
        const s = skills[skill];
        if (s.noDice) {
            s.finalCount = 0;
            s.finalDiceType = 'no-dice';
            s.display = '—';
        } else {
            s.finalCount = s.baseCount + s.bonusCount;
            const shifted = shiftDiceType(s.baseDiceType, s.diceTypeShift);
            if (shifted === 'no-dice') {
                s.finalDiceType = 'no-dice';
                s.finalCount = 0;
                s.display = '—';
            } else {
                s.finalDiceType = shifted;
                s.display = `${s.finalCount}${s.finalDiceType}`;
            }
        }
    });

    return skills;
};

const resolveTargets = (target, character, abilityName) => {
    if (typeof target === 'string') {
        if (target.startsWith('pick')) {
            // Look up user choice in character.abilityChoices
            return character.abilityChoices[abilityName] || [];
        }
        return [target];
    }
    if (Array.isArray(target)) return target;
    return [];
};

const computeGangSkills = (character, gameData) => {
    const models = character.gangModels || 5;
    const combatDice = Math.ceil(models / 2);
    const skills = {};
    SKILLS.forEach(skill => {
        const isCombat = ['Brawl', 'Shoot', 'Might'].includes(skill);
        skills[skill] = {
            baseCount: isCombat ? combatDice : 1,
            baseDiceType: 'd6',
            bonusCount: 0, diceTypeShift: 0, noDice: false, annotations: [],
        };
    });

    // Apply ability effects (gang abilities like Big can shift dice types)
    const allAbilityNames = [...character.abilities, ...character.freeAbilities];
    allAbilityNames.forEach(abilityName => {
        const ability = getAllAbilities(gameData).find(a => a.name === abilityName);
        if (!ability) return;
        ability.effects.forEach(effect => {
            if (effect.type === 'shiftUp') effect.target.forEach(s => { if (skills[s]) { skills[s].diceTypeShift += 1; skills[s].annotations.push(`Shift up from ${abilityName}`); }});
            if (effect.type === 'shiftDown') effect.target.forEach(s => { if (skills[s]) { skills[s].diceTypeShift -= 1; skills[s].annotations.push(`Shift down from ${abilityName}`); }});
            if (effect.type === 'addDice') {
                const targets = resolveTargets(effect.target, character, abilityName);
                targets.forEach(s => { if (skills[s]) { skills[s].bonusCount += effect.count; skills[s].annotations.push(`+${effect.count} from ${abilityName}`); }});
            }
            if (effect.type === 'setNoDice') effect.target.forEach(s => { if (skills[s]) { skills[s].noDice = true; skills[s].annotations.push(`No-dice from ${abilityName}`); }});
        });
    });

    // Compute finals
    SKILLS.forEach(skill => {
        const s = skills[skill];
        if (s.noDice) {
            s.finalCount = 0; s.finalDiceType = 'no-dice'; s.display = '—';
        } else {
            s.finalCount = s.baseCount + s.bonusCount;
            const shifted = shiftDiceType(s.baseDiceType, s.diceTypeShift);
            if (shifted === 'no-dice') { s.finalDiceType = 'no-dice'; s.finalCount = 0; s.display = '—'; }
            else { s.finalDiceType = shifted; s.display = `${s.finalCount}${s.finalDiceType}`; }
        }
    });
    return skills;
};
```

- [ ] **Step 2: Commit**

```bash
git add BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add skill computation engine with ability effect application"
```

---

## Task 8: TopBar + RosterSidebar Components

**Files:**
- Modify: `BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html` (Section 5a-5c)

- [ ] **Step 1: Build TopBar component**

Displays league name (editable), slot counter, and action buttons (Save, Export, Import, Print, Back to League List):

The TopBar should use `var(--leather)` background, `var(--crimson)` for the "PULP ALLEY — LEAGUE DOSSIER" title, and a CLASSIFIED stamp. League name is an inline-editable input. Show buttons for Save, Export, Print, and a back arrow to return to the league selector.

- [ ] **Step 2: Build RosterSidebar component**

Left sidebar showing all characters as compact cards. Each card shows:
- Level badge (color: crimson for Leader, teal for Sidekick, amber for Ally, grey for Follower)
- Character name
- Mini thumbnail if image exists
- Health die type
- Ability count
- Slot cost

Below the character list: Perks section, Associates section, and "+ Add" button.

Clicking a character card sets `selectedCharId` and switches `activeView` to `'character'`. The "+ Add" button opens the AddCharacterModal.

- [ ] **Step 3: Build AddCharacterModal**

A modal overlay letting the user pick what to add: Sidekick, Ally, Follower, Gang, Perk, or Associate. Each option shows its slot cost and whether it's available (greyed out if no slots or at limit). Selecting one creates the appropriate empty entity and opens its editor.

Include the slot availability check: disable Sidekick if at max, disable options that would exceed slot limit.

- [ ] **Step 4: Update App component to wire everything together**

Replace the placeholder sidebar and topbar with the real components. Wire up the state management: clicking a sidebar card opens the character editor, the "+" button opens the modal, etc.

- [ ] **Step 5: Verify in browser**

Expected:
1. Create a new league — see the TopBar with "New League" and the empty sidebar
2. Click "+ Add" — see the modal with character type options
3. Add a Leader — see it appear in the sidebar with crimson badge
4. Edit the league name in the TopBar — auto-saves

- [ ] **Step 6: Commit**

```bash
git add BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add TopBar, RosterSidebar, and AddCharacterModal components"
```

---

## Task 9: CharacterEditor — Skill Assignment Grid

**Files:**
- Modify: `BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html` (Section 5d, first half)

- [ ] **Step 1: Build the SkillAssignmentGrid component**

This is the core UI for assigning skill dice. For each of the 6 skills, display a row with:
- Skill name (typewriter font)
- Dice count dropdown (pool-depletion: selecting "3" for one skill removes it from others)
- Dice type dropdown (same pool-depletion mechanic)
- Final computed value (teal, bold, shows ability bonuses)

Implementation approach:
- Track `remainingCounts` and `remainingTypes` as derived state from the character's `skillAssignments`
- Each dropdown shows only values still in the pool (plus the currently selected value for that row)
- For Follower (all 1d6), render as read-only text instead of dropdowns since there is no choice
- For Ally, the dice **type** column is read-only (all d6), but the dice **count** dropdowns still require user assignment (which 2 skills get 2d vs which 4 get 1d)
- Show "Remaining: 2×3d, 1×2d" above the grid to help users track
- When all assignments are made, show a green checkmark

- [ ] **Step 2: Build the character header area**

Above the skill grid: character name input, type badge, health die display, and the mini image upload area. The image area is a clickable square that shows a placeholder icon or the uploaded thumbnail. Clicking opens a file picker, image is resized via `resizeImage()`.

- [ ] **Step 3: Wire CharacterEditor into the center panel**

When `activeView === 'character'` and `selectedCharId` is set, render the CharacterEditor. It receives the current character from `league.characters` and an update callback.

- [ ] **Step 4: Verify in browser**

Expected:
1. Add a Leader → opens character editor
2. See 6 skill rows with dropdown selectors
3. Assign: 4 skills to "3" dice, 2 to "2" dice — pool depletes correctly
4. Assign: 4 skills to "d10", 2 to "d8" — pool depletes correctly
5. Final column shows computed values like "3d10"
6. Upload a mini image — see thumbnail appear

- [ ] **Step 5: Commit**

```bash
git add BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add CharacterEditor with skill assignment grid and image upload"
```

---

## Task 10: CharacterEditor — Ability Picker

**Files:**
- Modify: `BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html` (Section 5d, second half)

- [ ] **Step 1: Build the AbilityPicker component**

Below the skill grid in the character editor. Shows:
- Currently selected abilities as cards (with remove button, flavor name input, description, level badge)
- "Add Ability" button that opens a filterable list
- Ability count indicator (e.g., "2/3 abilities")

The ability list:
- Filtered by character's max level
- Search/filter by name or description text
- Each ability shows: name, level, description, and a status indicator
- Incompatible abilities are greyed out with the reason shown in a tooltip (from `validateAbilityForCharacter`)
- Clicking an available ability adds it to the character

For abilities with `pick2` targets (Animal, Two-Fisted), after selecting the ability, show a follow-up prompt to pick which 2 skills get the +1 die bonus. Store choices in `character.abilityChoices[abilityName]`.

For abilities with `grantFreeAbilities` (Beast, Goon, Dithering), after selecting, show a sub-picker for the free ability choices. Store in `character.freeAbilities`.

- [ ] **Step 2: Add flavor name editing**

Each selected ability card has an italic text input for the flavor name. Default is the ability's real name. Changes are stored in `character.abilityFlavourNames`. The real ability name is always shown in small text below.

- [ ] **Step 3: Verify skill grid updates live when abilities change**

When an ability like "Marksman" (+1 Shoot) is added, the skill grid's final column should immediately update to reflect the bonus. The annotation "Shoot: +1 from Marksman" should appear.

- [ ] **Step 4: Verify in browser**

Expected:
1. With a Leader open, click "Add Ability" — see filtered list of L1-L4 abilities
2. Search for "Marksman" — find it, click to add
3. Skill grid shows Shoot with +1 bonus
4. Try to add "Marksman" again — greyed out with "Already selected"
5. Add "Animal" — prompted to pick 2 skills for the +1d bonus
6. Shoot becomes no-dice, chosen skills get +1
7. Rename "Animal" flavor to "Wolf Spirit" — see it update

- [ ] **Step 5: Commit**

```bash
git add BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add ability picker with validation, flavor names, and pick-target UI"
```

---

## Task 11: Perk Browser + Associate Builder

**Files:**
- Modify: `BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html` (Section 5e-5f)

- [ ] **Step 1: Build PerkBrowser component**

Center panel view when `activeView === 'perks'`. Shows:
- List of all perks grouped by slot cost (0, 1, 2, 3, 4, 10)
- Each perk shows: name, slot cost, description, prerequisites
- Already-selected perks highlighted
- Perks whose prerequisites aren't met are greyed out with reason
- Clicking toggles selection (add/remove from league)
- Slot impact shown: "Adding this uses 2 of your 3 remaining slots"
- League-restructuring perks show a warning about their structural impact

- [ ] **Step 2: Build AssociateBuilder component**

Center panel view when `activeView === 'associates'`. Shows:
- List of current associates (name, 2 abilities each)
- "Add Associate" button (disabled if at max)
- For each associate: name input, 2 dropdown/picker selectors from the shared ability pool
- No duplicate abilities across all associates in the league
- Remove button per associate

- [ ] **Step 3: Wire sidebar to open perk and associate views**

Clicking the Perks section header or "+ Perk" in the sidebar switches to perks view. Same for Associates.

- [ ] **Step 4: Verify in browser**

Expected:
1. Click Perks in sidebar — see all perks listed by slot cost
2. Select "Well Armed" (1 slot) — slot counter updates, perk appears in sidebar
3. Try adding a 2-slot perk — see slot impact preview
4. Add an Associate — pick 2 abilities, see them appear
5. Try duplicating an ability across associates — greyed out

- [ ] **Step 5: Commit**

```bash
git add BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add PerkBrowser and AssociateBuilder components"
```

---

## Task 12: Gang Editor

**Files:**
- Modify: `BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html` (Section 5g)

- [ ] **Step 1: Build GangEditor component**

A variant of CharacterEditor for gang-type characters:
- No skill assignment grid (skills are computed from model count)
- Model count display (5, or 6 with Sixth-Man)
- Gang ability picker: select 1 from the gang abilities list, plus allowed L1/L2 abilities
- Display computed skills based on model count
- Name and image upload same as regular characters

- [ ] **Step 2: Verify in browser**

Expected:
1. Add a Gang — see the GangEditor with 5 models
2. See computed skills: Brawl/Shoot/Might 3d6, Dodge/Cunning/Finesse 1d6
3. Select "Sixth-Man" ability — model count becomes 6, combat dice become 3d6 (same since ceil(6/2)=3)
4. Select "Dangerous" ability — description shown

- [ ] **Step 3: Commit**

```bash
git add BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add GangEditor component for gang character type"
```

---

## Task 13: Print Output

**Files:**
- Modify: `BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html` (Section 6: PRINT OUTPUT)

- [ ] **Step 1: Build PrintableRoster component**

A `print-only` div (hidden on screen, visible when printing) that renders:

1. **Roster summary header:** League name, genre (if any), total characters, perks list, slot usage
2. **Character cards** (one per character): styled like the book's format
   - Name (large, serif font), mini image thumbnail if set
   - Health die type
   - 6-row skill table with final computed values
   - Abilities listed with flavor names and descriptions
   - For gangs: model count instead of health
3. **Associates section:** each associate with their 2 abilities
4. **Campaign summary** (if active): Reputation, XP, Resources

Style the cards with period-appropriate borders on white background. Each card uses `break-inside: avoid` for clean page breaks.

- [ ] **Step 2: Wire print button**

TopBar's Print button calls `window.print()`. The CSS `@media print` rules hide `no-print` elements and show `print-only` elements.

- [ ] **Step 3: Verify print output**

Click Print → browser print preview should show:
1. Clean white background with roster summary
2. Individual character cards with all skills and abilities
3. No UI chrome visible
4. Cards don't break across pages

- [ ] **Step 4: Commit**

```bash
git add BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add printable roster cards and print stylesheet"
```

---

## Task 14: Polish — Delete Character, Reorder, Toast Notifications, Edge Cases

**Files:**
- Modify: `BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html`

- [ ] **Step 1: Add delete character functionality**

Each character card in the sidebar gets a delete button (trash icon). Confirmation prompt before deletion. Deleting reclaims roster slots.

- [ ] **Step 2: Add delete league functionality**

In the league selector, add a delete button per league. Confirmation prompt.

- [ ] **Step 3: Handle empty state gracefully**

When no characters exist, center panel shows: "Start by creating your Leader" with a button that opens the add modal pre-set to Leader type. The Leader option in the add modal is mandatory and highlighted when the roster is empty.

- [ ] **Step 4: Add character completeness indicators**

In the sidebar, incomplete characters (unassigned skills, missing name, unfilled ability slots) show an amber warning badge. Add a function `isCharacterComplete(character, gameData)` that checks all required fields.

- [ ] **Step 5: Verify all edge cases**

- Create and delete characters, verify slot counts update
- Try to exceed slot limit — button should be disabled
- Create a league with all character types: Leader, Sidekick, Ally, Follower, Gang
- Export as JSON, delete league, import it back — verify all data intact
- Print a full roster — all characters render correctly

- [ ] **Step 6: Commit**

```bash
git add BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add delete, empty states, completeness indicators, polish"
```

---

## Task 15: Final Integration Test — Build the Sky Pirates Example

**Files:** No new files — this is a verification task.

- [ ] **Step 1: Recreate the Sky Pirates league from the rulebook (page 18)**

Using the app, build the complete Sky Pirates league:
- **League Perk:** Well Armed (1 slot)
- **Associate:** Ghibli Russo (Tinker, Lucky Charm)
- **Leader — Raven:** Health d10, Brawl 2d8, Shoot 3d10, Dodge 3d10, Might 2d8, Finesse 3d10, Cunning 3d10. Abilities: Quick-Shot (L3), Marksman (L1, +1 Shoot → 4d10), Agile (L1, +1 Dodge → 4d10)
- **Sidekick — Number-Two:** Health d8, Brawl 3d8, Shoot 3d8, Dodge 3d8, Might 2d6, Finesse 2d6, Cunning 2d6. Abilities: Marksman (L1, +1 Shoot → 4d8), Agile (L1, +1 Dodge → 4d8)
- **Ally — Twenty-One:** Health d6, Brawl 2d6, Shoot 2d6, Dodge 2d6, Might 1d6, Finesse 1d6, Cunning 1d6. Ability: Trick (L1)
- **Ally — Twenty-Four:** Health d6, Brawl 2d6, Shoot 2d6, Dodge 1d6, Might 1d6, Finesse 1d6, Cunning 1d6. Ability: Burst Fire (L2)
- **Follower — Speedy:** Health d6*, all skills 1d6 except Dodge 2d6. Ability: Agile (L1, +1 Dodge → 2d6)

Slot check: 0 (Leader) + 3 (Sidekick) + 2+2 (Allies) + 1 (Follower) + 1 (Perk) + 1 (Associate) = 10/10 slots.

- [ ] **Step 2: Verify all computed values match the rulebook**

Compare every character's final skill values against the book's example on page 18. All should match exactly.

- [ ] **Step 3: Print the roster**

Click Print and verify the output looks like clean character cards matching the Pulp Dossier theme.

- [ ] **Step 4: Export and re-import**

Export as JSON, delete the league, import it, verify everything restored correctly.

- [ ] **Step 5: Commit (if any fixes were needed)**

```bash
git add BlitzTacticsBundle_1/PulpAlley/PulpAlley_Forge.html
git commit -m "fix(pulp-alley): integration fixes from Sky Pirates example verification"
```

---

## Summary

| Task | Description | Scope |
|---|---|---|
| 1 | JSON data — templates + L1 abilities | Core |
| 2 | JSON data — all remaining abilities | Core |
| 3 | JSON data — perks, associates, weapons, societies | Core + Extended data |
| 4 | HTML scaffold — CDN, theme, app shell, data loader | Core |
| 5 | Persistence — LocalStorage, UUID, league state | Core |
| 6 | Validation engine — pure logic | Core |
| 7 | Skill computation engine | Core |
| 8 | TopBar + RosterSidebar + AddModal | Core |
| 9 | CharacterEditor — skill assignment grid | Core |
| 10 | CharacterEditor — ability picker | Core |
| 11 | PerkBrowser + AssociateBuilder | Core |
| 12 | GangEditor | Core |
| 13 | Print output | Core |
| 14 | Polish — delete, empty states, edge cases | Core |
| 15 | Integration test — Sky Pirates example | Core |

**Total: 15 tasks covering full Core scope.** Extended scope (Epic characters, Campaign panel, Weapons, Genres) can be added as follow-up tasks after Core is verified.
