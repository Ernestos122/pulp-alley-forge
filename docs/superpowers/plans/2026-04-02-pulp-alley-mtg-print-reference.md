# MTG Card Print + Complete Reference Tables Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace print layout with MTG-sized character cards, add 19 toggleable reference table sections configured in Settings, add 11 new data tables to JSON, rename main file to index.html.

**Architecture:** Data-first approach — add all new JSON data, then modify the single HTML file: replace PrintableRoster with MTG card format, remove PrintModal, add printSettings to league state and SettingsPanel, add reference section renderers, rename file.

**Tech Stack:** React 18, Tailwind CSS (CDN), inline JSX/Babel — files: `index.html` (renamed), `PulpAlley_Data.json`

**Working directory:** `C:\Users\soyer\Downloads\Blitz_Tactics\PulpAlley`

---

### Task 1: Add New Reference Data to PulpAlley_Data.json

**Files:**
- Modify: `PulpAlley_Data.json`

This task adds 11 new top-level keys to the JSON data file. The data is transcribed from the Pulp Alley 2E Core Rules PDF.

- [ ] **Step 1: Add `resourcesOverview` (Tips + general resource rules)**

Add after the `harrowingEscape` key. This replaces the original `tipsAssets` concept since Tips don't have a purchase table — they have 3 spending rules.

```json
"resourcesOverview": {
  "description": "Resource points (Tips, Contacts, Backup, and Gear) may be earned at the end of each scenario, based on the plot points (rewards) your league holds. During scenario set-up, you may spend some or all of your resource points to select assets for the upcoming scenario. Points are permanently removed when used. Assets do not carry over to the next scenario unless you pay again.",
  "tips": [
    { "name": "Tip-off", "description": "During a scenario, you may spend one Tips point at the start of any turn to draw +1 Fortune card." },
    { "name": "Solid Leads", "description": "Before a scenario you may convert two of your Tips to one other resource — Backup, Contact, or Gear." },
    { "name": "Associate Tip", "description": "Instead of rolling for an Associate, you may spend one Tips point to automatically pass a Support check." }
  ],
  "dominion": "If you select the Dominion perk, you gain access to Minions, Cult, and Gifts assets. However, you cannot use resource points to select from normal Backup, Contacts, or Gear tables. You may still spend Gear points on Gadgets, Vehicles, and Guns. You may also use Tips as normal."
},
```

- [ ] **Step 2: Add `contactsAssets`**

```json
"contactsAssets": [
  { "name": "Friendly Local", "cost": 1, "description": "You gain one random level 1 Backup character." },
  { "name": "Good Rumors", "cost": 1, "description": "You may draw one extra Fortune card on turn #1." },
  { "name": "Snitch", "cost": 1, "description": "Shift your die-type up when rolling for starting Director." },
  { "name": "A Few Maneuvers", "cost": 2, "description": "Your Leader gains: Mindgame — Roll an opposed Finesse check against the Director's Leader. If you win, you immediately become Director. May be used once per scenario." },
  { "name": "Unexpected Ally", "cost": 2, "description": "You gain one random level 2 Backup character." },
  { "name": "Savoir-Faire", "cost": 2, "description": "(one use) Your Leader may pass one challenge automatically — instead of rolling." },
  { "name": "Informant", "cost": 3, "description": "Draw one Fortune card each time you become Director." },
  { "name": "Quick Travel", "cost": 3, "description": "You may pick a Random Event — instead of rolling." },
  { "name": "One Step Ahead", "cost": 3, "description": "Pick one of the following abilities for your Leader: Lucky Devil, Danger Sense, or Master of Disguise." },
  { "name": "Disguise", "cost": 3, "description": "Your Leader starts in hiding. While hiding, you may discard one Fortune card to automatically win an opposed spotting check — instead of rolling. Cannot be used when attempting an ambush." },
  { "name": "Double-Cross!", "cost": 4, "description": "Your Leader gains: Double-Cross — (one use) At the start of any turn, select one enemy character within 12\" of your Leader. That character immediately switches sides and is under your control for the remainder of the scenario. The targeted character keeps their current Health and conditions." }
],
```

- [ ] **Step 3: Add `gearAssets`**

