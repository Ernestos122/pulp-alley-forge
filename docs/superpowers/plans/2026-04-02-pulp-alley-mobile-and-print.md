# Pulp Alley Mobile Responsive + Print Rules Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add full rule descriptions to the print sheet, and make the league builder fully usable on mobile with a bottom tab bar and full-screen views.

**Architecture:** Single-file changes to `PulpAlley/PulpAlley_Forge.html`. A `useIsMobile()` hook drives conditional rendering — mobile gets a bottom tab bar + full-screen views while desktop layout stays untouched. Print sheet gets expanded rule description blocks.

**Tech Stack:** React 18, Tailwind CSS (CDN), inline JSX/Babel

---

### Task 1: Print Sheet — Full Genre Modifier Descriptions

**Files:**
- Modify: `PulpAlley/PulpAlley_Forge.html:2522-2526`

- [ ] **Step 1: Replace genre modifier names-only with name + description blocks**

In the `PrintableRoster` component (~line 2522), replace:

```jsx
{activeGenre && (
    <div style={{fontSize:'10px', fontFamily:'Special Elite, monospace', marginTop:'4px', color:'#555'}}>
        Genre Rules: {activeGenre.modifiers.map(m => m.name).join(', ')}
    </div>
)}
```

With:

```jsx
{activeGenre && (
    <div style={{fontSize:'10px', fontFamily:'Special Elite, monospace', marginTop:'4px', color:'#555'}}>
        <div style={{fontWeight:'bold', marginBottom:'2px'}}>Genre: {activeGenre.name}</div>
        {activeGenre.modifiers.map(mod => (
            <div key={mod.name} style={{marginLeft:'12px', marginBottom:'2px'}}>
                <span style={{fontWeight:'bold'}}>{mod.name}</span> — {mod.description}
            </div>
        ))}
    </div>
)}
```

- [ ] **Step 2: Verify by opening in browser, selecting a genre (e.g. Pulp Westerns), and using Print Preview (Ctrl+P)**

Expected: Genre section shows "Genre: Pulp Westerns" with "Riding the Range — All characters may begin..." and "Gritty — In this setting, all dice-types over d8..." each on their own line, indented.

- [ ] **Step 3: Commit**

```bash
git add PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add full genre modifier descriptions to print sheet"
```

---

### Task 2: Print Sheet — Full Optional Rule Descriptions

**Files:**
- Modify: `PulpAlley/PulpAlley_Forge.html:2527-2541`

- [ ] **Step 1: Replace optional rules labels-only with label + description blocks**

In `PrintableRoster` (~line 2527), replace the entire optional rules IIFE:

```jsx
{(() => {
    const rules = league.optionalRules || {};
    const allSettings = [
        { key: 'weapons', label: 'Weapons' },
        { key: 'supers', label: 'Supers' },
        ...(gameData.optionalSettings || []).map(s => ({ key: s.name, label: s.label })),
    ];
    const active = allSettings.filter(s => rules[s.key]);
    if (active.length === 0) return null;
    return (
        <div style={{fontSize:'10px', fontFamily:'Special Elite, monospace', marginTop:'4px', color:'#555'}}>
            Optional Rules: {active.map(s => s.label).join(', ')}
        </div>
    );
})()}
```

With:

```jsx
{(() => {
    const rules = league.optionalRules || {};
    const coreDescriptions = {
        weapons: 'Characters may equip weapons (Ranged and Melee). Each character may carry one weapon.',
        supers: 'Allows Epic characters in the league roster. Epic characters cost 5 roster slots and have access to Epic Abilities.',
    };
    const allSettings = [
        { key: 'weapons', label: 'Weapons', description: coreDescriptions.weapons },
        { key: 'supers', label: 'Supers', description: coreDescriptions.supers },
        ...(gameData.optionalSettings || []).map(s => ({ key: s.name, label: s.label, description: s.description })),
    ];
    const active = allSettings.filter(s => rules[s.key]);
    if (active.length === 0) return null;
    return (
        <div style={{fontSize:'10px', fontFamily:'Special Elite, monospace', marginTop:'4px', color:'#555'}}>
            <div style={{fontWeight:'bold', marginBottom:'2px'}}>Optional Rules:</div>
            {active.map(s => (
                <div key={s.key} style={{marginLeft:'12px', marginBottom:'2px'}}>
                    <span style={{fontWeight:'bold'}}>{s.label}</span> — {s.description}
                </div>
            ))}
        </div>
    );
})()}
```

