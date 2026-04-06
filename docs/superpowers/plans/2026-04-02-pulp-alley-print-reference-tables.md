# Print Reference Tables Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace direct `window.print()` with a Print Settings Modal that lets users toggle reference tables (weapons, harrowing escape, associate abilities, reputation, secret society perks, character templates) to include in the printed output.

**Architecture:** A new `PrintModal` component computes smart defaults from league state and renders checkboxes. `PrintableRoster` gains a `printOptions` prop and renders reference appendix sections after the existing content. All `window.print()` calls are replaced with modal openers.

**Tech Stack:** React 18, Tailwind CSS (CDN), inline JSX/Babel — single file `PulpAlley/PulpAlley_Forge.html`

---

### Task 1: Remove Dead `harrowing` Key from EMPTY_LEAGUE

**Files:**
- Modify: `PulpAlley/PulpAlley_Forge.html:199`

- [ ] **Step 1: Remove `harrowing: false` from optionalRules**

On line 199, change:

```jsx
optionalRules: { weapons: false, harrowing: false, supers: false, deadAlive: false, heartless: false, roadWarriors: false, scavengeAndSurvive: false, singleShot: false },
```

To:

```jsx
optionalRules: { weapons: false, supers: false, deadAlive: false, heartless: false, roadWarriors: false, scavengeAndSurvive: false, singleShot: false },
```

- [ ] **Step 2: Verify no other code references `harrowing` as an optional rule**

Search for `harrowing` in the file. Only `gameData.harrowingEscape` references should remain (campaign panel roller). No code should check `optionalRules.harrowing`.

- [ ] **Step 3: Commit**

```bash
git add PulpAlley/PulpAlley_Forge.html
git commit -m "fix(pulp-alley): remove dead harrowing key from EMPTY_LEAGUE optionalRules"
```

---

### Task 2: Add PrintModal Component

**Files:**
- Modify: `PulpAlley/PulpAlley_Forge.html` — insert before the LeagueSelector component (before line ~2949)

- [ ] **Step 1: Add the PrintModal component**

Find the line `const LeagueSelector =`. Insert the following component BEFORE it:

```jsx
// ==========================================
// 5n. PRINT SETTINGS MODAL
// ==========================================
const PrintModal = ({ league, gameData, onClose }) => {
    const cs = league.campaignState || {};
    const hasCampaign = cs.reputation > 0 || cs.xp > 0;
    const [options, setOptions] = useState({
        weapons: !!league.optionalRules?.weapons,
        harrowingEscape: hasCampaign,
        associateAbilities: league.associates.length > 0,
        reputationTable: hasCampaign,
        secretSociety: !!cs.secretSociety,
        characterTemplates: false,
    });

    const toggle = (key) => setOptions(prev => ({ ...prev, [key]: !prev[key] }));

    const handlePrint = () => {
        // Store options in a ref on the DOM so PrintableRoster can read them synchronously
        window.__pulpAlleyPrintOptions = options;
        // Small delay to let React re-render with the options before print dialog opens
        setTimeout(() => {
            window.print();
            delete window.__pulpAlleyPrintOptions;
        }, 100);
    };

    const toggles = [
        { key: 'weapons', label: 'Weapons Table', desc: '6 ranged + 6 melee weapons with descriptions' },
        { key: 'harrowingEscape', label: 'Harrowing Escape', desc: 'Full Recovery & Critical Injury roll tables' },
        { key: 'associateAbilities', label: 'Associate Abilities', desc: 'All 15 associate abilities reference' },
        { key: 'reputationTable', label: 'Reputation Table', desc: '22 reputation milestones (10–175)' },
        { key: 'secretSociety', label: 'Secret Society Perks', desc: cs.secretSociety ? `${cs.secretSociety} — all ranks` : 'Requires a society selected in Campaign' },
        { key: 'characterTemplates', label: 'Character Templates', desc: 'Stat templates for all character types' },
    ];

    return (
        <div className="fixed inset-0 z-50 flex items-center justify-center" style={{background:'rgba(0,0,0,0.6)'}} onClick={onClose}>
            <div className="p-6 rounded-lg max-w-lg w-full mx-4 max-h-[90vh] overflow-y-auto" style={{background:'var(--paper)', border:'2px solid var(--border)'}} onClick={e => e.stopPropagation()}>
                <div className="flex items-center justify-between mb-2">
                    <h2 className="brand-font text-lg font-bold" style={{color:'var(--ink)'}}>Print Roster</h2>
                    <button onClick={onClose} className="cursor-pointer hover:opacity-70" style={{color:'var(--ink-faded)', background:'transparent', border:'none'}}>
                        <Icons.X size={18} />
                    </button>
                </div>
                <div className="typewriter text-xs mb-4" style={{color:'var(--ink-faded)'}}>{league.name}</div>

                <div className="mb-4">
                    <div className="typewriter text-xs uppercase tracking-wider mb-3" style={{color:'var(--crimson)'}}>Include Reference Tables</div>
                    <div className="space-y-2">
                        {toggles.map(({ key, label, desc }) => (
                            <div key={key} className="flex items-center gap-3 p-2 rounded cursor-pointer hover:opacity-90"
                                style={{background: options[key] ? 'var(--leather)' : 'var(--paper-dark)', border: `1px solid ${options[key] ? 'var(--crimson)' : 'var(--border)'}`}}
                                onClick={() => toggle(key)}>
                                <div className="w-5 h-5 rounded border-2 flex items-center justify-center flex-shrink-0"
                                    style={{borderColor: options[key] ? 'var(--crimson)' : 'var(--border)', background: options[key] ? 'var(--crimson)' : 'transparent'}}>
                                    {options[key] && <span style={{color:'white', fontSize:'12px', fontWeight:'bold'}}>✓</span>}
                                </div>
                                <div className="flex-1">
                                    <div className="brand-font text-sm font-bold" style={{color: options[key] ? 'var(--paper)' : 'var(--ink)'}}>{label}</div>
                                    <div className="typewriter text-xs" style={{color: options[key] ? 'var(--paper-dark)' : 'var(--ink-faded)'}}>{desc}</div>
                                </div>
                            </div>
                        ))}
                    </div>
                </div>

                <div className="flex gap-3">
                    <button onClick={handlePrint}
                        className="flex-1 py-2 rounded typewriter text-sm cursor-pointer hover:opacity-80 flex items-center justify-center gap-1"
                        style={{background:'var(--crimson)', color:'var(--paper)', border:'none'}}>
                        <Icons.Printer size={14} /> Print
                    </button>
                    <button onClick={onClose}
                        className="px-4 py-2 rounded typewriter text-sm cursor-pointer hover:opacity-80"
                        style={{background:'var(--paper-dark)', color:'var(--ink)', border:'1px solid var(--border)'}}>
                        Cancel
                    </button>
                </div>
            </div>
        </div>
    );
};
```

- [ ] **Step 2: Verify no syntax errors by refreshing the page**

Expected: Page loads normally. Modal not rendered yet.

- [ ] **Step 3: Commit**

```bash
git add PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add PrintModal component with smart-defaulted reference table toggles"
```

---

### Task 3: Add Reference Table Sections to PrintableRoster

**Files:**
- Modify: `PulpAlley/PulpAlley_Forge.html:2521` (PrintableRoster component signature)
- Modify: `PulpAlley/PulpAlley_Forge.html:2710-2713` (end of PrintableRoster, before closing `</div>`)

- [ ] **Step 1: Update PrintableRoster signature to accept printOptions**

Change line 2521 from:

```jsx
const PrintableRoster = ({ league, gameData }) => {
```

To:

```jsx
const PrintableRoster = ({ league, gameData }) => {
    const printOptions = window.__pulpAlleyPrintOptions || {};
```

- [ ] **Step 2: Add reference table sections before the closing `</div>` of PrintableRoster**

Find the end of the Associates section in PrintableRoster. After the associates closing `)}` and before the final `</div>` and `);` (around lines 2710-2712), insert:

