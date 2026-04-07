# Mobile Dropdown Menu Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the broken bottom tab bar mobile layout with a CSS-driven dropdown menu panel approach — hamburger menu in TopBar, full-screen views, 44px touch targets, single React render path.

**Architecture:** CSS media queries at 768px handle layout switching (hide sidebar, show hamburger/dropdown). React renders one layout — CSS controls visibility. Old mobile components (MobileTabBar, MobileRoster, MorePanel, useIsMobile) are deleted. TopBar gains a dropdown panel with navigation + settings. New `MobileCharacterList` component for roster view. App uses single render path with `activeView: 'roster'` for mobile character list.

**Tech Stack:** React 18, Tailwind CSS (CDN), inline JSX/Babel — single file `index.html`

**Working directory:** `C:\Users\soyer\Downloads\Blitz_Tactics\PulpAlley`

---

### Task 1: Add CSS Media Queries

**Files:**
- Modify: `index.html:62-76` (CSS section)

- [ ] **Step 1: Add mobile CSS rules after the `.print-only { display: none; }` line (line 63)**

After line 63 (`.print-only { display: none; }`), add:

```css
/* Mobile Layout */
@media (max-width: 768px) {
    .desktop-sidebar { display: none !important; }
    .desktop-only { display: none !important; }
    .mobile-menu-btn { display: flex !important; }
    .mobile-dropdown { display: none; }
    .mobile-dropdown.open { display: block; }
    select, button:not(.no-min-height), input { min-height: 44px; }
    .mobile-back-btn { display: flex !important; }
}
@media (min-width: 769px) {
    .mobile-menu-btn { display: none !important; }
    .mobile-dropdown { display: none !important; }
    .mobile-back-btn { display: none !important; }
}
```

- [ ] **Step 2: Verify no syntax errors by refreshing**

Expected: Page loads. Desktop layout unchanged.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat(pulp-alley): add CSS media queries for mobile layout at 768px breakpoint"
```

---

### Task 2: Delete Old Mobile Components and useIsMobile Hook

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Delete the `useIsMobile` hook (lines 88-97)**

Remove the entire hook from:
```jsx
const useIsMobile = (breakpoint = 640) => {
```
through the closing `};`

- [ ] **Step 2: Delete `MobileTabBar` component (section 5m, starts at line ~3185)**

Remove the entire section from `// 5m. MOBILE TAB BAR` through its closing `};`

- [ ] **Step 3: Delete `MobileRoster` component (section 5m2)**

Remove the entire section from `// 5m2. MOBILE ROSTER` through its closing `};`

- [ ] **Step 4: Delete `MorePanel` component (section 5m3)**

Remove the entire section from `// 5m3. MORE PANEL` through its closing `};`

- [ ] **Step 5: Verify no syntax errors**