```json
"gearAssets": [
  { "name": "Diving Suit", "cost": 1, "description": "You gain the Aquatic ability." },
  { "name": "Gadget X", "cost": 1, "description": "(one use) You automatically pass one Plot Point — instead of rolling." },
  { "name": "Smoke Grenades", "cost": 1, "description": "(one use) As an action, place a 3\" (dia.) area within 6\". This area blocks line-of-sight until the end of the turn." },
  { "name": "Utility Belt", "cost": 1, "description": "(one use) You automatically pass one Peril — instead of rolling." },
  { "name": "Bulletproof Vest", "cost": 2, "description": "You always count as in cover." },
  { "name": "Chemical X", "cost": 2, "description": "You gain one ability: Speedy, Eagle-Eyed, or Shadowy." },
  { "name": "Explosives", "cost": 2, "description": "You gain: Plant Charge — Place a charge counter next to your base. Detonate — Place a 3\" burst centered on each of your charges, resolve as normal then remove charges." },
  { "name": "Flight Pack", "cost": 2, "description": "You gain the Winged ability." },
  { "name": "Deflector", "cost": 2, "description": "You gain the Agile ability." },
  { "name": "Body Armor", "cost": 3, "description": "You gain the Armored ability." },
  { "name": "Microgrenades", "cost": 3, "description": "You gain the Burst Fire ability." },
  { "name": "Weapon X", "cost": 3, "description": "You gain one ability: Marksman or Fierce." },
  { "name": "Experimental Jetpack", "cost": 4, "description": "(one use) Instead of moving normally, you may move to anywhere on the table, ignoring intervening terrain and characters. You encounter a peril at the destination." }
],
```

- [ ] **Step 4: Add `gadgets`**

```json
"gadgets": {
  "rules": "Each Gadget must be assigned to a specific character. You cannot select any Gadget more than once per scenario. Mishap: Each time you draw to determine the X number for a Gadget, check the story-icon in the bottom-right corner of the card. A mishap occurs when you draw the Alarm icon. The gadget does not function and is removed from play. You must immediately roll for the peril challenge on the same card.",
  "assets": [
    { "name": "Boom-Bot", "cost": 1, "description": "Add one Boom-Bot backup (Level 1, 1d6 all skills). Boom-Bot is knocked out if it activates over 12\" from its assigned character. BOOM: When Boom-Bot is knocked out, all characters within X\" must roll for the peril on the X card." },
    { "name": "Ray Projector", "cost": 1, "description": "Once per turn, gain a +X Shoot bonus." },
    { "name": "7-League Boots", "cost": 1, "description": "During your activation, you may move an extra X\" + 3\"." },
    { "name": "Shock Gauntlet", "cost": 1, "description": "Once per turn, gain a +X Brawl bonus." },
    { "name": "Sonic Spanner", "cost": 1, "description": "Instead of rolling for a challenge, you may automatically score X successes." },
    { "name": "Tesla Shield", "cost": 2, "description": "When you take two or more hits, you may cancel X hits." },
    { "name": "Phase Vest", "cost": 2, "description": "Instead of moving normally, remove your character and place a phase counter. On the next activation, place your character within X\" of the counter." },
    { "name": "Disintegrator", "cost": 3, "description": "Once per turn, target one character within 12\". Target must roll X successes for a Health check or be immediately knocked out." },
    { "name": "Death Ray", "cost": 3, "description": "Once per turn, gain a +X Shoot bonus. Maximum range 24\"." },
    { "name": "Force Field", "cost": 3, "description": "At the start of your activation, place a X\" (dia.) force field centered on your base. All characters inside gain cover. The field lasts until your next activation." }
  ]
},
```

- [ ] **Step 5: Add `backupCharacters`**

```json
"backupCharacters": {
  "rules": "All assets are temporary, purchased for one scenario only. Skills not listed are 1d6. d6* means the character is knocked out when they fail a Health check instead of going down.",
  "assets": [
    { "name": "Level 1: Brawler", "cost": 1, "skills": "Brawl 2d6", "health": "d6*" },
    { "name": "Level 1: Shooter", "cost": 1, "skills": "Shoot 2d6", "health": "d6*" },
    { "name": "Level 1: Scout", "cost": 1, "skills": "Dodge 2d6", "health": "d6*" },
    { "name": "Level 1: Animal", "cost": 1, "skills": "Brawl 2d6, Shoot no-dice, Dodge 2d6", "health": "d6*" },
    { "name": "Level 2: Brawler", "cost": 2, "skills": "Brawl 3d6, Dodge 2d6", "health": "d6" },
    { "name": "Level 2: Shooter", "cost": 2, "skills": "Shoot 3d6, Dodge 2d6", "health": "d6" },
    { "name": "Level 2: Scout", "cost": 2, "skills": "Dodge 3d6, Finesse 2d6", "health": "d6" },
    { "name": "Level 2: Animal", "cost": 2, "skills": "Brawl 3d6, Shoot no-dice, Dodge 3d6", "health": "d6" },
    { "name": "Level 2: Gang (Armed)", "cost": 2, "skills": "Gang — Armed (see Gangs)", "health": "Gang" },
    { "name": "Level 2: Gang (Dangerous)", "cost": 2, "skills": "Gang — Dangerous (see Gangs)", "health": "Gang" },
    { "name": "Level 3: Brawler", "cost": 3, "skills": "Brawl 4d8, Shoot 2d6, Dodge 3d8, Might 4d8, Finesse 2d6, Cunning 2d6", "health": "d8" },
    { "name": "Level 3: Shooter", "cost": 3, "skills": "Brawl 2d6, Shoot 4d8, Dodge 3d8, Might 2d6, Finesse 2d6, Cunning 4d8", "health": "d8" },
    { "name": "Level 3: Scout", "cost": 3, "skills": "Brawl 2d6, Shoot 2d6, Dodge 4d8, Might 2d6, Finesse 4d8, Cunning 3d8", "health": "d8" },
    { "name": "Level 3: Animal", "cost": 3, "skills": "Brawl 5d8, Shoot no-dice, Dodge 4d8, Might 3d8, Finesse 2d6, Cunning 2d6", "health": "d8" },
    { "name": "Level 3: Custom", "cost": 4, "skills": "Create one Sidekick of your choice", "health": "TBD" }
  ]
},
```