- [ ] **Step 2: Verify by enabling Weapons + Single-Shot in the app, then Print Preview**

Expected: "Optional Rules:" header, then each rule on its own line with bold label and full description.

- [ ] **Step 3: Commit**

```bash
git add PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add full optional rule descriptions to print sheet"
```

---

### Task 3: Add `useIsMobile` Hook and New Icons

**Files:**
- Modify: `PulpAlley/PulpAlley_Forge.html:86` (after useState import), `PulpAlley/PulpAlley_Forge.html:112` (Icons object)

- [ ] **Step 1: Add `useIsMobile` hook after the React imports (~line 86)**

After the line:
```jsx
const { useState, useRef, useEffect, useCallback } = React;
```

Add:

```jsx
const useIsMobile = (breakpoint = 640) => {
    const [isMobile, setIsMobile] = useState(() => window.innerWidth < breakpoint);
    useEffect(() => {
        const mql = window.matchMedia(`(max-width: ${breakpoint - 1}px)`);
        const handler = (e) => setIsMobile(e.matches);
        mql.addEventListener('change', handler);
        return () => mql.removeEventListener('change', handler);
    }, [breakpoint]);
    return isMobile;
};
```

- [ ] **Step 2: Add Star and MoreHorizontal icons to the Icons object (~line 112)**

Before the closing `};` of the Icons object, add:

```jsx
Star: createIcon(<polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"/>),
MoreHorizontal: createIcon(<><circle cx="12" cy="12" r="1"/><circle cx="19" cy="12" r="1"/><circle cx="5" cy="12" r="1"/></>),
Settings: createIcon(<><circle cx="12" cy="12" r="3"/><path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06-.06A1.65 1.65 0 0 0 4.68 15a1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06A1.65 1.65 0 0 0 9 4.68a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06A1.65 1.65 0 0 0 19.4 9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z"/></>),
```

- [ ] **Step 3: Verify no syntax errors by refreshing the page in browser**

Expected: Page loads normally, no console errors.

- [ ] **Step 4: Commit**

```bash
git add PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add useIsMobile hook and new icons for mobile UI"
```

---

### Task 4: Make TopBar Responsive

**Files:**
- Modify: `PulpAlley/PulpAlley_Forge.html:632-696` (TopBar component)

- [ ] **Step 1: Add `isMobile` prop to TopBar and conditionalize controls**

Replace the entire TopBar component (lines 632-696):

