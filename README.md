# Pulp Alley League Forge

A web-based league builder for **Pulp Alley 2nd Edition** — the tabletop miniatures skirmish game of daring adventure.

Build complete league rosters with full rule enforcement, then print clean reference cards for play.

## Features

- **Full League Builder** — Leader, Sidekick, Allies, Followers, Gangs with all constraints enforced
- **136 Abilities** across Levels 1-4, Epic, and Gang with live validation
- **40 Background Perks** with slot costs, prerequisites, and league structure overrides
- **15 Associate Abilities** with no-duplicate enforcement
- **Skill Assignment** — pool-depletion dropdowns that enforce the rulebook's dice allocation rules
- **Ability Effect Tracking** — skills update live as abilities are added/removed
- **Flavor Renaming** — rename any ability for your setting without changing mechanics
- **Mini Image Upload** — attach reference photos of your miniatures
- **Save/Load** — LocalStorage auto-save with JSON export/import
- **Print-Ready** — clean character cards optimized for printing
- **Pulp Dossier Theme** — aged paper, typewriter fonts, crimson accents

## Quick Start

**Option 1 — Local:** Open `src/index.html` in your browser. The app auto-loads the data file when served via HTTP.

```bash
cd src
python -m http.server 8080
# Open http://localhost:8080
```

**Option 2 — Deploy:** Push to Vercel, Netlify, or any static host. Set `src/` as the publish directory.

## Project Structure

```
src/
  index.html            # Complete app (React 18 + Tailwind via CDN, single file)
  PulpAlley_Data.json   # All abilities, perks, associates, weapons, genres, societies
docs/
  specs/design.md       # Full design specification
  plans/implementation.md  # Implementation plan (15 tasks)
```

## Tech Stack

- **React 18** + ReactDOM via CDN (no build step)
- **Tailwind CSS** via CDN
- **Babel standalone** for in-browser JSX
- **Google Fonts** — Special Elite, Playfair Display, Roboto Slab
- Zero dependencies. No npm. No compilation. Just open the HTML.

## Game Data

The `PulpAlley_Data.json` file contains all game rules encoded as structured data:

| Category | Count |
|---|---|
| Character Templates | 6 (Leader, Sidekick, Ally, Follower, Gang, Epic) |
| Abilities | 136 (32 L1, 22 L2, 25 L3, 32 L4, 19 Epic, 6 Gang) |
| Background Perks | 40 |
| Associate Abilities | 15 |
| Weapons | 12 (6 ranged, 6 melee) |
| Secret Societies | 3 (with 4 ranks each) |
| Genre Presets | 6 |
| Reputation Milestones | 22 |

## Credits

Rules and army building concepts from **Pulp Alley Core Rules: Second Edition** by David Phipps. Published by Pulp Alley. [pulpalley.com](https://pulpalley.com)

This is a fan-made tool. Pulp Alley is a trademark of Pulp Alley LLC.