- [ ] **Step 6: Add `mounts`**

```json
"mounts": {
  "rules": "Mounts are not characters and cannot activate. Riders may move up to 16\" (instead of 12\") and may freely move away from an engaged enemy. Normally up to two characters (colleagues only) may ride the same mount. Loose mounts wander 1d8\" in a random direction at the end of each turn. A basic mount costs 1 Resource point (any type). Each additional mount ability costs +1 point of the listed type.",
  "abilities": [
    { "name": "Skimmer", "cost": 0, "description": "Ignores the rule for Loose Mounts." },
    { "name": "Single-Rider", "cost": 0, "description": "Cannot carry more than one rider. Select one free ability: Firepower, Flying, Stalwart, or Watchful." },
    { "name": "Aggressive", "cost": 1, "description": "Riders receive a +1 Brawl and -1 Dodge." },
    { "name": "Blaze", "cost": 1, "description": "Riders gain the Short Burst ability. One rider per turn may perform this action while mounted." },
    { "name": "Carapace", "cost": 1, "description": "Riders automatically pass all 1d Health checks. Mount cannot move over 6\" except to rush." },
    { "name": "Defensive", "cost": 1, "description": "Riders receive a +1 Dodge and -1d Brawl." },
    { "name": "Faithful", "cost": 1, "description": "(one per league) Once per game, at the start of any turn, place this mount 1d8\" away from your Leader." },
    { "name": "Firepower", "cost": 1, "description": "Riders gain +1 Shoot if mount is pointing directly at the target." },
    { "name": "Flying", "cost": 1, "description": "Riders gain the Winged ability." },
    { "name": "Large", "cost": 1, "description": "Mount may carry 2 extra colleagues. Cannot move over 12\"." },
    { "name": "Stalwart", "cost": 1, "description": "The first time this mount would be removed from play, the result is ignored and the mount remains in play." },
    { "name": "Sturdy", "cost": 1, "description": "Riders gain the Rugged ability. If you roll a 1 on the re-roll, the character is injured and the mount is removed from play." },
    { "name": "Trusty", "cost": 1, "description": "This mount never wanders." },
    { "name": "Watchful", "cost": 1, "description": "Riders gain a +1 bonus for all perils and spotting checks." }
  ]
},
```

- [ ] **Step 7: Add `vehicles`**

```json
"vehicles": {
  "rules": "Driving cannot be combined with normal movement. Cruise: Move up to 8\" in a straight line forward (not an action). Maneuver: Move up to 8\" in any direction (action, cannot combine with other actions or fighting). Passengers: Total characters a vehicle may safely carry, including driver. Extra characters equal to Size may hang-on (perilous). Control checks required when: hit by burst, maneuvering in hazardous area, driver fails Health check, or vehicle collision.",
  "basic": [
    { "name": "Motorcycle", "size": 1, "passengers": 1, "reliability": "d6", "special": "Nimble, Open", "cost": 1 },
    { "name": "Motorcycle & Sidecar", "size": 1, "passengers": 2, "reliability": "d6", "special": "Open", "cost": 1 },
    { "name": "Roadster", "size": 2, "passengers": 2, "reliability": "d8", "special": "Racer", "cost": 2 },
    { "name": "Coupe / Sedan", "size": 2, "passengers": 4, "reliability": "d8", "special": "—", "cost": 2 },
    { "name": "Pickup", "size": 2, "passengers": 6, "reliability": "d8", "special": "Lumbering", "cost": 2 },
    { "name": "Cargo Truck", "size": 3, "passengers": 10, "reliability": "d10", "special": "Lumbering", "cost": 3 }
  ],
  "specials": [
    { "name": "Lumbering", "description": "Vehicle cannot maneuver over 4\" if you change its heading." },
    { "name": "Nimble", "description": "The driver may cruise before or after maneuvering." },
    { "name": "Open", "description": "Characters on this vehicle may engage and be engaged by an enemy in contact. Either may move away on their activation rather than remaining engaged." },
    { "name": "Racer", "description": "You may add an additional +1d6\" to your cruise move." }
  ]
},
```