```jsx
const TopBar = ({ league, updateLeague, onBack, gameData }) => (
    <div className="no-print" style={{background:'var(--leather)', borderBottom:'3px solid var(--crimson)'}}>
        <div className="p-2 flex items-center justify-between">
            <div className="flex items-center gap-3">
                <button onClick={onBack} className="cursor-pointer hover:opacity-80" style={{color:'var(--paper)'}}>
                    <Icons.ArrowLeft size={18} />
                </button>
                <span className="typewriter text-sm tracking-widest hidden sm:inline" style={{color:'var(--crimson)'}}>PULP ALLEY — LEAGUE DOSSIER</span>
            </div>
            <div className="flex items-center gap-2">
                <input value={league.name} onChange={(e) => updateLeague(l => ({...l, name: e.target.value}))}
                    className="bg-transparent border-b typewriter text-sm px-1 outline-none" style={{color:'var(--paper)', borderColor:'var(--crimson)', width:'180px'}} />
                <select
                    value={league.genre || ''}
                    onChange={(e) => updateLeague(l => ({...l, genre: e.target.value || null}))}
                    className="typewriter text-xs px-2 py-1 rounded cursor-pointer outline-none"
                    style={{background:'var(--ink)', color:'var(--paper)', border:'1px solid var(--ink-faded)'}}>
                    <option value="">Classic Pulp</option>
                    {(gameData?.genres || []).map(g => <option key={g.name} value={g.name}>{g.name}</option>)}
                </select>
                <button
                    onClick={() => updateLeague(l => ({...l, optionalRules: {...(l.optionalRules || {}), weapons: !l.optionalRules?.weapons}}))}
                    title="Weapons optional rule: characters may equip weapons"
                    className="px-2 py-1 rounded typewriter text-xs cursor-pointer hover:opacity-80 transition-all"
                    style={{
                        background: league.optionalRules?.weapons ? 'var(--crimson)' : 'var(--ink)',
                        color: 'var(--paper)',
                        border: league.optionalRules?.weapons ? '1px solid var(--crimson)' : '1px solid var(--ink-faded)',
                    }}>
                    Weapons
                </button>
                <button
                    onClick={() => updateLeague(l => ({...l, optionalRules: {...(l.optionalRules || {}), supers: !l.optionalRules?.supers}}))}
                    title="Supers optional rule: allows Epic characters"
                    className="px-2 py-1 rounded typewriter text-xs cursor-pointer hover:opacity-80 transition-all"
                    style={{
                        background: league.optionalRules?.supers ? 'var(--crimson)' : 'var(--ink)',
                        color: 'var(--paper)',
                        border: league.optionalRules?.supers ? '1px solid var(--crimson)' : '1px solid var(--ink-faded)',
                    }}>
                    Supers
                </button>
                <button onClick={() => exportLeagueJSON(league)} className="cursor-pointer p-1 rounded hover:opacity-80" style={{color:'var(--paper)'}} title="Export JSON">
                    <Icons.Download size={16} />
                </button>
                <button onClick={() => window.print()} className="cursor-pointer p-1 rounded hover:opacity-80" style={{color:'var(--paper)'}} title="Print">
                    <Icons.Printer size={16} />
                </button>
            </div>
        </div>
        {league.genre && (() => {
            const genre = (gameData?.genres || []).find(g => g.name === league.genre);
            if (!genre) return null;
            return (
                <div className="px-3 pb-2 flex flex-wrap gap-2">
                    {(genre.modifiers || []).map(mod => (
                        <span key={mod.name} className="typewriter text-xs px-2 py-0.5 rounded" style={{background:'rgba(139,26,26,0.3)', color:'var(--paper)', border:'1px solid rgba(139,26,26,0.5)'}}>
                            {mod.name}: <span style={{color:'var(--border)'}}>{mod.description.length > 60 ? mod.description.slice(0,60) + '...' : mod.description}</span>
                        </span>
                    ))}
                </div>
            );
        })()}
    </div>
);
```

With:

```jsx
const TopBar = ({ league, updateLeague, onBack, gameData, isMobile }) => (
    <div className="no-print" style={{background:'var(--leather)', borderBottom:'3px solid var(--crimson)'}}>
        <div className="p-2 flex items-center justify-between">
            <div className="flex items-center gap-3">
                <button onClick={onBack} className="cursor-pointer hover:opacity-80" style={{color:'var(--paper)'}}>
                    <Icons.ArrowLeft size={18} />
                </button>
                {!isMobile && <span className="typewriter text-sm tracking-widest" style={{color:'var(--crimson)'}}>PULP ALLEY — LEAGUE DOSSIER</span>}
            </div>
            <div className="flex items-center gap-2">
                <input value={league.name} onChange={(e) => updateLeague(l => ({...l, name: e.target.value}))}
                    className="bg-transparent border-b typewriter text-sm px-1 outline-none" style={{color:'var(--paper)', borderColor:'var(--crimson)', width: isMobile ? '140px' : '180px'}} />
                {!isMobile && (
                    <>
                        <select
                            value={league.genre || ''}
                            onChange={(e) => updateLeague(l => ({...l, genre: e.target.value || null}))}
                            className="typewriter text-xs px-2 py-1 rounded cursor-pointer outline-none"
                            style={{background:'var(--ink)', color:'var(--paper)', border:'1px solid var(--ink-faded)'}}>
                            <option value="">Classic Pulp</option>
                            {(gameData?.genres || []).map(g => <option key={g.name} value={g.name}>{g.name}</option>)}
                        </select>
                        <button
                            onClick={() => updateLeague(l => ({...l, optionalRules: {...(l.optionalRules || {}), weapons: !l.optionalRules?.weapons}}))}
                            title="Weapons optional rule: characters may equip weapons"
                            className="px-2 py-1 rounded typewriter text-xs cursor-pointer hover:opacity-80 transition-all"
                            style={{
                                background: league.optionalRules?.weapons ? 'var(--crimson)' : 'var(--ink)',
                                color: 'var(--paper)',
                                border: league.optionalRules?.weapons ? '1px solid var(--crimson)' : '1px solid var(--ink-faded)',
                            }}>
                            Weapons
                        </button>
                        <button
                            onClick={() => updateLeague(l => ({...l, optionalRules: {...(l.optionalRules || {}), supers: !l.optionalRules?.supers}}))}
                            title="Supers optional rule: allows Epic characters"
                            className="px-2 py-1 rounded typewriter text-xs cursor-pointer hover:opacity-80 transition-all"
                            style={{
                                background: league.optionalRules?.supers ? 'var(--crimson)' : 'var(--ink)',
                                color: 'var(--paper)',
                                border: league.optionalRules?.supers ? '1px solid var(--crimson)' : '1px solid var(--ink-faded)',
                            }}>
                            Supers
                        </button>
                        <button onClick={() => exportLeagueJSON(league)} className="cursor-pointer p-1 rounded hover:opacity-80" style={{color:'var(--paper)'}} title="Export JSON">
                            <Icons.Download size={16} />
                        </button>
                        <button onClick={() => window.print()} className="cursor-pointer p-1 rounded hover:opacity-80" style={{color:'var(--paper)'}} title="Print">
                            <Icons.Printer size={16} />
                        </button>
                    </>
                )}
            </div>
        </div>
        {!isMobile && league.genre && (() => {
            const genre = (gameData?.genres || []).find(g => g.name === league.genre);
            if (!genre) return null;
            return (
                <div className="px-3 pb-2 flex flex-wrap gap-2">
                    {(genre.modifiers || []).map(mod => (
                        <span key={mod.name} className="typewriter text-xs px-2 py-0.5 rounded" style={{background:'rgba(139,26,26,0.3)', color:'var(--paper)', border:'1px solid rgba(139,26,26,0.5)'}}>
                            {mod.name}: <span style={{color:'var(--border)'}}>{mod.description.length > 60 ? mod.description.slice(0,60) + '...' : mod.description}</span>
                        </span>
                    ))}
                </div>
            );
        })()}
    </div>
);
```