Expected: Page will have errors since App still references deleted components. That's OK — Task 4 fixes App.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "refactor(pulp-alley): remove old mobile components (MobileTabBar, MobileRoster, MorePanel, useIsMobile)"
```

---

### Task 3: Rewrite TopBar with Dropdown Menu Panel

**Files:**
- Modify: `index.html` — TopBar component (starts at line ~668)

- [ ] **Step 1: Replace the entire TopBar component**

Replace the TopBar component (from `const TopBar =` through its closing `);`) with:

```jsx
const TopBar = ({ league, updateLeague, onBack, gameData, mobileMenuOpen, onToggleMobileMenu, onMobileNavigate }) => (
    <div className="no-print" style={{background:'var(--leather)', borderBottom:'3px solid var(--crimson)'}}>
        <div className="p-2 flex items-center justify-between">
            <div className="flex items-center gap-3">
                <button onClick={onBack} className="cursor-pointer hover:opacity-80" style={{color:'var(--paper)', background:'transparent', border:'none'}}>
                    <Icons.ArrowLeft size={18} />
                </button>
                <span className="typewriter text-sm tracking-widest desktop-only" style={{color:'var(--crimson)'}}>PULP ALLEY — LEAGUE DOSSIER</span>
            </div>
            <div className="flex items-center gap-2">
                <input value={league.name} onChange={(e) => updateLeague(l => ({...l, name: e.target.value}))}
                    className="bg-transparent border-b typewriter text-sm px-1 outline-none" style={{color:'var(--paper)', borderColor:'var(--crimson)', width:'180px', flex:'1', minWidth:'100px', maxWidth:'200px'}} />
                {/* Desktop controls */}
                <select
                    value={league.genre || ''}
                    onChange={(e) => updateLeague(l => ({...l, genre: e.target.value || null}))}
                    className="typewriter text-xs px-2 py-1 rounded cursor-pointer outline-none desktop-only no-min-height"
                    style={{background:'var(--ink)', color:'var(--paper)', border:'1px solid var(--ink-faded)'}}>
                    <option value="">Classic Pulp</option>
                    {(gameData?.genres || []).map(g => <option key={g.name} value={g.name}>{g.name}</option>)}
                </select>
                <button
                    onClick={() => updateLeague(l => ({...l, optionalRules: {...(l.optionalRules || {}), weapons: !l.optionalRules?.weapons}}))}
                    title="Weapons"
                    className="px-2 py-1 rounded typewriter text-xs cursor-pointer hover:opacity-80 transition-all desktop-only no-min-height"
                    style={{
                        background: league.optionalRules?.weapons ? 'var(--crimson)' : 'var(--ink)',
                        color: 'var(--paper)',
                        border: league.optionalRules?.weapons ? '1px solid var(--crimson)' : '1px solid var(--ink-faded)',
                    }}>
                    Weapons
                </button>
                <button
                    onClick={() => updateLeague(l => ({...l, optionalRules: {...(l.optionalRules || {}), supers: !l.optionalRules?.supers}}))}
                    title="Supers"
                    className="px-2 py-1 rounded typewriter text-xs cursor-pointer hover:opacity-80 transition-all desktop-only no-min-height"
                    style={{
                        background: league.optionalRules?.supers ? 'var(--crimson)' : 'var(--ink)',
                        color: 'var(--paper)',
                        border: league.optionalRules?.supers ? '1px solid var(--crimson)' : '1px solid var(--ink-faded)',
                    }}>
                    Supers
                </button>
                <button onClick={() => exportLeagueJSON(league)} className="cursor-pointer p-1 rounded hover:opacity-80 desktop-only no-min-height" style={{color:'var(--paper)'}} title="Export JSON">
                    <Icons.Download size={16} />
                </button>
                <button onClick={() => window.print()} className="cursor-pointer p-1 rounded hover:opacity-80 desktop-only no-min-height" style={{color:'var(--paper)'}} title="Print">
                    <Icons.Printer size={16} />
                </button>
                {/* Mobile hamburger */}
                <button onClick={onToggleMobileMenu} className="mobile-menu-btn cursor-pointer p-1.5 rounded items-center no-min-height"
                    style={{color:'var(--paper)', background: mobileMenuOpen ? 'var(--crimson)' : 'rgba(139,26,26,0.4)', border:'1px solid var(--crimson)', display:'none'}}>
                    {mobileMenuOpen ? <Icons.X size={18} /> : <Icons.MoreHorizontal size={18} />}
                </button>
            </div>
        </div>
        {/* Desktop genre pills */}
        {league.genre && (() => {
            const genre = (gameData?.genres || []).find(g => g.name === league.genre);
            if (!genre) return null;
            return (
                <div className="px-3 pb-2 flex flex-wrap gap-2 desktop-only">
                    {(genre.modifiers || []).map(mod => (
                        <span key={mod.name} className="typewriter text-xs px-2 py-0.5 rounded" style={{background:'rgba(139,26,26,0.3)', color:'var(--paper)', border:'1px solid rgba(139,26,26,0.5)'}}>
                            {mod.name}: <span style={{color:'var(--border)'}}>{mod.description.length > 60 ? mod.description.slice(0,60) + '...' : mod.description}</span>
                        </span>
                    ))}
                </div>
            );
        })()}
        {/* Mobile dropdown panel */}
        <div className={`mobile-dropdown ${mobileMenuOpen ? 'open' : ''}`} style={{background:'var(--leather)', padding:'8px 12px', borderTop:'1px solid var(--ink-faded)'}}>
            {/* Genre selector */}
            <select
                value={league.genre || ''}
                onChange={(e) => updateLeague(l => ({...l, genre: e.target.value || null}))}
                className="w-full typewriter text-sm px-3 py-2 rounded cursor-pointer outline-none mb-2"
                style={{background:'var(--ink)', color:'var(--paper)', border:'1px solid var(--ink-faded)'}}>
                <option value="">Classic Pulp</option>
                {(gameData?.genres || []).map(g => <option key={g.name} value={g.name}>{g.name}</option>)}
            </select>
            {/* Weapons / Supers toggles */}
            <div className="flex gap-2 mb-3">
                <button
                    onClick={() => updateLeague(l => ({...l, optionalRules: {...(l.optionalRules || {}), weapons: !l.optionalRules?.weapons}}))}
                    className="flex-1 py-2 rounded typewriter text-xs cursor-pointer hover:opacity-80 transition-all no-min-height"
                    style={{
                        background: league.optionalRules?.weapons ? 'var(--crimson)' : 'var(--ink)',
                        color: 'var(--paper)',
                        border: league.optionalRules?.weapons ? '1px solid var(--crimson)' : '1px solid var(--ink-faded)',
                    }}>
                    {league.optionalRules?.weapons ? '✓ ' : ''}Weapons
                </button>
                <button
                    onClick={() => updateLeague(l => ({...l, optionalRules: {...(l.optionalRules || {}), supers: !l.optionalRules?.supers}}))}
                    className="flex-1 py-2 rounded typewriter text-xs cursor-pointer hover:opacity-80 transition-all no-min-height"
                    style={{
                        background: league.optionalRules?.supers ? 'var(--crimson)' : 'var(--ink)',
                        color: 'var(--paper)',
                        border: league.optionalRules?.supers ? '1px solid var(--crimson)' : '1px solid var(--ink-faded)',
                    }}>
                    {league.optionalRules?.supers ? '✓ ' : ''}Supers
                </button>
            </div>
            {/* Navigation grid */}
            <div className="grid grid-cols-2 gap-2 mb-2">
                {[
                    { key: 'roster', label: 'Roster', icon: <Icons.Users size={14} /> },
                    { key: 'perks', label: 'Perks', icon: <Icons.Star size={14} /> },
                    { key: 'associates', label: 'Associates', icon: <Icons.Users size={14} /> },
                    { key: 'campaign', label: 'Campaign', icon: <Icons.FileText size={14} /> },
                    { key: 'settings', label: 'Settings', icon: <Icons.Edit2 size={14} /> },
                ].map(({ key, label, icon }) => (
                    <button key={key} onClick={() => onMobileNavigate(key)}
                        className="py-3 rounded typewriter text-xs cursor-pointer hover:opacity-80 flex items-center justify-center gap-1 no-min-height"
                        style={{background:'var(--paper)', color:'var(--ink)', border:'1px solid var(--border)'}}>
                        {icon} {label}
                    </button>
                ))}
                <div className="flex gap-2">
                    <button onClick={() => { exportLeagueJSON(league); onToggleMobileMenu(); }}
                        className="flex-1 py-3 rounded typewriter text-xs cursor-pointer hover:opacity-80 flex items-center justify-center gap-1 no-min-height"
                        style={{background:'var(--ink)', color:'var(--paper)', border:'none'}}>
                        <Icons.Download size={14} /> Export
                    </button>
                    <button onClick={() => { onToggleMobileMenu(); setTimeout(() => window.print(), 100); }}
                        className="flex-1 py-3 rounded typewriter text-xs cursor-pointer hover:opacity-80 flex items-center justify-center gap-1 no-min-height"
                        style={{background:'var(--ink)', color:'var(--paper)', border:'none'}}>
                        <Icons.Printer size={14} /> Print
                    </button>
                </div>
            </div>
        </div>
    </div>
);
```

- [ ] **Step 2: Verify no syntax errors (page won't fully work yet — App still references old props)**

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat(pulp-alley): rewrite TopBar with mobile dropdown menu panel and CSS visibility"
```