- [ ] **Step 8: Add `minions`**

```json
"minions": {
  "rules": "Requires the Dominion perk. Purchased with Backup points. All assets are temporary, one scenario only. Skills not listed are 1d6. d6* means knocked out on failed Health check instead of going down.",
  "assets": [
    { "name": "Level 1: Living Dead", "cost": 1, "health": "d6*", "skills": "Dodge no-dice", "abilities": "Reanimated" },
    { "name": "Level 1: Mindless", "cost": 1, "health": "d6*", "skills": "Brawl 1d8, Might 1d8, Finesse no-dice, Cunning no-dice", "abilities": "" },
    { "name": "Level 1: Swarm", "cost": 1, "health": "d6*", "skills": "", "abilities": "Swarm" },
    { "name": "Level 1: Flyer", "cost": 1, "health": "d6*", "skills": "", "abilities": "Winged" },
    { "name": "Level 2: Guardian", "cost": 2, "health": "d8", "skills": "Brawl 2d6, Dodge no-dice, Might 2d8, Finesse no-dice", "abilities": "" },
    { "name": "Level 2: Cultist", "cost": 2, "health": "d6", "skills": "Shoot 3d6, Dodge 2d6", "abilities": "" },
    { "name": "Level 2: Fanatic", "cost": 2, "health": "d6", "skills": "Brawl 3d6, Dodge 2d6", "abilities": "" },
    { "name": "Level 2: Mob", "cost": 1, "health": "Gang", "skills": "Gang", "abilities": "Mob (see Gangs)" },
    { "name": "Level 2: Fiends", "cost": 2, "health": "Gang", "skills": "Gang", "abilities": "Swarm (see Gangs)" },
    { "name": "Level 3: Skin-Walker", "cost": 3, "health": "d8", "skills": "Brawl 2d6, Shoot 2d8, Dodge 3d8, Might 2d6, Finesse 3d6, Cunning 4d8", "abilities": "Shapeshifter: Spawn, Mutant, or Spectre" },
    { "name": "Level 3: Spawn", "cost": 3, "health": "d8", "skills": "Brawl 4d8, Shoot no-dice, Dodge 3d8, Might 3d6, Finesse 3d8, Cunning 2d6", "abilities": "Indomitable" },
    { "name": "Level 3: Mutant", "cost": 3, "health": "d10", "skills": "Brawl 3d8, Shoot 2d6, Dodge 3d6, Might 3d10, Finesse no-dice, Cunning 2d6", "abilities": "Savage" },
    { "name": "Level 3: Spectre", "cost": 3, "health": "d8", "skills": "Brawl 2d6, Shoot 2d6, Dodge 4d8, Might 2d6, Finesse 3d8, Cunning 3d8", "abilities": "Shock" }
  ]
},
```

- [ ] **Step 9: Add `cultAssets`**

```json
"cultAssets": {
  "rules": "Requires the Dominion perk. Purchased with Contacts points. All assets are temporary, one scenario only. Cannot take any Cult asset more than once per scenario.",
  "assets": [
    { "name": "Eyes and Ears", "cost": 1, "description": "You may draw one extra Fortune card on turn #1." },
    { "name": "The Following", "cost": 1, "description": "During set-up, select one type of level 1 Minion for your Following. At the start of each turn, if you are the Director roll 1d6. On a 4+ deploy one of your Following." },
    { "name": "Receive Dreams", "cost": 1, "description": "Shift your die-type up when rolling for starting Director." },
    { "name": "The Brethren", "cost": 2, "description": "During set-up, select one type of level 2 Minion for your Brethren. At the start of each turn, if you are the Director roll 1d6. On a 4+ deploy one of your Brethren." },
    { "name": "Dark Oath", "cost": 2, "description": "Draw one Fortune card each time you become Director." },
    { "name": "Send Dreams", "cost": 2, "description": "Your Leader gains: Mindgame — Roll an opposed Finesse check against the enemy Director's Leader. If you win, you immediately become Director. May be used once per scenario." },
    { "name": "Blood Oath", "cost": 3, "description": "During set-up, remove one Ally or Follower from your roster to gain an additional ability for your Leader: Savage, Regenerate, or Inhuman." },
    { "name": "Hearts and Minds", "cost": 3, "description": "Instead of rolling a Recovery check for your Leader, remove a colleague from play within 6\" of your Leader. Your Leader automatically passes this Recovery check." },
    { "name": "Unspeakable Oath", "cost": 4, "description": "During set-up, remove one Sidekick from your roster. Your Leader gains one Epic ability." }
  ]
},
```