- [ ] **Step 2: Verify by resizing browser below 640px**

Expected: TopBar shows only back arrow + league name. Genre selector, toggles, export, print all hidden. Above 640px everything shows as before.

- [ ] **Step 3: Commit**

```bash
git add PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): make TopBar responsive with mobile-only compact view"
```

---

### Task 5: Create MobileTabBar Component

**Files:**
- Modify: `PulpAlley/PulpAlley_Forge.html` — insert before the `LeagueSelector` component (~line 2728)

- [ ] **Step 1: Add MobileTabBar component**

Insert before the `const LeagueSelector` line (~2728):

```jsx
// ==========================================
// 5m. MOBILE TAB BAR
// ==========================================
const MobileTabBar = ({ activeTab, onTabChange, hasSelectedChar }) => {
    const tabs = [
        { key: 'roster', label: 'Roster', icon: <Icons.Users size={20} /> },
        { key: 'editor', label: 'Editor', icon: <Icons.Edit2 size={20} />, dimmed: !hasSelectedChar },
        { key: 'perks', label: 'Perks', icon: <Icons.Star size={20} /> },
        { key: 'campaign', label: 'Campaign', icon: <Icons.FileText size={20} /> },
        { key: 'more', label: 'More', icon: <Icons.MoreHorizontal size={20} /> },
    ];
    return (
        <div className="no-print flex justify-around items-center py-1" style={{background:'var(--leather)', borderTop:'2px solid var(--crimson)', flexShrink:0}}>
            {tabs.map(tab => {
                const isActive = activeTab === tab.key;
                return (
                    <button key={tab.key} onClick={() => onTabChange(tab.key)}
                        className="flex flex-col items-center gap-0.5 px-2 py-1 cursor-pointer"
                        style={{
                            color: isActive ? 'var(--crimson)' : tab.dimmed ? 'var(--ink-faded)' : 'var(--paper)',
                            opacity: tab.dimmed && !isActive ? 0.4 : 1,
                            background:'transparent', border:'none',
                        }}>
                        {tab.icon}
                        <span className="typewriter" style={{fontSize:'9px'}}>{tab.label}</span>
                    </button>
                );
            })}
        </div>
    );
};
```

- [ ] **Step 2: Verify no syntax errors by refreshing the page**

Expected: Page loads normally. Tab bar not visible yet (not rendered in App).

- [ ] **Step 3: Commit**

```bash
git add PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add MobileTabBar component"
```

---

### Task 6: Create MobileRoster Component

