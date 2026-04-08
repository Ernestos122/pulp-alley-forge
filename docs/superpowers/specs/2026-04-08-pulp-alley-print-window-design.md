# Pulp Alley Forge: Print Window Approach

**Date:** 2026-04-08
**File:** `index.html`

## Overview

Replace the fragile `display: none` / `@media print` CSS approach with a dedicated print window. When the user clicks Print, a new browser window opens with a complete standalone HTML page containing the roster and reference tables. This works identically on every device and browser.

## What Gets Removed

### PrintableRoster component
Delete the entire `PrintableRoster` component (section 5k. PRINTABLE ROSTER) — the component definition plus all its contents (MTG cards, associates, reference appendix IIFE with all 19 sections, refCard helper, table styles).

### PrintableRoster render in App
Delete `<PrintableRoster league={league} gameData={gameData} />` from the App return.

### Print CSS
Delete the `@media print` block (lines 55-62) and the `.print-only { display: none; }` rule (line 63). Also remove any `no-print` classes from components — they're no longer needed since the print window is separate.

Actually, keep the `no-print` classes for now — removing them from dozens of elements is unnecessary churn and they're harmless. Only delete the `@media print` and `.print-only` CSS rules.

## What Gets Added

### `buildPrintHTML(league, gameData)` function

Pure function that takes `league` and `gameData` and returns a complete HTML string. No React, no JSX — plain string concatenation with template literals.

The function builds:

```
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>{league.name} — League Roster</title>
  <style>
    /* Google Fonts import for Special Elite + Playfair Display */
    /* Page layout: @page { margin: 0.5cm; } */
    /* Body: white background, Georgia serif default */
    /* Card styles: 63mm x 88mm, flex, parchment gradient */
    /* Skill boxes: combat red, action green */
    /* Health track styles */
    /* Reference table styles: borders, padding, monospace */
    /* Print-specific: page-break-inside avoid on cards */
  </style>
</head>
<body>
  <!-- Roster Header -->
  <!-- Character Cards (flex wrap, 2 per row) -->
  <!-- Associates -->
  <!-- Reference Tables (conditional on printSettings) -->
</body>
</html>
```

### Structure of buildPrintHTML

```js
const buildPrintHTML = (league, gameData) => {
    const ps = league.printSettings || {};
    const activeGenre = league.genre ? (gameData?.genres || []).find(g => g.name === league.genre) : null;

    // Helper: escape HTML
    const esc = (s) => String(s || '').replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');

    // Build CSS string
    const css = `...`;

    // Build roster header HTML
    const headerHTML = `...`;

    // Build character cards HTML
    const cardsHTML = league.characters.map(character => {
        // Compute skills, health, abilities — same logic as old PrintableRoster
        return `<div class="card">...</div>`;
    }).join('');

    // Build associates HTML
    const associatesHTML = league.associates.length > 0 ? `...` : '';

    // Build reference tables HTML (conditional on ps.KEY)
    let refHTML = '';
    if (ps.quickReference && gameData.quickReference) refHTML += buildQuickRef(gameData);
    if (ps.characterTemplates && gameData.characterTemplates) refHTML += buildTemplatesTable(gameData);
    if (ps.weapons && gameData.weapons) refHTML += buildWeaponsRef(gameData);
    // ... all 19 sections ...

    return `<!DOCTYPE html><html><head>...${css}...</head><body>${headerHTML}${cardsHTML}${associatesHTML}${refHTML}</body></html>`;
};
```

Each reference section is built by a small helper function that returns an HTML string. This keeps `buildPrintHTML` readable.

### `openPrintWindow(league, gameData)` function

```js
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
```

Handles popup blockers gracefully with an alert.

### Print button changes

Both print triggers call `openPrintWindow(league, gameData)` instead of `window.print()`:

**Desktop TopBar**: Change `onClick={() => window.print()}` to `onClick={() => openPrintWindow(league, gameData)}`

**Mobile dropdown**: Same change.

Since `openPrintWindow` needs `league` and `gameData`, and both TopBar and the mobile dropdown already have access to these via props, this is straightforward.

## Character Card HTML Generation

The MTG card layout is converted from JSX to HTML template literals. The logic stays identical:

- Card size: 63mm x 88mm
- Parchment gradient background
- Type + name header
- Image (22mm) or silhouette placeholder + color-coded skill boxes
- Weapon row with description
- Abilities section
- Health track bottom bar (or Gang KO info)

### Skill computation

The `computeCharacterSkills` function is called from within `buildPrintHTML` — it already exists as a pure function in the codebase, so it can be used directly. Same for `getTemplate`, `getTotalSlots`, `getUsedSlots`, `getAllAbilities`.

## Reference Table HTML Generation

Each of the 19 sections is a small function returning an HTML string:

```js
const buildRefCard = (title, content) => `
    <div style="break-inside:avoid; page-break-inside:avoid; border:2px solid #000; border-radius:4px; padding:12px; margin-top:16px; background:#fff;">
        <div style="font-family:Playfair Display,Georgia,serif; font-size:14px; font-weight:bold; border-bottom:2px solid #000; margin-bottom:8px; padding-bottom:4px;">${title}</div>
        <div style="font-size:10px; font-family:Special Elite,monospace;">${content}</div>
    </div>
`;
```

Then each section: `buildQuickRef`, `buildTemplatesTable`, `buildWeaponsRef`, `buildPerksRef`, `buildGenresRef`, `buildHarrowingRef`, `buildReputationRef`, `buildSecretSocietyRef`, `buildAssociateAbilitiesRef`, `buildResourcesRef`, `buildContactsRef`, `buildGearRef`, `buildGadgetsRef`, `buildBackupRef`, `buildMountsRef`, `buildVehiclesRef`, `buildMinionsRef`, `buildCultRef`, `buildGiftsRef`.

## What Stays Unchanged

- SettingsPanel print reference toggles
- `league.printSettings` data model
- `EMPTY_LEAGUE` printSettings defaults
- Mobile dropdown layout
- Desktop layout
- All editor components
- JSON data structure
- `no-print` classes on existing elements (harmless, can clean up later)