- [ ] **Step 10: Add `gifts`**

```json
"gifts": {
  "rules": "Requires the Dominion perk. Purchased with Gear points. All assets are temporary, one scenario only. Each Gift must be assigned to a specific character. A character cannot be assigned the same asset more than once.",
  "assets": [
    { "name": "Aura of Night", "cost": 1, "description": "(one-use) As an action, you cannot be rushed, targeted, or attacked for the remainder of this turn." },
    { "name": "Glyph of Knowing", "cost": 1, "description": "(one-use) You automatically pass one challenge — instead of rolling." },
    { "name": "Shadow Step", "cost": 1, "description": "Instead of moving normally, move anywhere within 12\", ignoring all intervening terrain and characters. May be used even if engaged." },
    { "name": "Talisman", "cost": 2, "description": "You always count as in cover." },
    { "name": "Ensorcelled Rune", "cost": 2, "description": "You gain one ability: Winged, Shadowy, or Aquatic." },
    { "name": "Glyph of Defense", "cost": 2, "description": "Your Dodge is increased by +1d." },
    { "name": "Doppelganger", "cost": 3, "description": "(Leader only) As a full action, target one Level 1 or 2 character within 12\" to place a Doppelganger within 3\" of your base. The Doppelganger's profile is identical to the target, under your control, but does not come into play ready." },
    { "name": "Glyph of Power", "cost": 3, "description": "You gain one ability: Marksman or Fierce." },
    { "name": "Scry", "cost": 3, "description": "You may pick a Random Event — instead of rolling." },
    { "name": "Summoning Circle", "cost": 4, "description": "(Leader only) During set-up, select one type of level 1 Minion for your summoning. As a full action, if you are ready, deploy one of your summoned Minions within 3\" of your base." }
  ]
},
```

- [ ] **Step 11: Add `quickReference`**

```json
"quickReference": {
  "title": "Action Sequence Quick Reference",
  "phases": [
    { "name": "DIRECT", "description": "The Director selects a player to activate one character. The Director can select any player that has one or more ready characters — including themselves." },
    { "name": "ACT", "steps": [
      "1. Fortune Effects: Allow time for other players to play Fortune effects when you activate your character.",
      "2. Automatic Effects: Resolve effects that occur automatically when you activate — such as perilous areas, horror, and so on.",
      "3. Fight On: If you activate in contact with an enemy then you must immediately fight.",
      "4. Move: You normally have the option to move up to 12\". If you move over 6\" then you cannot perform an action.",
      "5. Attack or Action: If you move into contact with an enemy, you must fight. If not engaged, you may make a ranged attack or perform an action. Activation ends after action or fight.",
      "6. End of Activation: Your character's activation ends."
    ]},
    { "name": "REPEAT", "description": "Repeat the Action Sequence (Direct & Act) until there are no ready characters in play." }
  ],
  "diceRules": "Each die that rolls a 4 or higher counts as one success. Bonus/Penalty: Temporarily increase or decrease the number of dice. Shift Up/Down: Roll next higher/lower dice-type. Re-Rolls apply to one die only, cannot re-roll the same die twice. No Dice: Character automatically fails any check involving a no-dice skill."
}
```

- [ ] **Step 12: Verify JSON is valid**

Run: `python -c "import json; json.load(open('PulpAlley_Data.json', encoding='utf-8')); print('Valid JSON')"`

Expected: `Valid JSON`

- [ ] **Step 13: Commit**

```bash
git add PulpAlley_Data.json
git commit -m "feat(pulp-alley): add 11 reference data tables from rulebook (contacts, gear, gadgets, backup, mounts, vehicles, minions, cult, gifts, resources, quick reference)"
```

---

### Task 2: Add printSettings to EMPTY_LEAGUE and Remove PrintModal

**Files:**
- Modify: `PulpAlley_Forge.html` (before rename)

- [ ] **Step 1: Add `printSettings` to EMPTY_LEAGUE**

Find `EMPTY_LEAGUE` factory function. After the `genre: null,` line, add a new `printSettings` object. Also remove `harrowing` from optionalRules if still present. The `optionalRules` line and new `printSettings` should look like:

```jsx
optionalRules: { weapons: false, supers: false, deadAlive: false, heartless: false, roadWarriors: false, scavengeAndSurvive: false, singleShot: false },
printSettings: {
    weapons: false,
    harrowingEscape: false,
    associateAbilities: false,
    reputationTable: false,
    secretSociety: false,
    characterTemplates: true,
    genres: false,
    perks: false,
    resourcesOverview: false,
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
},
```

- [ ] **Step 2: Add backwards compatibility in league load path**

Find where leagues are loaded from localStorage. In the `loadLeague` function or wherever a league object is returned, add after the league is parsed:

```jsx
if (!league.printSettings) league.printSettings = EMPTY_LEAGUE().printSettings;
```

Search for `JSON.parse` calls that return league objects and add this guard.

- [ ] **Step 3: Delete the PrintModal component**

Find and remove the entire `// 5n. PRINT SETTINGS MODAL` section (the `PrintModal` component).

- [ ] **Step 4: Remove showPrintModal state from App**

Find and remove: `const [showPrintModal, setShowPrintModal] = useState(false);`

- [ ] **Step 5: Remove PrintModal render from App's return**

Find and remove the block:
```jsx
{showPrintModal && (
    <PrintModal
        league={league}
        gameData={gameData}
        onClose={() => setShowPrintModal(false)}
    />
)}
```

- [ ] **Step 6: Revert TopBar print button to window.print()**

In the TopBar component, change `onPrint` back to `onClick={() => window.print()}`. Remove the `onPrint` prop from the destructured props.

In the App's TopBar usage, remove the `onPrint` prop.

- [ ] **Step 7: Revert MorePanel print button to window.print()**

In the MorePanel component, change `onPrint` back to `onClick={() => window.print()}`. Remove the `onPrint` prop from the destructured props.

In the App's renderMobileContent, remove the `onPrint` prop from the MorePanel usage.

- [ ] **Step 8: Remove window.__pulpAlleyPrintOptions from PrintableRoster**

Find and remove `const printOptions = window.__pulpAlleyPrintOptions || {};` from PrintableRoster. This will be replaced with `league.printSettings` in a later task.

- [ ] **Step 9: Verify page loads without errors**

Open in browser, refresh. Expected: Page loads, print button calls window.print() directly.

- [ ] **Step 10: Commit**

```bash
git add PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add printSettings to league state, remove PrintModal, revert to direct print"
```

---

### Task 3: Replace PrintableRoster with MTG Card Format

**Files:**
- Modify: `PulpAlley_Forge.html`

This task replaces the entire PrintableRoster component with the old MTG card format from commit `6811f61`, adapted to work with the current codebase. The implementer should read the old code from `C:\Users\soyer\Downloads\Blitz_Tactics\BlitzTacticsBundle_1` using:

```bash
cd C:\Users\soyer\Downloads\Blitz_Tactics\BlitzTacticsBundle_1 && git show 6811f61:src/index.html | sed -n '2823,2980p'
```

This extracts the old PrintableRoster. The implementer needs to:

- [ ] **Step 1: Read the old PrintableRoster from the bundle repo**

Read commit `6811f61:src/index.html` lines 2823-2980 from `C:\Users\soyer\Downloads\Blitz_Tactics\BlitzTacticsBundle_1`.

- [ ] **Step 2: Read the current PrintableRoster**

Read the current `PrintableRoster` component in `PulpAlley_Forge.html`.

- [ ] **Step 3: Replace the PrintableRoster component**

Replace the entire current `PrintableRoster` component with the old MTG card version, making these adaptations:

1. **Keep our roster header**: The header section should retain the genre description expansion and optional rule descriptions from our current version (the `activeGenre` modifier descriptions and the `coreDescriptions` IIFE for optional rules).

2. **Add printSettings reading**: At the top of the function, add `const printSettings = league.printSettings || {};` — this replaces the old `window.__pulpAlleyPrintOptions`.

3. **Fix computeCharacterSkills call**: The old code calls `computeCharacterSkills(character, gameData, activeGenre, league?.optionalRules, league)`. Check if the current function signature accepts those extra params. If not, use the current signature `computeCharacterSkills(character, gameData, activeGenre)`.

4. **Add actingLeaderId support**: The old code references `league.actingLeaderId`. If this doesn't exist in the current EMPTY_LEAGUE, that's fine — the `isActingLeader` check will just be false.

5. **Keep the reference appendix placeholder**: After the Associates section, add a comment `{/* Reference appendix sections rendered in Task 5 */}` where the reference tables will go.

6. **Remove all old reference table sections** that used `printOptions.*` — these will be re-added in Task 5 using `printSettings.*`.

- [ ] **Step 4: Verify print output in browser**