**Files:**
- Modify: `PulpAlley/PulpAlley_Forge.html` — insert after MobileTabBar (before LeagueSelector)

- [ ] **Step 1: Add MobileRoster component**

Insert right after MobileTabBar:

```jsx
// ==========================================
// 5m2. MOBILE ROSTER (full-screen character list)
// ==========================================
const MobileRoster = ({ league, gameData, selectedCharId, onSelectChar, onOpenAddModal }) => {
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
        <div className="flex-1 overflow-y-auto p-3" style={{background:'var(--paper)'}}>
            <SlotBar used={used} total={total} />
            <div className="typewriter text-xs uppercase tracking-wider mb-3" style={{color:'var(--crimson)'}}>Field Operatives</div>
            {groups.map(({ title, chars }) => {
                if (chars.length === 0) return null;
                return (
                    <div key={title} className="mb-3">
                        <div className="typewriter text-xs uppercase tracking-wider mb-1" style={{color:'var(--ink-faded)'}}>{title}</div>
                        {chars.map(c => {
                            const tmpl = getTemplate(gameData, c.type);
                            const isSelected = selectedCharId === c.id;
                            const hasEmptyName = !c.name || c.name.trim() === '';
                            const hasUnassignedSkills = c.type !== 'gang' && c.type !== 'follower' && SKILLS.some(s => c.skillAssignments?.[s]?.count == null || c.skillAssignments?.[s]?.diceType == null);
                            const incomplete = hasEmptyName || hasUnassignedSkills;
                            return (
                                <div key={c.id} onClick={() => onSelectChar(c.id)}
                                    className="p-3 mb-2 rounded cursor-pointer transition-all hover:opacity-90"
                                    style={{
                                        background:'var(--paper-dark)',
                                        border: isSelected ? '2px solid var(--crimson)' : '1px solid var(--border)',
                                        borderLeft: `4px solid ${LEVEL_COLORS[c.type]}`,
                                    }}>
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

- [ ] **Step 2: Verify no syntax errors by refreshing**

Expected: Page loads normally. MobileRoster not rendered yet.

- [ ] **Step 3: Commit**

```bash
git add PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add MobileRoster full-screen component"
```

---

### Task 7: Create MorePanel Component

**Files:**
- Modify: `PulpAlley/PulpAlley_Forge.html` — insert after MobileRoster (before LeagueSelector)

- [ ] **Step 1: Add MorePanel component**

Insert right after MobileRoster:

```jsx
// ==========================================
// 5m3. MORE PANEL (mobile genre/settings/export)
// ==========================================
const MorePanel = ({ league, gameData, updateLeague, onNavigate }) => {
    const rules = league.optionalRules || {};
    return (
        <div className="flex-1 overflow-y-auto p-4" style={{background:'var(--paper)'}}>
            <h2 className="brand-font text-lg font-bold mb-4" style={{color:'var(--ink)'}}>League Settings</h2>

            {/* Genre Selector */}
            <div className="mb-4 p-3 rounded" style={{background:'var(--paper-dark)', border:'1px solid var(--border)'}}>
                <div className="typewriter text-xs uppercase tracking-wider mb-2" style={{color:'var(--crimson)'}}>Genre</div>
                <select
                    value={league.genre || ''}
                    onChange={(e) => updateLeague(l => ({...l, genre: e.target.value || null}))}
                    className="w-full typewriter text-sm px-3 py-2 rounded cursor-pointer outline-none"
                    style={{background:'var(--ink)', color:'var(--paper)', border:'1px solid var(--ink-faded)'}}>
                    <option value="">Classic Pulp</option>
                    {(gameData?.genres || []).map(g => <option key={g.name} value={g.name}>{g.name}</option>)}
                </select>
                {league.genre && (() => {
                    const genre = (gameData?.genres || []).find(g => g.name === league.genre);
                    if (!genre) return null;
                    return (
                        <div className="mt-2 flex flex-wrap gap-1">
                            {(genre.modifiers || []).map(mod => (
                                <span key={mod.name} className="typewriter text-xs px-2 py-0.5 rounded" style={{background:'rgba(139,26,26,0.15)', color:'var(--crimson)', border:'1px solid rgba(139,26,26,0.3)'}}>
                                    {mod.name}
                                </span>
                            ))}
                        </div>
                    );
                })()}
            </div>

            {/* Core Toggles */}
            <div className="mb-4 p-3 rounded" style={{background:'var(--paper-dark)', border:'1px solid var(--border)'}}>
                <div className="typewriter text-xs uppercase tracking-wider mb-2" style={{color:'var(--crimson)'}}>Core Rules</div>
                {[
                    { key: 'weapons', label: 'Weapons', desc: 'Characters may equip weapons' },
                    { key: 'supers', label: 'Supers', desc: 'Allows Epic characters' },
                ].map(({ key, label, desc }) => (
                    <div key={key} className="flex items-center justify-between p-2 mb-1 rounded cursor-pointer hover:opacity-90"
                        style={{background: rules[key] ? 'var(--leather)' : 'var(--paper)', border: `1px solid ${rules[key] ? 'var(--crimson)' : 'var(--border)'}`}}
                        onClick={() => updateLeague(l => ({...l, optionalRules: {...(l.optionalRules || {}), [key]: !l.optionalRules?.[key]}}))}>
                        <div>
                            <div className="brand-font text-sm font-bold" style={{color: rules[key] ? 'var(--paper)' : 'var(--ink)'}}>{label}</div>
                            <div className="typewriter text-xs" style={{color: rules[key] ? 'var(--paper-dark)' : 'var(--ink-faded)'}}>{desc}</div>
                        </div>
                        <div className="w-10 h-6 rounded-full relative flex-shrink-0 ml-2" style={{background: rules[key] ? 'var(--crimson)' : 'var(--border)'}}>
                            <div className="absolute top-0.5 w-5 h-5 rounded-full transition-all" style={{background:'white', left: rules[key] ? '18px' : '2px'}} />
                        </div>
                    </div>
                ))}
            </div>

            {/* Navigation Links */}
            <div className="mb-4 p-3 rounded" style={{background:'var(--paper-dark)', border:'1px solid var(--border)'}}>
                <div className="typewriter text-xs uppercase tracking-wider mb-2" style={{color:'var(--crimson)'}}>More</div>
                {[
                    { key: 'associates', label: 'Associates', count: league.associates.length },
                    { key: 'settings', label: 'Advanced Settings', count: Object.entries(rules).filter(([k,v]) => v && k !== 'weapons' && k !== 'supers').length },
                ].map(({ key, label, count }) => (
                    <button key={key} onClick={() => onNavigate(key)}
                        className="w-full text-left p-2 mb-1 rounded cursor-pointer hover:opacity-90 flex items-center justify-between"
                        style={{background:'var(--paper)', border:'1px solid var(--border)'}}>
                        <span className="brand-font text-sm font-bold" style={{color:'var(--ink)'}}>{label}</span>
                        <span className="typewriter text-xs" style={{color:'var(--ink-faded)'}}>{count > 0 ? `(${count})` : ''} ›</span>
                    </button>
                ))}
            </div>

            {/* Actions */}
            <div className="flex gap-2">
                <button onClick={() => exportLeagueJSON(league)}
                    className="flex-1 py-2 rounded typewriter text-sm cursor-pointer hover:opacity-80 flex items-center justify-center gap-1"
                    style={{background:'var(--leather)', color:'var(--paper)', border:'none'}}>
                    <Icons.Download size={14} /> Export
                </button>
                <button onClick={() => window.print()}
                    className="flex-1 py-2 rounded typewriter text-sm cursor-pointer hover:opacity-80 flex items-center justify-center gap-1"
                    style={{background:'var(--leather)', color:'var(--paper)', border:'none'}}>
                    <Icons.Printer size={14} /> Print
                </button>
            </div>
        </div>
    );
};
```

- [ ] **Step 2: Verify no syntax errors by refreshing**

Expected: Page loads normally. MorePanel not rendered yet.

- [ ] **Step 3: Commit**

```bash
git add PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add MorePanel component for mobile settings/actions"
```

---

### Task 8: Wire Up Mobile Layout in App Component

**Files:**
- Modify: `PulpAlley/PulpAlley_Forge.html:2784-2914` (App component)

- [ ] **Step 1: Add mobile state and hook to App**

After line 2791 (`const [toast, setToast] = useState(null);`), add:

```jsx
const isMobile = useIsMobile();
const [mobileTab, setMobileTab] = useState('roster');
```

- [ ] **Step 2: Add mobile tab change handler that auto-navigates to editor**

After the `handleSetView` callback (~line 2844), add:

```jsx
const handleMobileSelectChar = useCallback((id) => {
    setSelectedCharId(id);
    setActiveView('character');
    setMobileTab('editor');
}, []);

