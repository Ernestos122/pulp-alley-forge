# Pulp Alley Forge: Mobile Responsive + Print Rules

**Date:** 2026-04-02
**File:** `PulpAlley/PulpAlley_Forge.html`
**Data:** `PulpAlley/PulpAlley_Data.json` (read-only)

## Feature 1: Print Sheet — Full Rule Descriptions

### Current behavior
- Genre modifiers printed as comma-joined names: `Genre Rules: Gritty, Riding the Range`
- Optional rules printed as comma-joined labels: `Optional Rules: Weapons, Single-Shot`
- No descriptions — useless as a reference during play

### Target behavior
Each active rule prints with its full description on its own line.

**Genre Rules block** (PrintableRoster, ~line 2522-2526):
```
Genre: Pulp Westerns
  Riding the Range — All characters may begin each scenario on a free basic mount...
  Gritty — In this setting, all dice-types over d8 are shifted down to d8...
```

**Optional Rules block** (PrintableRoster, ~line 2527-2541):
```
Optional Rules:
  Weapons — Allow characters to equip weapons...
  Single-Shot — Each character capable of shooting begins with one Loaded counter...
```

### Implementation
- Replace `activeGenre.modifiers.map(m => m.name).join(', ')` with a map rendering `<div>` per modifier: bold name + description
- Replace `active.map(s => s.label).join(', ')` with a map rendering `<div>` per rule: bold label + description
- For Weapons and Supers, hardcode descriptions (they aren't in optionalSettings):
  - Weapons: "Characters may equip weapons (Ranged and Melee)."
  - Supers: "Allows Epic characters in the league roster."
- Lookup optionalSettings descriptions from `gameData.optionalSettings` by matching `s.name`
- Styling: `fontSize: 10px`, `fontFamily: Special Elite, monospace`, color `#555`, each rule in its own `<div>` with `marginLeft: 12px`

## Feature 2: Mobile Responsive Layout

### Breakpoint
`640px` — aligns with Tailwind `sm:` breakpoint. Below 640px = mobile layout. Above = current desktop layout unchanged.

### Mobile detection
Add a `useIsMobile` hook using `window.matchMedia('(max-width: 639px)')` with a resize listener. This drives conditional rendering.

### TopBar changes (mobile)
- **Mobile**: Show only: ← back button + league name input (full remaining width)
- **Desktop** (`sm:` and up): Unchanged — all current controls remain
- Genre selector, Weapons toggle, Supers toggle, Export, Print buttons are hidden on mobile (moved to More tab)
- Genre modifier pills row: hidden on mobile

### Sidebar changes
- Add `hidden sm:flex` to sidebar container (currently `w-60 ... flex flex-col`)
- Desktop: unchanged
- Mobile: sidebar is completely hidden; its functionality moves to the Roster tab

### New: Bottom Tab Bar (mobile only)
- Rendered with `sm:hidden` — invisible on desktop
- Fixed to bottom of screen, above any safe-area inset
- `no-print` class
- 5 tabs using existing `Icons` object:
  1. **Roster** — `Icons.Users` — Full-screen character list
  2. **Editor** — `Icons.Edit2` — Character editor (greyed if none selected)
  3. **Perks** — star icon (new simple path) — PerkBrowser full-screen
  4. **Campaign** — `Icons.FileText` — CampaignPanel full-screen
  5. **More** — "⋯" text or three-dot icon — Settings/config panel

### New state: `mobileTab`
- Added to App component: `const [mobileTab, setMobileTab] = useState('roster')`
- Values: `'roster'` | `'editor'` | `'perks'` | `'campaign'` | `'more'`
- Only used when `isMobile` is true

### Tab: Roster (mobile)
- Full-screen list of characters grouped by type (reuses Section/CharCard pattern from RosterSidebar)
- SlotBar at top
- "Add Character" button at bottom
- Tapping a character card: sets `selectedCharId` and switches to `'editor'` tab

### Tab: Editor (mobile)
- Full-screen CharacterEditor for the selected character
- If no character selected: EmptyState component
- No separate back button needed — user taps Roster tab to go back

### Tab: Perks (mobile)
- Full-screen PerkBrowser (already works full-width)

### Tab: Campaign (mobile)
- Full-screen CampaignPanel (already works full-width)

### Tab: More (mobile)
- Full-screen panel containing:
  - **Genre selector**: `<select>` dropdown, full-width, same styling as current TopBar select but larger touch target
  - **Weapons toggle**: Toggle switch with label and description
  - **Supers toggle**: Toggle switch with label and description
  - **Associates**: Navigates to AssociateBuilder rendered full-screen in the content area (replaces More panel)
  - **Settings**: Navigates to SettingsPanel rendered full-screen in the content area (replaces More panel)
  - **Export JSON**: Button
  - **Print**: Button
- Styled consistently with SettingsPanel aesthetic (toggle switches, leather/paper theme)

### App component render logic (mobile)
```
if (isMobile) {
  return (
    <div className="h-full flex flex-col">
      <TopBar ... />  {/* compact mobile version */}
      <div className="flex-1 overflow-y-auto">
        {mobileTab === 'roster' && <MobileRoster ... />}
        {mobileTab === 'editor' && (selectedChar ? <CharacterEditor ... /> : <EmptyState ... />)}
        {mobileTab === 'perks' && <PerkBrowser ... />}
        {mobileTab === 'campaign' && <CampaignPanel ... />}
        {mobileTab === 'more' && <MorePanel ... />}
      </div>
      <MobileTabBar activeTab={mobileTab} onTabChange={setMobileTab} hasSelectedChar={!!selectedChar} />
      <PrintableRoster ... />
    </div>
  );
}
// else: current desktop layout unchanged
```

### New components
1. **`MobileTabBar`** — 5-icon tab bar, fixed bottom, `no-print sm:hidden`
2. **`MobileRoster`** — Full-screen character list (extracts list rendering from RosterSidebar)
3. **`MorePanel`** — Genre, toggles, associates, settings, export, print

### What stays unchanged
- Desktop layout (sidebar + center panel + TopBar) — zero changes above 640px
- PrintableRoster layout — only content additions (rule descriptions)
- All existing components (CharacterEditor, PerkBrowser, CampaignPanel, AssociateBuilder, SettingsPanel) — they already render full-width within their container
- Data model / persistence / JSON format — no changes
- LeagueSelector — already mobile-friendly (centered card)
- DataLoader — already mobile-friendly