```jsx
                    {/* === REFERENCE APPENDIX === */}

                    {printOptions.weapons && (() => {
                        const ranged = gameData.weapons?.ranged || [];
                        const melee = gameData.weapons?.melee || [];
                        return (
                            <div style={{breakInside:'avoid', pageBreakInside:'avoid', border:'2px solid #000', borderRadius:'4px', padding:'12px', marginTop:'16px', background:'#fff'}}>
                                <div style={{fontFamily:'Playfair Display, Georgia, serif', fontSize:'14px', fontWeight:'bold', borderBottom:'2px solid #000', marginBottom:'8px', paddingBottom:'4px'}}>Weapons Reference</div>
                                <div style={{fontSize:'10px', fontFamily:'Special Elite, monospace'}}>
                                    <div style={{fontWeight:'bold', marginBottom:'4px', color:'#333'}}>Ranged</div>
                                    {ranged.map(w => (
                                        <div key={w.name} style={{marginBottom:'3px', marginLeft:'8px'}}>
                                            <span style={{fontWeight:'bold'}}>{w.name}</span> — {w.description}
                                        </div>
                                    ))}
                                    <div style={{fontWeight:'bold', marginTop:'8px', marginBottom:'4px', color:'#333'}}>Melee</div>
                                    {melee.map(w => (
                                        <div key={w.name} style={{marginBottom:'3px', marginLeft:'8px'}}>
                                            <span style={{fontWeight:'bold'}}>{w.name}</span> — {w.description}
                                        </div>
                                    ))}
                                </div>
                            </div>
                        );
                    })()}

                    {printOptions.harrowingEscape && (() => {
                        const he = gameData.harrowingEscape;
                        if (!he) return null;
                        const TableSection = ({ title, rows }) => (
                            <div style={{marginBottom:'8px'}}>
                                <div style={{fontWeight:'bold', marginBottom:'4px', color:'#333'}}>{title}</div>
                                <table style={{width:'100%', borderCollapse:'collapse', fontSize:'10px', fontFamily:'Special Elite, monospace'}}>
                                    <thead>
                                        <tr style={{borderBottom:'2px solid #000'}}>
                                            <th style={{textAlign:'left', padding:'2px 6px', width:'40px'}}>Roll</th>
                                            <th style={{textAlign:'left', padding:'2px 6px', width:'100px'}}>Result</th>
                                            <th style={{textAlign:'left', padding:'2px 6px'}}>Effect</th>
                                        </tr>
                                    </thead>
                                    <tbody>
                                        {rows.map(r => (
                                            <tr key={r.roll} style={{borderBottom:'1px solid #ccc'}}>
                                                <td style={{padding:'2px 6px'}}>{r.roll}</td>
                                                <td style={{padding:'2px 6px', fontWeight:'bold'}}>{r.name}</td>
                                                <td style={{padding:'2px 6px'}}>{r.description}</td>
                                            </tr>
                                        ))}
                                    </tbody>
                                </table>
                            </div>
                        );
                        return (
                            <div style={{breakInside:'avoid', pageBreakInside:'avoid', border:'2px solid #000', borderRadius:'4px', padding:'12px', marginTop:'16px', background:'#fff'}}>
                                <div style={{fontFamily:'Playfair Display, Georgia, serif', fontSize:'14px', fontWeight:'bold', borderBottom:'2px solid #000', marginBottom:'8px', paddingBottom:'4px'}}>Harrowing Escape</div>
                                <TableSection title="Full Recovery" rows={he.fullRecovery} />
                                <TableSection title="Critical Injury" rows={he.criticalInjury} />
                            </div>
                        );
                    })()}

                    {printOptions.associateAbilities && (() => {
                        const abilities = gameData.associateAbilities || [];
                        if (abilities.length === 0) return null;
                        return (
                            <div style={{breakInside:'avoid', pageBreakInside:'avoid', border:'2px solid #000', borderRadius:'4px', padding:'12px', marginTop:'16px', background:'#fff'}}>
                                <div style={{fontFamily:'Playfair Display, Georgia, serif', fontSize:'14px', fontWeight:'bold', borderBottom:'2px solid #000', marginBottom:'8px', paddingBottom:'4px'}}>Associate Abilities Reference</div>
                                <div style={{fontSize:'10px', fontFamily:'Special Elite, monospace'}}>
                                    {abilities.map(a => (
                                        <div key={a.name} style={{marginBottom:'3px'}}>
                                            <span style={{fontWeight:'bold'}}>{a.name}</span> — {a.description}
                                        </div>
                                    ))}
                                </div>
                            </div>
                        );
                    })()}

                    {printOptions.reputationTable && (() => {
                        const table = gameData.reputationTable || [];
                        if (table.length === 0) return null;
                        return (
                            <div style={{breakInside:'avoid', pageBreakInside:'avoid', border:'2px solid #000', borderRadius:'4px', padding:'12px', marginTop:'16px', background:'#fff'}}>
                                <div style={{fontFamily:'Playfair Display, Georgia, serif', fontSize:'14px', fontWeight:'bold', borderBottom:'2px solid #000', marginBottom:'8px', paddingBottom:'4px'}}>Reputation Table</div>
                                <table style={{width:'100%', borderCollapse:'collapse', fontSize:'10px', fontFamily:'Special Elite, monospace'}}>
                                    <thead>
                                        <tr style={{borderBottom:'2px solid #000'}}>
                                            <th style={{textAlign:'left', padding:'2px 6px', width:'40px'}}>Rep</th>
                                            <th style={{textAlign:'left', padding:'2px 6px', width:'120px'}}>Milestone</th>
                                            <th style={{textAlign:'left', padding:'2px 6px'}}>Effect</th>
                                        </tr>
                                    </thead>
                                    <tbody>
                                        {table.map(r => (
                                            <tr key={r.threshold} style={{borderBottom:'1px solid #ccc'}}>
                                                <td style={{padding:'2px 6px'}}>{r.threshold}</td>
                                                <td style={{padding:'2px 6px', fontWeight:'bold'}}>{r.name}</td>
                                                <td style={{padding:'2px 6px'}}>{r.description}</td>
                                            </tr>
                                        ))}
                                    </tbody>
                                </table>
                            </div>
                        );
                    })()}

                    {printOptions.secretSociety && (() => {
                        const societyName = league.campaignState?.secretSociety;
                        if (!societyName) return null;
                        const society = (gameData.secretSocieties || []).find(s => s.name === societyName);
                        if (!society) return null;
                        return (
                            <div style={{breakInside:'avoid', pageBreakInside:'avoid', border:'2px solid #000', borderRadius:'4px', padding:'12px', marginTop:'16px', background:'#fff'}}>
                                <div style={{fontFamily:'Playfair Display, Georgia, serif', fontSize:'14px', fontWeight:'bold', borderBottom:'2px solid #000', marginBottom:'8px', paddingBottom:'4px'}}>{society.name}</div>
                                <div style={{fontSize:'10px', fontFamily:'Special Elite, monospace'}}>
                                    {society.ranks.map(rank => (
                                        <div key={rank.rank} style={{marginBottom:'6px'}}>
                                            <div style={{fontWeight:'bold', color:'#333', marginBottom:'2px'}}>Rank {rank.rank} (Rep {rank.reputationRequired}+)</div>
                                            {(rank.perks || []).map(p => (
                                                <div key={p.name} style={{marginLeft:'8px', marginBottom:'2px'}}>
                                                    <span style={{fontWeight:'bold'}}>{p.name}</span> — {p.description}
                                                </div>
                                            ))}
                                        </div>
                                    ))}
                                </div>
                            </div>
                        );
                    })()}

                    {printOptions.characterTemplates && (() => {
                        const templates = gameData.characterTemplates || [];
                        if (templates.length === 0) return null;
                        return (
                            <div style={{breakInside:'avoid', pageBreakInside:'avoid', border:'2px solid #000', borderRadius:'4px', padding:'12px', marginTop:'16px', background:'#fff'}}>
                                <div style={{fontFamily:'Playfair Display, Georgia, serif', fontSize:'14px', fontWeight:'bold', borderBottom:'2px solid #000', marginBottom:'8px', paddingBottom:'4px'}}>Character Templates</div>
                                <table style={{width:'100%', borderCollapse:'collapse', fontSize:'9px', fontFamily:'Special Elite, monospace'}}>
                                    <thead>
                                        <tr style={{borderBottom:'2px solid #000'}}>
                                            <th style={{textAlign:'left', padding:'2px 4px'}}>Type</th>
                                            <th style={{textAlign:'center', padding:'2px 4px'}}>Lvl</th>
                                            <th style={{textAlign:'center', padding:'2px 4px'}}>Health</th>
                                            <th style={{textAlign:'center', padding:'2px 4px'}}>Pools</th>
                                            <th style={{textAlign:'center', padding:'2px 4px'}}>Dice</th>
                                            <th style={{textAlign:'center', padding:'2px 4px'}}>Abilities</th>
                                            <th style={{textAlign:'center', padding:'2px 4px'}}>Max Lvl</th>
                                            <th style={{textAlign:'center', padding:'2px 4px'}}>Slots</th>
                                        </tr>
                                    </thead>
                                    <tbody>
                                        {templates.map(t => (
                                            <tr key={t.type} style={{borderBottom:'1px solid #ccc'}}>
                                                <td style={{padding:'2px 4px', fontWeight:'bold', textTransform:'capitalize'}}>{t.type}</td>
                                                <td style={{padding:'2px 4px', textAlign:'center'}}>{t.level}</td>
                                                <td style={{padding:'2px 4px', textAlign:'center'}}>{t.health}</td>
                                                <td style={{padding:'2px 4px', textAlign:'center'}}>{t.skillDicePools}</td>
                                                <td style={{padding:'2px 4px', textAlign:'center'}}>{t.skillDiceTypes}</td>
                                                <td style={{padding:'2px 4px', textAlign:'center'}}>{t.abilitySlots}</td>
                                                <td style={{padding:'2px 4px', textAlign:'center'}}>{t.maxAbilityLevel}</td>
                                                <td style={{padding:'2px 4px', textAlign:'center'}}>{t.rosterSlotCost}</td>
                                            </tr>
                                        ))}
                                    </tbody>
                                </table>
                            </div>
                        );
                    })()}
```