---

### Task 4: Add MobileCharacterList Component

**Files:**
- Modify: `index.html` — insert before the LeagueSelector component (section 6)

- [ ] **Step 1: Add MobileCharacterList component**

Find the line `// 6. LEAGUE SELECTOR COMPONENT`. Insert before it:

```jsx
// ==========================================
// 5n. MOBILE CHARACTER LIST
// ==========================================
const MobileCharacterList = ({ league, gameData, onSelectChar, onOpenAddModal }) => {
    const used = getUsedSlots(league, gameData);
    const total = getTotalSlots(league, gameData);
    const groups = [
        { title: 'Leader', chars: league.characters.filter(c => c.type === 'leader') },
        { title: 'Epic', chars: league.characters.filter(c => c.type === 'epic') },
        { title: 'Sidekicks', chars: league.characters.filter(c => c.type === 'sidekick') },
        { title: 'Allies', chars: league.characters.filter(c => c.type === 'ally') },
        { title: 'Followers', chars: league.characters.filter(c => c.type === 'follower') },
        { title: 'Gangs', chars: league.characters.filter(c => c.type === 'gang') },
    ];
    return (
        <div className="flex-1 overflow-y-auto p-4" style={{background:'var(--paper)'}}>
            <SlotBar used={used} total={total} />
            <div className="typewriter text-xs uppercase tracking-wider mb-3" style={{color:'var(--crimson)'}}>Field Operatives</div>
            {groups.map(({ title, chars }) => {
                if (chars.length === 0) return null;
                return (
                    <div key={title} className="mb-3">
                        <div className="typewriter text-xs uppercase tracking-wider mb-1" style={{color:'var(--ink-faded)'}}>{title}</div>
                        {chars.map(c => {
                            const tmpl = getTemplate(gameData, c.type);
                            const hasEmptyName = !c.name || c.name.trim() === '';
                            const hasUnassignedSkills = c.type !== 'gang' && c.type !== 'follower' && SKILLS.some(s => c.skillAssignments?.[s]?.count == null || c.skillAssignments?.[s]?.diceType == null);
                            const incomplete = hasEmptyName || hasUnassignedSkills;
                            return (
                                <div key={c.id} onClick={() => onSelectChar(c.id)}
                                    className="p-3 mb-2 rounded cursor-pointer transition-all hover:opacity-90"
                                    style={{background:'var(--paper-dark)', border:'1px solid var(--border)', borderLeft:`4px solid ${LEVEL_COLORS[c.type]}`}}>
                                    <div className="flex items-center justify-between">
                                        <div className="flex items-center gap-2">
                                            {incomplete && <Icons.AlertTriangle size={14} style={{color:'var(--crimson)'}} />}
                                            <div>
                                                <div className="typewriter text-xs uppercase" style={{color: LEVEL_COLORS[c.type]}}>{LEVEL_LABELS[c.type]}</div>
                                                <div className="brand-font text-sm font-bold" style={{color:'var(--ink)'}}>{c.name || 'Unnamed'}</div>
                                            </div>
                                        </div>
                                        {tmpl && <span className="typewriter text-xs" style={{color:'var(--ink-faded)'}}>Health: {tmpl.health}</span>}
                                    </div>
                                </div>
                            );
                        })}
                    </div>
                );
            })}
            {league.characters.length === 0 && (
                <div className="text-center py-8">
                    <Icons.Users size={48} className="mx-auto mb-4" style={{color:'var(--ink-faded)'}} />
                    <p className="typewriter text-sm" style={{color:'var(--ink-faded)'}}>No operatives yet. Tap "Add" below to get started.</p>
                </div>
            )}
            <button onClick={onOpenAddModal}
                className="w-full py-3 rounded typewriter text-sm cursor-pointer hover:opacity-80 flex items-center justify-center gap-1 mt-2"
                style={{background:'var(--leather)', color:'var(--paper)', border:'none'}}>
                <Icons.Plus size={14} /> Add Character
            </button>
        </div>
    );
};
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "feat(pulp-alley): add MobileCharacterList component for mobile roster view"
```

