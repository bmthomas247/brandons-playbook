# CLAUDE.md — Brandon's Playbook (site)

Private offense playbook for Brandon (Cyprus HS football). Offense language, rules and build queue live in
`CLAUDE.local.md` (not checked in). Read it first. It is the source of truth for every football word. This file covers the code only.

## Files
| Path | What it is |
|---|---|
| `index.html` | The whole app: CSS, Supabase auth, data load, hash router, all tab views, admin edit dialogs. No framework, no build step. |
| `book.css` | Styles for HTML stored in `pages` rows (Learn tab "book" pages). |
| `config.js` | `window.PIRATE_CONFIG` = Supabase URL + **publishable/anon key** (safe to be public; RLS does the locking). |
| `manifest.json` | PWA manifest (add to home screen). |
| `private/` | **Untracked** (in `.git/info/exclude`). Working copies of the drawing module (`app-draw.js`, `*.live*.js` snapshots) and `patch*.json`. The live module is the Supabase row, not these files. |

## How it works
- Plain JS. `index.html` loads supabase-js 2.45.4 from jsDelivr, then `config.js`.
- `boot()`: if there's no session, it shows a magic-link login (`signInWithOtp`; it only creates a user for `bmooset@gmail.com`). With a session, it loads `plays`, `assignments`, `pages`, then `loadMods()`.
- **Drawing module:** `loadMods()` takes the `html` of the `pages` row `slug='app-draw'` and runs it with `new Function`. It exposes `window.DRAW` (renderer, editor, auto-draw, Board tab). If it fails, play pages fall back to `plays.diagram_svg`.
- **Routing** (hash): `#/form` (landing), `#/run`, `#/pass`, `#/prot` (each takes an optional section key), `#/play/<id>`, `#/pos/<letter>`, `#/learn/<slug>`, `#/board`, `#/draw/...`, `#/search`.
- Tabs come from `TABS`. Rail and sidebar sections come from `SECTIONS`; the `pages` row `slug='sections-json'` overrides `SECTIONS` at runtime.
- Phone (<700px): bottom tab bar plus a left rail. Desktop: a sticky sidebar that lists every play per section.
- Demo mode (no `SUPABASE_URL`) reads `demo-data.json` and `draw.js`. Neither file is in the repo.

## Supabase (project `dwbzaelrhirwsrmlinfs`)
| Table | Used for |
|---|---|
| `plays` | Every formation/play/protection card. `phase` decides the tab; also `family`, `subgroup`, `concept`, `hidden`, `hudl_url`, `flags`, text fields. |
| `assignments` | Position jobs per play (`play_id`, `position`, `job`, `confidence` = confirmed/book/inferred). |
| `pages` | HTML blocks by `slug`: info boxes `call-pass`, `motions`, `protection`, `rb-release`; code `app-draw`; also `sections-json`, `built-calls`, Learn pages (`section='_sys'` hides them from Learn). |
| `diagrams` | Saved play drawings (JSON) from the Board/editor (`select`/`upsert`/`delete` by `id`). |

**RLS:** only `bmooset@gmail.com` can read or write. The anon role has no grants on `plays`/`pages`/`assignments` (401), and `diagrams` returns `[]` (checked Nov 12, 2026).
**Never weaken RLS, never add staff access, never commit a service-role/secret key.** Only the publishable key belongs in `config.js`.
Content changes (pages rows, drawing module) need Brandon's signed-in session or his OK for a key. Don't delete `pages`/`diagrams` rows without his OK.

## Run locally
```bash
python -m http.server 8000 --bind 127.0.0.1
```
Open http://127.0.0.1:8000/. You get the same login gate as live. Magic links redirect to the page they were sent from. Data only loads after Brandon signs in on that origin.

## Deploy
GitHub Pages, legacy build from `main` at `/`. No build step. Push to `main` → the "pages build and deployment" Action (~35s) → https://bmthomas247.github.io/brandons-playbook/. Check it with `gh run list -R bmthomas247/brandons-playbook -L 1`.
Changes to the drawing module deploy by updating the `app-draw` row in Supabase, not by git.

## Diagram colors (as coded in `private/app-draw.js`, copy of the `app-draw` row)
Field 720×470, LOS y=270, offense goes up, 10px = 1 yd.
| Thing | Code |
|---|---|
| Offense player | White circle, dark ink `#1f2937` outline + letter. Center = square. Eligible OL = orange `#ea580c` ring + "ELIGIBLE". |
| Defense | White triangle, red `#dc2626` outline + letter. |
| Ball carrier / run path | Orange `#ea580c` (`BALL`). Old imports in amber `#f59e0b` also count as ball. |
| Block | Gray `#4b5563`, T-cap end. |
| Pull, kick, lead, wrap | Blue `#2563eb` (`LEAD`), T-cap. |
| QB roll / boot / fake-set | Orange `#ea580c` arrow, labeled. |
| Fake by a back | Gray `#4b5563` arrow labeled "fake". |
| Motion | Wavy line (style `motion`), any color. Imported `#3b82f6` = motion. |
| Option / alt route | Dashed arrow (`dashArrow`). |
| Read order | Amber `#f59e0b` numbered badge. |
| Labels | "HOT" / "QB reads" red `#dc2626`. "unblocked" brown `#b45309`. |
| Selected | Amber `#f59e0b`. |
Legacy SVG import maps `#7c3aed` purple → puller, `#16a34a` green → lead back, `#f43f5e` → QB boot path. New auto-draws don't use these colors.
Editor palette: `#222222 #4b5563 #dc2626 #2563eb #16a34a #ea580c #7c3aed`.

## Rules
- Mobile-first, short UI text. Keep INFERRED flags visible.
- Org rule: no data without a source unless Kim or Brandon OKs it. Concept pages keep their source credits.
- Claude commits, pushes and checks the live URL. No upload lists for Brandon.