Open browser, create/load a league with characters, Ctrl+P. Expected: MTG-sized cards, 2 per row, with images, color-coded skills, health track, weapon details, abilities.

- [ ] **Step 5: Commit**

```bash
git add PulpAlley_Forge.html
git commit -m "feat(pulp-alley): replace PrintableRoster with MTG card format (63mm x 88mm)"
```

---

### Task 4: Add Print References Section to SettingsPanel

**Files:**
- Modify: `PulpAlley_Forge.html` — SettingsPanel component

- [ ] **Step 1: Read the current SettingsPanel component**

Find the SettingsPanel component in the file.

- [ ] **Step 2: Add Print References section after existing content**

After the existing "Advanced Settings" `</div>` and before the SettingsPanel's closing `</div>` and `);`, add a new section:

```jsx
{/* Print Reference Tables */}
<div className="p-4 rounded mt-6" style={{background:'var(--paper-dark)', border:'1px solid var(--border)'}}>
    <div className="typewriter text-xs uppercase tracking-wider mb-3" style={{color:'var(--crimson)'}}>Print Reference Tables</div>
    <p className="typewriter text-xs mb-4" style={{color:'var(--ink-faded)'}}>Select which reference tables to include when printing your roster.</p>

    {[
        { group: 'Core Reference', items: [
            { key: 'quickReference', label: 'Quick Reference', description: 'Action sequence summary and dice rules' },
            { key: 'characterTemplates', label: 'Character Templates', description: 'Stat templates for all character types' },
            { key: 'weapons', label: 'Weapons Table', description: 'Ranged and melee weapons with descriptions' },
            { key: 'perks', label: 'Perks Reference', description: 'All 40 league perks with descriptions' },
            { key: 'genres', label: 'Genre Reference', description: 'Selected genre description and modifiers' },
        ]},
        { group: 'Campaign', items: [
            { key: 'harrowingEscape', label: 'Harrowing Escape', description: 'Full Recovery & Critical Injury tables' },
            { key: 'reputationTable', label: 'Reputation Table', description: '22 reputation milestones (10-175)' },
            { key: 'secretSociety', label: 'Secret Society Perks', description: 'Selected society ranks and perks' },
            { key: 'associateAbilities', label: 'Associate Abilities', description: 'All 15 associate abilities' },
        ]},
        { group: 'Resources & Assets', items: [
            { key: 'resourcesOverview', label: 'Resources Overview', description: 'Tips spending rules and Dominion overview' },
            { key: 'contactsAssets', label: 'Contacts Assets', description: 'Contacts spending table' },
            { key: 'gearAssets', label: 'Gear Assets', description: 'Gear spending table' },
            { key: 'gadgets', label: 'Gadgets', description: 'Experimental gadgets with Mishap rules' },
            { key: 'backupCharacters', label: 'Backup Characters', description: 'Hired help stat blocks' },
        ]},
        { group: 'Mounts & Vehicles', items: [
            { key: 'mounts', label: 'Mounts', description: 'Mount types and abilities' },
            { key: 'vehicles', label: 'Vehicles', description: 'Vehicle profiles and driving rules' },
        ]},
        { group: 'Dominion', items: [
            { key: 'minions', label: 'Minions', description: 'Dominion perk minion stat blocks' },
            { key: 'cultAssets', label: 'Cult Assets', description: 'Dominion perk cult assets' },
            { key: 'gifts', label: 'Gifts', description: 'Dominion perk gift assets' },
        ]},
    ].map(({ group, items }) => (
        <div key={group} className="mb-4">
            <div className="typewriter text-xs font-bold mb-2" style={{color:'var(--ink-light)'}}>{group}</div>
            <div className="space-y-1">
                {items.map(({ key, label, description }) => {
                    const enabled = league.printSettings?.[key] || false;
                    return (
                        <div key={key} className="flex items-center justify-between p-2 rounded cursor-pointer hover:opacity-90"
                            style={{background: enabled ? 'var(--leather)' : 'var(--paper)', border: `1px solid ${enabled ? 'var(--crimson)' : 'var(--border)'}`}}
                            onClick={() => updateLeague(l => ({...l, printSettings: {...(l.printSettings || {}), [key]: !l.printSettings?.[key]}}))}>
                            <div className="flex-1 mr-2">
                                <div className="brand-font text-sm font-bold" style={{color: enabled ? 'var(--paper)' : 'var(--ink)'}}>{label}</div>
                                <div className="typewriter text-xs" style={{color: enabled ? 'var(--paper-dark)' : 'var(--ink-faded)'}}>{description}</div>
                            </div>
                            <div className="w-10 h-6 rounded-full relative flex-shrink-0" style={{background: enabled ? 'var(--crimson)' : 'var(--border)'}}>
                                <div className="absolute top-0.5 w-5 h-5 rounded-full transition-all" style={{background:'white', left: enabled ? '18px' : '2px'}} />
                            </div>
                        </div>
                    );
                })}
            </div>
        </div>
    ))}
</div>
```