---

### Task 5: Rewrite App Component — Single Render Path

**Files:**
- Modify: `index.html` — App component (section 7, starts at line ~3432)

- [ ] **Step 1: Replace App state and handlers**

In the App component, replace lines from `const isMobile = useIsMobile();` through `const handleMoreNavigate = ...` (the mobile state and all mobile handlers) with:

```jsx
const [mobileMenuOpen, setMobileMenuOpen] = useState(false);
```

Keep `showToast`, `updateLeague`, `addCharacter`, `updateCharacter`, `deleteCharacter`, `handleSelectChar`, `handleSetView` unchanged.

- [ ] **Step 2: Add mobile navigation handler**

After `handleSetView`, add:

```jsx
const handleMobileNavigate = useCallback((view) => {
    setActiveView(view === 'roster' ? 'roster' : view);
    if (view !== 'character') setSelectedCharId(null);
    setMobileMenuOpen(false);
}, []);
```

- [ ] **Step 3: Add 'roster' case to renderCenterPanel**

At the top of `renderCenterPanel`, before the `perks` check, add:

```jsx
if (activeView === 'roster') {
    return <MobileCharacterList league={league} gameData={gameData} onSelectChar={handleSelectChar} onOpenAddModal={() => setShowAddModal(true)} />;
}
```

- [ ] **Step 4: Replace the entire return block**

