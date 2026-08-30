# S.L.A.G. Tools
### Stock, Logistics & Acquisition Gateway — Eagle Co. crafting quartermaster tools

A single-file, no-build web tool for planning crafting builds, tracking a shopping
list, and finding live material prices on UEX — built by/for Eagle Co.

## What's in here

- **`quartermaster.html`** — the tool itself. Open it in a browser, no server or
  build step needed. It saves your personal state (build queue, on-hand counts)
  in your own browser and pulls the shared org catalog from the JSON files below.
- **`data/materials.json`** — the shared catalog of raw/refined materials the org
  crafts with. This is what turns a material name into a real UEX item link and a
  correct price-unit conversion.
- **`data/craftables.json`** — the shared catalog of things the org crafts, and
  what materials (and at what quality) each one needs.

Materials and craftables live in this repo, not in any one person's browser, so
the whole org sees the same catalog. Personal stuff — your build queue, your
on-hand counts, how much extra of something you personally need — stays local to
your own browser and is never written back here automatically. The tool has an
**Export for shared catalog** button that generates ready-to-commit JSON for
anything you add locally, if you want to promote it into the shared files.

## Setting it up

1. Push this repo to GitHub under the Eagle Co. org (or your own account).
2. Open `quartermaster.html` (locally, or host it — GitHub Pages works fine for
   a static file like this) and go to **Settings**.
3. Point the two catalog URLs at this repo's raw file paths, e.g.:
   ```
   https://raw.githubusercontent.com/<org>/slag-tools/main/data/materials.json
   https://raw.githubusercontent.com/<org>/slag-tools/main/data/craftables.json
   ```
4. Hit **Load shared catalog**. From then on it loads automatically.

## Data schema

### `materials.json`
```json
[
  {
    "id": "lindinium",
    "idItem": 5927,
    "name": "Lindinium",
    "unitBasis": "scu"
  }
]
```
- `id` — a short, stable, unique slug. Used to link craftables to this material —
  don't change it once craftables reference it.
- `idItem` — the UEX catalog item ID (what the tool uses to pull live prices).
  Find it by searching for the material in the tool, or from a UEX item page URL
  (`uexcorp.space/items/info?id=####`). `null` if there's no UEX listing for it.
- `unitBasis` — `"scu"` for most raw/refined materials (1 "Unit" listing ==
  1 SCU), or `"piece"` for things sold as individual items, like gems, where a
  "Unit" listing is literally one piece.

### `craftables.json`
```json
[
  {
    "id": "pulverizer-lmg",
    "name": "Pulverizer LMG",
    "category": "Weapons",
    "blueprintSlug": "pulverizer-lmg",
    "materials": [
      { "materialId": "lindinium", "qty": 0.06, "quality": 900 }
    ]
  }
]
```
- `materialId` — must match an `id` in `materials.json`.
- `qty` — amount needed per unit crafted, in the material's own unit basis (SCU
  or pieces).
- `quality` — the minimum quality (0–1000) this build calls for that ingredient.
- `blueprintSlug` — optional, for a future feature: pulling real quality-to-stat
  scaling data from the Star Citizen Wiki API so the tool can preview how
  sliding an ingredient's quality changes the finished item's stats, the way
  SCMDB and SC Craft Tools do. Not wired up yet — the slug alone doesn't cost
  anything to include now.

## Data sources & attribution

- Live prices and item catalog: [UEX Corporation](https://uexcorp.space)'s public
  API (`api.uexcorp.uk`), community-maintained.
- Planned stat-scaling data: the [Star Citizen Wiki API](https://api.star-citizen.wiki),
  built on [`StarCitizenWiki/scunpacked-data`](https://github.com/StarCitizenWiki/scunpacked-data),
  a community datamining project. Not affiliated with Cloud Imperium Games.

This is an internal Eagle Co. tool, not a public product — no warranty, use at
your own risk, don't blame us if a listing disappears before you get there.