- [ ] **Step 3: Verify Settings panel shows new section**

Open browser, navigate to Settings panel. Expected: New "Print Reference Tables" section with 5 groups of toggles. Quick Reference and Character Templates should be ON by default for new leagues.

- [ ] **Step 4: Commit**

```bash
git add PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add Print Reference Tables section to SettingsPanel with 19 toggles"
```

---

### Task 5: Add All 19 Reference Table Sections to PrintableRoster

**Files:**
- Modify: `PulpAlley_Forge.html` — PrintableRoster component

This task adds the rendering code for all 19 reference table sections inside PrintableRoster, after the Associates block, controlled by `printSettings`.

- [ ] **Step 1: Add all 19 reference sections**

Find the reference appendix placeholder comment in PrintableRoster. Replace it with the 19 conditional sections. Each follows the same pattern: check `printSettings.KEY`, render a print-card div with header and content.

The implementer should add all 19 sections. The first 6 (weapons, harrowingEscape, associateAbilities, reputationTable, secretSociety, characterTemplates) already had implementations in the old version — adapt those to use `printSettings` instead of `printOptions`. The remaining 13 are new.

Key rendering patterns:

**Simple list** (contactsAssets, gearAssets, cultAssets, gifts, associateAbilities): Bold name + cost (if applicable) + description.

**Table with columns** (backupCharacters, minions, vehicles.basic, reputationTable): HTML table with thead/tbody.

**Mixed** (gadgets, mounts, vehicles): Rules preamble text + asset list or table.

**Text block** (quickReference, resourcesOverview): Structured text with phase headers.

**Perks** (perks): All 40 perks from `gameData.perks`, grouped or listed with name, slotCost, category, description.

**Genres** (genres): Selected genre's full description + all modifier descriptions. If no genre selected, show all genres.

Each section uses:
- Container: `breakInside:'avoid', pageBreakInside:'avoid', border:'2px solid #000', borderRadius:'4px', padding:'12px', marginTop:'16px', background:'#fff'`
- Header: `fontFamily:'Playfair Display, Georgia, serif', fontSize:'14px', fontWeight:'bold', borderBottom:'2px solid #000', marginBottom:'8px', paddingBottom:'4px'`
- Content: `fontSize:'10px', fontFamily:'Special Elite, monospace'`
- Tables: `width:'100%', borderCollapse:'collapse'` with `borderBottom:'1px solid #ccc'` on rows

- [ ] **Step 2: Verify print output with all tables enabled**

Enable all 19 toggles in Settings, Ctrl+P. Expected: Character cards followed by all reference tables in order.

- [ ] **Step 3: Verify print output with all tables disabled**

Disable all toggles, Ctrl+P. Expected: Only character cards and associates.

- [ ] **Step 4: Commit**

```bash
git add PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add 19 reference table sections to PrintableRoster controlled by printSettings"
```

---

### Task 6: Rename File to index.html

**Files:**
- Rename: `PulpAlley_Forge.html` → `index.html`

- [ ] **Step 1: Rename the file**

```bash
git mv PulpAlley_Forge.html index.html
```

- [ ] **Step 2: Verify the file opens correctly**

Open `index.html` in browser. Expected: Everything works as before.

- [ ] **Step 3: Commit**

```bash
git add -A
git commit -m "refactor(pulp-alley): rename PulpAlley_Forge.html to index.html"
```

---

### Task 7: Final Verification and Push

- [ ] **Step 1: Full desktop verification**

Open `index.html` at full width:
1. Load data file
2. Create/open league
3. Settings panel: verify 19 print reference toggles exist in 5 groups
4. Toggle some on, print — verify reference tables appear
5. Character cards: MTG format with image, skills, health track, weapon
6. Verify desktop layout unchanged (sidebar + center panel)

- [ ] **Step 2: Mobile verification**

Resize to < 640px:
1. Bottom tab bar works
2. More > Settings shows print reference toggles
3. More > Print prints with selected reference tables
4. Character cards print in MTG format

- [ ] **Step 3: Print with everything enabled**

Enable all 19 toggles, create a league with:
- Pulp Westerns genre
- Weapons enabled
- Characters with abilities and weapons
- Associates
- Campaign reputation > 0
- Secret society selected

Print. Verify all 19 sections render correctly.

- [ ] **Step 4: Push**

```bash
git push origin main:master
```