Delete `renderMobileContent` entirely. Replace the return block (from `return (` to the final `);`) with:

```jsx
return (
    <div className="h-full flex flex-col" style={{background:'var(--paper)'}}>
        <TopBar league={league} updateLeague={updateLeague} gameData={gameData}
            mobileMenuOpen={mobileMenuOpen}
            onToggleMobileMenu={() => setMobileMenuOpen(prev => !prev)}
            onMobileNavigate={handleMobileNavigate}
            onBack={() => { setShowLeagueSelector(true); setSelectedCharId(null); setActiveView(null); setMobileMenuOpen(false); }} />

        <div className="flex-1 overflow-hidden flex no-print">
            <RosterSidebar className="desktop-sidebar"
                league={league} gameData={gameData}
                selectedCharId={selectedCharId}
                onSelectChar={handleSelectChar}
                onOpenAddModal={() => setShowAddModal(true)}
                activeView={activeView}
                onSetView={handleSetView}
            />
            <div className="flex-1 overflow-y-auto">
                {/* Mobile back button — shown only when viewing a character on mobile */}
                {activeView === 'character' && selectedChar && (
                    <div className="mobile-back-btn p-2 items-center gap-2" style={{background:'var(--paper-dark)', borderBottom:'1px solid var(--border)', display:'none'}}>
                        <button onClick={() => { setActiveView('roster'); setSelectedCharId(null); }}
                            className="cursor-pointer hover:opacity-80 flex items-center gap-1 no-min-height"
                            style={{color:'var(--crimson)', background:'transparent', border:'none'}}>
                            <Icons.ArrowLeft size={16} />
                            <span className="typewriter text-xs">Back to Roster</span>
                        </button>
                    </div>
                )}
                {renderCenterPanel()}
            </div>
        </div>

        <PrintableRoster league={league} gameData={gameData} />

        {showAddModal && (
            <AddCharacterModal
                league={league} gameData={gameData}
                onAdd={addCharacter}
                onClose={() => setShowAddModal(false)}
            />
        )}

        {toast && <div className="fixed top-4 right-4 px-4 py-2 rounded shadow-lg typewriter text-sm toast z-50" style={{background: toast.type==='error'?'var(--crimson)':'var(--leather)', color:'var(--paper)'}}>{toast.msg}</div>}
    </div>
);
```

- [ ] **Step 5: Pass className to RosterSidebar**

The RosterSidebar component needs to accept and apply the `className` prop. Find the RosterSidebar return (line ~808):

```jsx
<div className="w-60 overflow-y-auto p-3 no-print flex flex-col" style={{background:'var(--paper-dark)', borderRight:'2px solid var(--border)'}}>
```

The component signature is `const RosterSidebar = ({ league, gameData, selectedCharId, onSelectChar, onOpenAddModal, activeView, onSetView }) => {`. Add `className` to the destructured props:

```jsx
const RosterSidebar = ({ league, gameData, selectedCharId, onSelectChar, onOpenAddModal, activeView, onSetView, className }) => {
```

And update the outer div to include the className:

```jsx
<div className={`w-60 overflow-y-auto p-3 no-print flex flex-col ${className || ''}`} style={{background:'var(--paper-dark)', borderRight:'2px solid var(--border)'}}>
```

- [ ] **Step 6: Verify page loads without errors**

Open in browser. Desktop should work identically. Resize to < 768px: sidebar should hide, hamburger appears, dropdown works, navigation works.

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat(pulp-alley): rewrite App with single render path, CSS-driven mobile layout, dropdown navigation"
```

---

### Task 6: Final Verification and Push

- [ ] **Step 1: Desktop verification (> 768px)**

1. Sidebar visible, all TopBar controls visible
2. No hamburger icon
3. Genre selector, Weapons, Supers, Export, Print all in TopBar
4. Character editor, Perks, Associates, Campaign, Settings all work

- [ ] **Step 2: Mobile verification (< 768px)**

1. Sidebar hidden
2. Hamburger icon visible in TopBar
3. Tap hamburger: dropdown panel opens with genre, weapons/supers, nav grid, export/print
4. Tap Roster: full-screen character list
5. Tap character: full-screen editor with "Back to Roster" bar at top
6. Tap Perks/Associates/Campaign/Settings: full-screen view
7. All buttons/inputs have 44px minimum touch target
8. Content area scrolls properly
9. Print button works from dropdown
10. Export button works from dropdown

- [ ] **Step 3: Push**

```bash
git push origin main:master
```