- [ ] **Step 3: Verify no syntax errors by refreshing**

Expected: Page loads. Print output unchanged (no options set by default via window global).

- [ ] **Step 4: Commit**

```bash
git add PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): add 6 reference table sections to PrintableRoster"
```

---

### Task 4: Wire Up PrintModal in App and Replace window.print() Calls

**Files:**
- Modify: `PulpAlley/PulpAlley_Forge.html` — App component (lines 3003-3200)
- Modify: `PulpAlley/PulpAlley_Forge.html:691` (TopBar desktop print button)
- Modify: `PulpAlley/PulpAlley_Forge.html:2934` (MorePanel mobile print button)

- [ ] **Step 1: Add showPrintModal state to App**

After line 3012 (`const [mobileTab, setMobileTab] = useState('roster');`), add:

```jsx
const [showPrintModal, setShowPrintModal] = useState(false);
```

- [ ] **Step 2: Add PrintModal render in App's return block**

After the existing AddCharacterModal block (after line ~3195 `)}` closing the showAddModal conditional), add:

```jsx
{showPrintModal && (
    <PrintModal
        league={league}
        gameData={gameData}
        onClose={() => setShowPrintModal(false)}
    />
)}
```

- [ ] **Step 3: Pass onPrint callback to TopBar**

In the App return, change the TopBar line from:

```jsx
<TopBar league={league} updateLeague={updateLeague} gameData={gameData} isMobile={isMobile} onBack={() => { setShowLeagueSelector(true); setSelectedCharId(null); setActiveView(null); }} />
```

To:

```jsx
<TopBar league={league} updateLeague={updateLeague} gameData={gameData} isMobile={isMobile} onBack={() => { setShowLeagueSelector(true); setSelectedCharId(null); setActiveView(null); }} onPrint={() => setShowPrintModal(true)} />
```

- [ ] **Step 4: Update TopBar to use onPrint prop**

In the TopBar component signature, add `onPrint` to the destructured props:

```jsx
const TopBar = ({ league, updateLeague, onBack, gameData, isMobile, onPrint }) => (
```

Then change the desktop Print button (inside the `{!isMobile && (` block) from:

```jsx
<button onClick={() => window.print()} className="cursor-pointer p-1 rounded hover:opacity-80" style={{color:'var(--paper)'}} title="Print">
```

To:

```jsx
<button onClick={onPrint} className="cursor-pointer p-1 rounded hover:opacity-80" style={{color:'var(--paper)'}} title="Print">
```

- [ ] **Step 5: Update MorePanel to use onPrint prop**

Change the MorePanel component signature from:

```jsx
const MorePanel = ({ league, gameData, updateLeague, onNavigate }) => {
```

