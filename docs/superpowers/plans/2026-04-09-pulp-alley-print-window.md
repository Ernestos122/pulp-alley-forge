# Print Window Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the fragile CSS `display:none` / `@media print` print approach with a dedicated print window that generates a standalone HTML page — works reliably on all devices and browsers.

**Architecture:** A `buildPrintHTML(league, gameData)` pure function generates a complete standalone HTML string. An `openPrintWindow(league, gameData)` function opens a new browser window, writes the HTML, and triggers print. The old `PrintableRoster` React component and all `@media print` CSS are removed.

**Tech Stack:** Vanilla JS (template literals for HTML generation), no React/JSX in the print output. Single file: `index.html`

**Working directory:** `C:\Users\soyer\Downloads\Blitz_Tactics\PulpAlley`

---

### Task 1: Add `openPrintWindow` and `buildPrintHTML` Skeleton

**Files:**
- Modify: `index.html` — add new functions in the constants/utilities section (after the persistence helpers, before the DataLoader component)

- [ ] **Step 1: Find the insertion point**

Find the line `// 3. DATA LOADER COMPONENT`. Insert the new functions BEFORE it.

- [ ] **Step 2: Add the `openPrintWindow` function and `buildPrintHTML` skeleton**

Insert:

```jsx
// ==========================================
// 2b. PRINT WINDOW
// ==========================================
const openPrintWindow = (league, gameData) => {
    const html = buildPrintHTML(league, gameData);
    const win = window.open('', '_blank');
    if (!win) { alert('Please allow popups for this site to print.'); return; }
    win.document.write(html);
    win.document.close();
    win.onload = () => {
        win.print();
        win.onafterprint = () => win.close();
    };
};

const buildPrintHTML = (league, gameData) => {
    const ps = league.printSettings || {};
    const esc = (s) => String(s ?? '').replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
    const activeGenre = league.genre ? (gameData?.genres || []).find(g => g.name === league.genre) : null;
    const totalSlots = getTotalSlots(league, gameData);
    const usedSlots = getUsedSlots(league, gameData);
    const allAbPool = [...getAllAbilities(gameData), ...(gameData.abilities.epic || [])];

    // ── CSS ──
    const css = `
        @import url('https://fonts.googleapis.com/css2?family=Special+Elite&family=Playfair+Display:wght@400;700;900&display=swap');
        @page { margin: 0.5cm; }
        * { box-sizing: border-box; }
        body { font-family: Georgia, serif; color: #000; background: #fff; padding: 8px; margin: 0; }
        .header { border-bottom: 3px double #3b2a1a; padding-bottom: 6px; margin-bottom: 12px; }
        .header h1 { font-family: 'Playfair Display', Georgia, serif; font-size: 24px; font-weight: bold; margin: 0; color: #2a1f0e; }
        .header-row { display: flex; justify-content: space-between; align-items: baseline; }
        .header-meta { font-size: 10px; font-family: 'Special Elite', monospace; color: #5a4a2e; margin-top: 2px; }
        .cards { display: flex; flex-wrap: wrap; gap: 4mm; justify-content: flex-start; }
        .card { break-inside: avoid; page-break-inside: avoid; border: 1.5px solid #3b2a1a; border-radius: 4px; background: linear-gradient(145deg, #d4c5a9, #c4b48e); padding: 0; overflow: hidden; width: 63mm; height: 88mm; display: flex; flex-direction: column; font-family: 'Special Elite', monospace; font-size: 8px; color: #2a1f0e; box-shadow: 1px 1px 2px rgba(0,0,0,0.2); }
        .card-header { display: flex; justify-content: space-between; align-items: center; padding: 3px 5px; border-bottom: 1px solid #a89060; background: rgba(59,47,30,0.08); }
        .card-header .type { font-weight: bold; font-size: 7px; text-transform: uppercase; letter-spacing: 0.5px; color: #8b1a1a; }
        .card-header .name { font-family: 'Playfair Display', Georgia, serif; font-weight: bold; font-size: 10px; }
        .card-body { display: flex; padding: 4px 5px; gap: 4px; }
        .card-img { width: 22mm; flex-shrink: 0; }
        .card-img img { width: 22mm; height: 22mm; object-fit: cover; border-radius: 3px; border: 1px solid #a89060; }
        .card-img .placeholder { width: 22mm; height: 22mm; border-radius: 3px; border: 1px solid #a89060; background: rgba(168,144,96,0.2); display: flex; align-items: center; justify-content: center; }
        .combat-box { background: rgba(180,80,60,0.15); border: 1px solid rgba(180,80,60,0.3); border-radius: 2px; padding: 2px 4px; margin-bottom: 2px; }
        .action-box { background: rgba(60,120,60,0.12); border: 1px solid rgba(60,120,60,0.25); border-radius: 2px; padding: 2px 4px; }
        .skill-row { display: flex; justify-content: space-between; padding: 0.5px 0; font-size: 8px; }
        .combat-label { font-weight: bold; color: #6b2a1a; }
        .action-label { font-weight: bold; color: #2a5a2a; }
        .weapon-row { padding: 2px 5px; border-top: 1px solid #a89060; font-size: 7px; background: rgba(139,26,26,0.05); }
        .abilities { padding: 2px 5px; flex: 1; border-top: 1px solid #a89060; font-size: 7px; line-height: 1.25; overflow: hidden; }
        .health-track { display: flex; align-items: center; border-top: 1.5px solid #3b2a1a; background: rgba(59,47,30,0.1); margin-top: auto; }
        .health-label { padding: 2px 4px; font-weight: bold; font-size: 7px; color: #8b1a1a; white-space: nowrap; }
        .health-cell { flex: 1; text-align: center; padding: 2px 1px; font-size: 7px; font-weight: bold; }
        .health-down { color: #8b6914; background: rgba(139,105,20,0.1); }
        .health-out { color: #8b1a1a; background: rgba(139,26,26,0.1); }
        .associates { break-inside: avoid; page-break-inside: avoid; margin-top: 12px; border: 2px solid #3b2a1a; border-radius: 6px; padding: 10px; background: linear-gradient(145deg, #d4c5a9, #c4b48e); }
        .associates h2 { font-family: 'Playfair Display', Georgia, serif; font-size: 14px; font-weight: bold; border-bottom: 1px solid #a89060; margin: 0 0 6px; padding-bottom: 3px; color: #2a1f0e; }
        .ref-card { break-inside: avoid; page-break-inside: avoid; border: 2px solid #000; border-radius: 4px; padding: 12px; margin-top: 16px; background: #fff; }
        .ref-card h3 { font-family: 'Playfair Display', Georgia, serif; font-size: 14px; font-weight: bold; border-bottom: 2px solid #000; margin: 0 0 8px; padding-bottom: 4px; }
        .ref-content { font-size: 10px; font-family: 'Special Elite', monospace; }
        .ref-table { width: 100%; border-collapse: collapse; font-size: 10px; font-family: 'Special Elite', monospace; }
        .ref-table th { text-align: left; padding: 3px 6px; border-bottom: 2px solid #000; font-weight: bold; background: #f0f0f0; }
        .ref-table td { padding: 3px 6px; border-bottom: 1px solid #ccc; vertical-align: top; }
    `;

    // ── HELPERS ──
    const COMBAT_SKILLS = ['Brawl', 'Shoot', 'Dodge'];
    const ACTION_SKILLS = ['Might', 'Finesse', 'Cunning'];
    const HEALTH_TRACK = { 'd12':['d12','d10','d8','d6','Down','Out'], 'd10':['d10','d8','d6','Down','Out'], 'd8':['d8','d6','Down','Out'], 'd6':['d6','Down','Out'], 'd6*':['d6','Out'], 'none':[] };

    const getHealthDie = (character, template) => {
        if (!template) return 'd6';
        if (league.perks.includes('Duo') && character.type === 'sidekick') return 'd10';
        return template.health;
    };

    const refCard = (title, content) => `<div class="ref-card"><h3>${esc(title)}</h3><div class="ref-content">${content}</div></div>`;

    // ── ROSTER HEADER ──
    const headerHTML = buildHeader(league, gameData, activeGenre, usedSlots, totalSlots, esc);

    // ── CHARACTER CARDS ──
    const cardsHTML = buildCards(league, gameData, activeGenre, allAbPool, COMBAT_SKILLS, ACTION_SKILLS, HEALTH_TRACK, getHealthDie, esc);

    // ── ASSOCIATES ──
    const associatesHTML = buildAssociates(league, gameData, esc);

    // ── REFERENCE TABLES ──
    const refHTML = buildAllReferenceTables(league, gameData, ps, refCard, esc);

    return '<!DOCTYPE html><html><head><meta charset="UTF-8"><title>' + esc(league.name) + ' — League Roster</title><style>' + css + '</style></head><body>' + headerHTML + '<div class="cards">' + cardsHTML + '</div>' + associatesHTML + refHTML + '</body></html>';
};
```

Note: `buildHeader`, `buildCards`, `buildAssociates`, and `buildAllReferenceTables` are helper functions added in subsequent steps.

- [ ] **Step 3: Commit skeleton**

```bash
git add index.html
git commit -m "feat(pulp-alley): add openPrintWindow and buildPrintHTML skeleton with CSS"
```

---

### Task 2: Add Header and Cards HTML Builders

**Files:**
- Modify: `index.html` — add helper functions right after the `buildPrintHTML` function

- [ ] **Step 1: Read the current PrintableRoster header and card rendering**

Read the PrintableRoster component (starts at line ~2679). Study the roster header (lines ~2700-2740) and character card rendering (lines ~2741-2900) to understand the exact JSX structure that needs to be converted to HTML strings.

- [ ] **Step 2: Add `buildHeader` function**

Insert right after the closing `};` of `buildPrintHTML`:

The function should convert the PrintableRoster's header JSX to HTML template literals. It must include:
- League name + slots count
- Genre + perks display
- Genre modifier descriptions (expanded, with bold name — description format)
- Optional rules with descriptions (using hardcoded descriptions for Weapons/Supers, gameData.optionalSettings for the rest)
- Campaign state (reputation, XP, resources, society)

Read the current header code from PrintableRoster lines ~2700-2740 and convert each JSX block to a template literal returning an HTML string.

- [ ] **Step 3: Add `buildCards` function**

The function takes the same parameters as buildPrintHTML and returns HTML for all character cards. For each character:
- Compute skills via `computeCharacterSkills(character, gameData, activeGenre)`
- Get health die and track
- Build the card HTML: header (type + name), body (image + skills), weapon row, abilities, health track
- Gang characters: show model count and KO threshold instead of health track
- Acting leader badge
- All abilities with flavour names, free tags, descriptions

Read the current card rendering code from PrintableRoster lines ~2741-2900 and convert each JSX block to HTML template literals. Use the CSS classes defined in the skeleton (`.card`, `.card-header`, `.card-body`, `.combat-box`, `.action-box`, `.skill-row`, `.health-track`, etc.).

- [ ] **Step 4: Add `buildAssociates` function**

Convert the associates JSX (lines ~2892-2910) to HTML. Parchment-styled container with associate names and ability descriptions.

- [ ] **Step 5: Verify functions are syntactically correct**

The functions aren't called yet (PrintableRoster still exists), but they should parse without errors. Refresh the page and check the browser console.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat(pulp-alley): add buildHeader, buildCards, buildAssociates HTML generators"
```

---

### Task 3: Add Reference Tables HTML Builder

**Files:**
- Modify: `index.html` — add `buildAllReferenceTables` function after the associates builder

- [ ] **Step 1: Read the current reference sections**

Read the PrintableRoster reference appendix (lines ~2912-3210). It has an IIFE that defines `refCard`, `tblSt`, `thSt`, `tdSt` and returns a `React.Fragment` with 19 conditional sections.

- [ ] **Step 2: Add `buildAllReferenceTables` function**

The function takes `(league, gameData, ps, refCard, esc)` and returns a concatenated HTML string of all enabled reference tables.

Convert each of the 19 JSX sections to HTML template literals:

1. **quickReference**: phases with steps, dice rules
2. **characterTemplates**: HTML table with Type/Lvl/Health/Pools/Dice/Abilities/MaxLvl/Slots
3. **weapons**: Ranged + Melee subsections, name — description
4. **perks**: All 40 perks with slotCost, category, description (two-column layout via CSS column-count)
5. **genres**: Selected genre or all genres, with modifier descriptions
6. **harrowingEscape**: Two tables (Full Recovery, Critical Injury) with Roll/Result/Effect columns
7. **reputationTable**: Table with Rep/Milestone/Effect columns
8. **secretSociety**: Selected society's ranks and perks
9. **associateAbilities**: List of all 15, name — description
10. **resourcesOverview**: Description + tips rules + dominion note
11. **contactsAssets**: Table with Name/Cost/Description
12. **gearAssets**: Table with Name/Cost/Description
13. **gadgets**: Rules text + table with Name/Cost/Description
14. **backupCharacters**: Rules text + table with Name/Cost/Skills/Health
15. **mounts**: Rules text + abilities list with name (cost) — description
16. **vehicles**: Rules text + basic vehicles table + specials list
17. **minions**: Rules text + table with Name/Cost/Health/Skills/Abilities
18. **cultAssets**: Rules text + list with name (cost) — description
19. **gifts**: Rules text + list with name (cost) — description

Each section: `if (ps.KEY && gameData.KEY) html += refCard('Title', '...');`

For table sections, build HTML table strings using the `.ref-table` CSS class.

- [ ] **Step 3: Verify no syntax errors**

Refresh page, check console.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat(pulp-alley): add buildAllReferenceTables with 19 reference section HTML generators"
```

---

### Task 4: Wire Print Buttons and Remove Old Print System

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Replace desktop print button**

Find the TopBar's desktop print button (line ~721):
```jsx
<button onClick={() => window.print()} className="cursor-pointer p-1 rounded hover:opacity-80 desktop-only no-min-height" style={{color:'var(--paper)'}} title="Print">
```

Change `onClick={() => window.print()}` to `onClick={() => openPrintWindow(league, gameData)}`.

- [ ] **Step 2: Replace mobile print button**

Find the mobile dropdown print button (line ~800):
```jsx
<button onClick={() => { onToggleMobileMenu(); setTimeout(() => window.print(), 100); }}
```

Change to `onClick={() => { onToggleMobileMenu(); openPrintWindow(league, gameData); }}`. Remove the `setTimeout`.

- [ ] **Step 3: Delete the PrintableRoster component**

Find the entire section from `// 5k. PRINTABLE ROSTER` through its closing `};` (lines ~2679-3217). Delete it entirely.

- [ ] **Step 4: Delete the PrintableRoster render in App**

Find and delete:
```jsx
<PrintableRoster league={league} gameData={gameData} />
```

- [ ] **Step 5: Delete the @media print CSS and .print-only rule**

Delete lines 55-63:
```css
@media print {
    @page { margin: 0.5cm; size: auto; }
    html, body { height: auto !important; overflow: visible !important; background: white !important; margin: 0 !important; padding: 0 !important; color: black !important; }
    .no-print { display: none !important; }
    .print-only { display: block !important; position: static !important; width: 100% !important; height: auto !important; visibility: visible !important; }
    .print-only * { visibility: visible !important; }
    .print-card { break-inside: avoid; border: 2px solid #000; border-radius: 4px; margin-bottom: 16px; padding: 12px; background: #fff; page-break-inside: avoid; }
}
.print-only { display: none; }
```

- [ ] **Step 6: Verify the page loads and print button opens a new window**

Open `index.html` in browser. Click Print. Expected: a new browser window opens with the roster and any enabled reference tables. The new window triggers the print dialog.

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat(pulp-alley): wire openPrintWindow to print buttons, remove old PrintableRoster and print CSS"
```

---

### Task 5: Final Verification and Push

- [ ] **Step 1: Desktop print test**

1. Open league with characters, weapons enabled, some perks, a genre selected
2. Enable several reference tables in Settings (weapons, gear, gadgets, quick reference, etc.)
3. Click Print in TopBar
4. New window opens with: roster header, MTG cards, associates, reference tables
5. Print dialog appears
6. Verify all enabled tables show correctly

- [ ] **Step 2: Mobile print test (< 768px)**

1. Resize browser to mobile width
2. Open dropdown menu
3. Tap Print
4. New window opens with same content
5. Verify it works on mobile viewport

- [ ] **Step 3: Cross-browser test**

If possible, test in Chrome and Firefox. The print window approach should produce identical results since it's a standalone HTML page.

- [ ] **Step 4: Verify no leftover print-only references**

Search for `print-only` and `@media print` in the file. Only `no-print` classes should remain (harmless).

- [ ] **Step 5: Push**

```bash
git push origin main:master
```