const handleMobileTabChange = useCallback((tab) => {
    setMobileTab(tab);
    if (tab === 'perks') setActiveView('perks');
    else if (tab === 'campaign') setActiveView('campaign');
    else if (tab === 'roster') { setActiveView(null); }
    else if (tab === 'editor') { setActiveView('character'); }
    else if (tab === 'more') { setActiveView(null); }
}, []);

const handleMoreNavigate = useCallback((key) => {
    setActiveView(key);
    setMobileTab('more');
}, []);
```

- [ ] **Step 3: Pass `isMobile` to TopBar**

Change line ~2887 from:

```jsx
<TopBar league={league} updateLeague={updateLeague} gameData={gameData} onBack={() => { setShowLeagueSelector(true); setSelectedCharId(null); setActiveView(null); }} />
```

To:

```jsx
<TopBar league={league} updateLeague={updateLeague} gameData={gameData} isMobile={isMobile} onBack={() => { setShowLeagueSelector(true); setSelectedCharId(null); setActiveView(null); }} />
```

- [ ] **Step 4: Replace the main layout return block with mobile/desktop branching**

Replace the entire return block (lines 2885-2913):

```jsx
return (
    <div className="h-full flex flex-col" style={{background:'var(--paper)'}}>
        <TopBar league={league} updateLeague={updateLeague} gameData={gameData} onBack={() => { setShowLeagueSelector(true); setSelectedCharId(null); setActiveView(null); }} />

        <div className="flex-1 overflow-hidden flex no-print">
            <RosterSidebar
                league={league} gameData={gameData}
                selectedCharId={selectedCharId}
                onSelectChar={handleSelectChar}
                onOpenAddModal={() => setShowAddModal(true)}
                activeView={activeView}
                onSetView={handleSetView}
            />
            {renderCenterPanel()}
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

With:

```jsx
const renderMobileContent = () => {
    // If we're in associates or settings via More navigation, show those
    if (activeView === 'associates') return <AssociateBuilder league={league} gameData={gameData} updateLeague={updateLeague} />;
    if (activeView === 'settings') return <SettingsPanel league={league} gameData={gameData} updateLeague={updateLeague} />;

    if (mobileTab === 'roster') {
        return <MobileRoster league={league} gameData={gameData} selectedCharId={selectedCharId} onSelectChar={handleMobileSelectChar} onOpenAddModal={() => setShowAddModal(true)} />;
    }
    if (mobileTab === 'editor') {
        const char = league.characters.find(c => c.id === selectedCharId);
        if (char) return <CharacterEditor key={char.id} character={char} gameData={gameData} league={league} onUpdate={updateCharacter} onDelete={(id) => { deleteCharacter(id); setMobileTab('roster'); }} />;
        return <EmptyState league={league} onAddLeader={() => addCharacter('leader')} onOpenAddModal={() => setShowAddModal(true)} />;
    }
    if (mobileTab === 'perks') return <PerkBrowser league={league} gameData={gameData} updateLeague={updateLeague} />;
    if (mobileTab === 'campaign') return <CampaignPanel league={league} gameData={gameData} updateLeague={updateLeague} updateCharacter={updateCharacter} />;
    if (mobileTab === 'more') return <MorePanel league={league} gameData={gameData} updateLeague={updateLeague} onNavigate={handleMoreNavigate} />;
    return null;
};

return (
    <div className="h-full flex flex-col" style={{background:'var(--paper)'}}>
        <TopBar league={league} updateLeague={updateLeague} gameData={gameData} isMobile={isMobile} onBack={() => { setShowLeagueSelector(true); setSelectedCharId(null); setActiveView(null); }} />

        {isMobile ? (
            <div className="flex-1 overflow-y-auto no-print">
                {renderMobileContent()}
            </div>
        ) : (
            <div className="flex-1 overflow-hidden flex no-print">
                <RosterSidebar
                    league={league} gameData={gameData}
                    selectedCharId={selectedCharId}
                    onSelectChar={handleSelectChar}
                    onOpenAddModal={() => setShowAddModal(true)}
                    activeView={activeView}
                    onSetView={handleSetView}
                />
                {renderCenterPanel()}
            </div>
        )}

        <PrintableRoster league={league} gameData={gameData} />

        {isMobile && (
            <MobileTabBar activeTab={mobileTab} onTabChange={handleMobileTabChange} hasSelectedChar={!!selectedCharId} />
        )}

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

- [ ] **Step 5: Verify mobile layout by resizing browser to < 640px width**

Expected:
- TopBar shows only back + league name
- Bottom tab bar appears with 5 tabs
- Roster tab shows full-screen character list
- Tapping a character switches to Editor tab
- More tab shows genre selector, Weapons/Supers toggles, Associates, Settings, Export, Print
- Desktop (> 640px): unchanged layout with sidebar

- [ ] **Step 6: Verify desktop layout is unchanged at full width**

Expected: Sidebar + center panel layout, no bottom tab bar, TopBar shows all controls.

- [ ] **Step 7: Commit**

```bash
git add PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): wire up mobile responsive layout with tab bar navigation"
```

---

### Task 9: Mobile Polish — Back Navigation from Associates/Settings

**Files:**
- Modify: `PulpAlley/PulpAlley_Forge.html` — the `renderMobileContent` function in App

- [ ] **Step 1: Add a back-to-More header when viewing Associates or Settings on mobile**

In the `renderMobileContent` function, replace:

```jsx
if (activeView === 'associates') return <AssociateBuilder league={league} gameData={gameData} updateLeague={updateLeague} />;
if (activeView === 'settings') return <SettingsPanel league={league} gameData={gameData} updateLeague={updateLeague} />;
```

With:

```jsx
if (activeView === 'associates' || activeView === 'settings') {
    return (
        <div className="flex-1 flex flex-col overflow-hidden">
            <div className="p-2 flex items-center gap-2" style={{background:'var(--paper-dark)', borderBottom:'1px solid var(--border)'}}>
                <button onClick={() => { setActiveView(null); setMobileTab('more'); }} className="cursor-pointer hover:opacity-80 flex items-center gap-1" style={{color:'var(--crimson)', background:'transparent', border:'none'}}>
                    <Icons.ArrowLeft size={16} />
                    <span className="typewriter text-xs">Back</span>
                </button>
                <span className="typewriter text-xs uppercase tracking-wider" style={{color:'var(--ink-faded)'}}>{activeView === 'associates' ? 'Associates' : 'Advanced Settings'}</span>
            </div>
            <div className="flex-1 overflow-y-auto">
                {activeView === 'associates'
                    ? <AssociateBuilder league={league} gameData={gameData} updateLeague={updateLeague} />
                    : <SettingsPanel league={league} gameData={gameData} updateLeague={updateLeague} />
                }
            </div>
        </div>
    );
}
```

- [ ] **Step 2: Verify on mobile — More tab > Associates shows back button, tapping returns to More**

Expected: Back button in sub-header, returns to More panel. Same for Settings.

- [ ] **Step 3: Commit**

```bash
git add PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add back navigation for mobile associates/settings views"
```

---

### Task 10: Final Verification

- [ ] **Step 1: Full mobile walkthrough**

Open browser at < 640px width. Verify:
1. Load data file → League selector works (centered card)
2. Create/open a league → TopBar compact, bottom tabs visible
3. Roster tab: character list, add button works
4. Tap character → Editor tab with full editor
5. Perks tab: PerkBrowser works
6. Campaign tab: CampaignPanel works
7. More tab: genre selector works (select Pulp Westerns), Weapons/Supers toggle, Associates link opens with back, Settings link opens with back, Export/Print buttons work
8. Delete character in editor → returns to Roster tab

- [ ] **Step 2: Full desktop walkthrough**

Open browser at > 640px width. Verify:
1. Layout unchanged: sidebar + center panel
2. TopBar: all controls visible (genre, weapons, supers, export, print)
3. Genre modifier pills visible under TopBar
4. No bottom tab bar visible

- [ ] **Step 3: Print verification**

Select Pulp Westerns genre, enable Weapons + Single-Shot. Ctrl+P:
1. Genre section: "Genre: Pulp Westerns" header, then "Riding the Range — ..." and "Gritty — ..." each on own line
2. Optional Rules section: "Optional Rules:" header, then "Weapons — ..." and "Single-Shot — ..." each with description
3. No bottom tab bar in print output
4. Layout otherwise unchanged

- [ ] **Step 4: Final commit if any fixes needed**

```bash
git add PulpAlley/PulpAlley_Forge.html
git commit -m "fix(pulp-alley): polish mobile responsive layout"
```
