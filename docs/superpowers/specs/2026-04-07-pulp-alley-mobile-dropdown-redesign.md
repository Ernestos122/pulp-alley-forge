# Pulp Alley Forge: Mobile Dropdown Menu Redesign

**Date:** 2026-04-07
**File:** `index.html`

## Overview

Replace the broken bottom tab bar mobile implementation with a dropdown menu panel approach. Use CSS media queries instead of React conditionals for layout switching. Single render path for both desktop and mobile.

## What Gets Removed

- `MobileTabBar` component (entire section 5m)
- `MobileRoster` component (entire section 5m2)
- `MorePanel` component (entire section 5m3)
- `useIsMobile` hook
- `isMobile` state variable in App
- `mobileTab` state and handlers: `handleMobileTabChange`, `handleMobileSelectChar`, `handleMoreNavigate`
- `renderMobileContent()` function in App
- All `isMobile ? ... : ...` conditional rendering in App
- `isMobile` prop from TopBar

## CSS Media Queries

### Breakpoint: 768px

Add to the existing `<style>` block:

```css
@media (max-width: 768px) {
    .desktop-sidebar { display: none !important; }
    .mobile-menu-btn { display: flex !important; }
    .mobile-dropdown { display: none; }
    .mobile-dropdown.open { display: block; }
    .content-area { min-width: 0 !important; }
}
@media (min-width: 769px) {
    .mobile-menu-btn { display: none !important; }
    .mobile-dropdown { display: none !important; }
}
```

### Touch targets
```css
@media (max-width: 768px) {
    select, button, input { min-height: 44px; }
}
```

## TopBar Changes

### New state
Add `mobileMenuOpen` state: `const [mobileMenuOpen, setMobileMenuOpen] = useState(false)`

This can live in the App component and be passed as a prop, or in the TopBar itself.

### Props
Remove `isMobile` prop. Add: `mobileMenuOpen`, `onToggleMobileMenu`, `onMobileNavigate`.

### Mobile rendering (always in DOM, shown/hidden via CSS)

After the existing TopBar row, add a `mobile-dropdown` div:

```
<div className={`mobile-dropdown ${mobileMenuOpen ? 'open' : ''}`}>
    - Genre selector (full-width, py-3 for 44px target)
    - Weapons / Supers toggle pills (py-2, larger text)
    - 2-column grid of nav buttons:
        Row 1: Roster | Perks
        Row 2: Associates | Campaign
        Row 3: Settings | (Export + Print)
    - Each nav button: onClick → onMobileNavigate(viewName) → sets activeView + closes menu
</div>
```

### Desktop rendering (unchanged)
The existing genre selector, Weapons/Supers toggles, Export, Print buttons remain in the TopBar row — shown on desktop, hidden on mobile. Remove the `{!isMobile && ...}` guards and replace with CSS class `hidden md:flex` pattern or simply always render (CSS handles visibility).

Actually, to keep it simple: always render the desktop controls in the TopBar row, but add a class like `desktop-only` that hides them at ≤768px. The mobile dropdown has its own copies of these controls styled for touch.

## App Layout Changes

### Single render path

Replace the `isMobile ? renderMobileContent() : (sidebar + centerPanel)` with:

```jsx
<div className="flex-1 overflow-hidden flex no-print">
    <RosterSidebar className="desktop-sidebar" ... />
    {renderCenterPanel()}
</div>
```

The sidebar gets `desktop-sidebar` class which CSS hides on mobile. The center panel always takes remaining space.

### New `activeView: 'roster'`

Add `'roster'` as a valid `activeView` value. In `renderCenterPanel()`:

```jsx
if (activeView === 'roster') {
    return <MobileCharacterList ... />;
}
```

This is a simple full-screen character list (not the old complex MobileRoster — just a straightforward list with character cards, slot bar, and add button).

### MobileCharacterList component

New lightweight component (replaces MobileRoster). Renders:
- SlotBar
- Character cards grouped by type (Leader, Epic, Sidekick, Ally, Follower, Gang)
- Each card: tap → `onSelectChar(id)` which sets `selectedCharId` + `activeView: 'character'`
- Add Character button at bottom
- Minimum 44px touch targets on all cards

### Character editor back navigation

When `activeView === 'character'` and a character is selected, the `CharacterEditor` renders full-screen in the center panel (same as desktop). On mobile, since sidebar is hidden, add a back button row above the editor:

```jsx
{activeView === 'character' && selectedChar && (
    <div className="mobile-menu-btn p-2" style={{...}}>
        <button onClick={() => { setActiveView('roster'); setSelectedCharId(null); }}>
            ← Back to Roster
        </button>
    </div>
)}
```

This uses `mobile-menu-btn` display class so it only shows on mobile.

### Dropdown navigation handler

```jsx
const handleMobileNav = (view) => {
    setActiveView(view === 'roster' ? 'roster' : view);
    setSelectedCharId(null);
    setMobileMenuOpen(false);
};
```

Called from each dropdown nav button.

## What Stays Unchanged

- Desktop layout (sidebar + center panel) — zero changes
- RosterSidebar component internals
- CharacterEditor component
- PerkBrowser, AssociateBuilder, CampaignPanel, SettingsPanel
- PrintableRoster
- Data model / JSON
- Print functionality
- LeagueSelector, DataLoader

## Component Summary

| Component | Action |
|---|---|
| `useIsMobile` | DELETE |
| `MobileTabBar` | DELETE |
| `MobileRoster` | DELETE (replaced by `MobileCharacterList`) |
| `MorePanel` | DELETE (replaced by dropdown panel in TopBar) |
| `MobileCharacterList` | NEW — simple character list for mobile roster view |
| `TopBar` | MODIFY — add hamburger + dropdown panel, remove isMobile prop |
| `App` | MODIFY — single render path, new mobileMenuOpen state, add 'roster' activeView |
| CSS | MODIFY — add media queries for sidebar hide, dropdown show/hide, touch targets |