To:

```jsx
const MorePanel = ({ league, gameData, updateLeague, onNavigate, onPrint }) => {
```

Then change the Print button in MorePanel from:

```jsx
<button onClick={() => window.print()}
```

To:

```jsx
<button onClick={onPrint}
```

- [ ] **Step 6: Pass onPrint to MorePanel in renderMobileContent**

In the App's `renderMobileContent` function, change:

```jsx
if (mobileTab === 'more') return <MorePanel league={league} gameData={gameData} updateLeague={updateLeague} onNavigate={handleMoreNavigate} />;
```

To:

```jsx
if (mobileTab === 'more') return <MorePanel league={league} gameData={gameData} updateLeague={updateLeague} onNavigate={handleMoreNavigate} onPrint={() => setShowPrintModal(true)} />;
```

- [ ] **Step 7: Verify by clicking Print button on desktop**

Expected: Modal opens with checkboxes. Smart defaults applied. Clicking "Print" opens browser print dialog. Clicking "Cancel" closes modal.

- [ ] **Step 8: Verify on mobile (< 640px) — More tab > Print button**

Expected: Same modal opens. Responsive (full-width with margins on small screen).

- [ ] **Step 9: Commit**

```bash
git add PulpAlley/PulpAlley_Forge.html
git commit -m "feat(pulp-alley): wire PrintModal into App, TopBar, and MorePanel"
```

---

### Task 5: Final Verification

- [ ] **Step 1: Desktop print with reference tables**

1. Open browser at full width
2. Select a league with: Pulp Westerns genre, Weapons enabled, some associates, campaign reputation > 0
3. Click Print button in TopBar
4. Modal opens with Weapons, Harrowing Escape, Associate Abilities, Reputation Table pre-checked
5. Check all 6 toggles
6. Click Print
7. In print preview, verify:
   - Roster header with genre + optional rule descriptions (from previous work)
   - Character cards
   - Associates
   - Weapons Reference table (ranged + melee)
   - Harrowing Escape tables (Full Recovery + Critical Injury)
   - Associate Abilities reference list
   - Reputation Table (22 rows)
   - Secret Society section (only if society selected)
   - Character Templates table (6 rows)

- [ ] **Step 2: Desktop print with no reference tables**

1. Click Print
2. Uncheck all toggles
3. Print
4. Verify: only roster, characters, associates — no reference appendix

- [ ] **Step 3: Mobile print verification**

1. Resize to < 640px
2. More tab > Print
3. Modal opens, checkboxes work
4. Print produces same output as desktop

- [ ] **Step 4: Verify `harrowing` key is gone**

Search for `harrowing:` in the file. Should only appear in `harrowingEscape` contexts, not as an optionalRules key.

- [ ] **Step 5: Commit any fixes**

```bash
git add PulpAlley/PulpAlley_Forge.html
git commit -m "fix(pulp-alley): polish print reference tables"
```
