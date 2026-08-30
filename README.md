# S.L.A.G. Tools
### Stock, Logistics & Acquisition Gateway — Eagle Co. crafting quartermaster tools

Three small, single-purpose static HTML tools — no server, no build step, no
login. Open any of them straight in a browser.

## The three tools

- **`catalog-admin.html`** — private, occasional use. The *only* tool that talks
  to UEX or the Star Citizen Wiki API. Search UEX for materials, sample their
  real quality bands, fetch and bake a blueprint's ingredient list, bulk-import
  your SCMDB "owned blueprints" export. Produces `materials.json` and
  `craftables.json`, ready to commit.
- **`builder.html`** — private, daily use. Reads the shared catalog (no live API
  calls at all), lets you queue up craftables with quality-per-ingredient
  sliders that snap to real bands, tracks your personal on-hand counts, and
  exports a `purchase-order.json` for the buyer.
- **`buyer-board.html`** — public, read-only. Opens straight to current org
  material needs with live UEX listings and buy links per material. Nothing to
  configure beyond pointing it at a `purchase-order.json` URL once.

## Why the split

Pulling from three different APIs (UEX's item catalog, UEX's live marketplace,
and the Wiki's blueprint data) inside a tool you open every day added a lot of
complexity and browser-stored cache for not much benefit, since none of that
data changes minute-to-minute except live prices. So the API-heavy work is
isolated to Catalog Admin, run occasionally, and Builder just reads
well-formed JSON — smaller, faster, and it keeps working even if one of the
upstream APIs has a bad day.

## Setting it up

1. This lives at [github.com/slaghag/slag-tools](https://github.com/slaghag/slag-tools), public — nothing writes back automatically from any of these tools, so someone else opening Builder or Catalog Admin can only mess with their own local browser storage, not the shared files. The only way `materials.json`/`craftables.json`/`purchase-order.json` change is you deliberately exporting and committing.
2. To get a shareable link for the buyer (rather than a raw file URL), turn on GitHub Pages: repo **Settings → Pages → Source → Deploy from a branch**, pick `main` and `/ (root)`. That serves this repo at `https://slaghag.github.io/slag-tools/`. `index.html` redirects that root URL straight to `buyer-board.html` — there's nothing else worth putting at the root, and this way the one link you'd actually hand to someone just works without them needing to know a filename.
3. Open `catalog-admin.html`, add your materials and craftables (or bulk-import from SCMDB), and export `materials.json` / `craftables.json` into `data/`. Commit those.
4. In `builder.html` → Settings, point the catalog URLs at:
   ```
   https://raw.githubusercontent.com/slaghag/slag-tools/main/data/materials.json
   https://raw.githubusercontent.com/slaghag/slag-tools/main/data/craftables.json
   ```
   (double-check `main` is actually your default branch name — view either file on GitHub and use its "Raw" button if unsure, that always gives the correct URL.)
5. In `buyer-board.html`, set `DEFAULT_SOURCE` near the top of the file to:
   ```
   https://raw.githubusercontent.com/slaghag/slag-tools/main/data/purchase-order.json
   ```
   Builder exports this file — commit it whenever your queue changes and you want the buyer to see the update.

## Data schema

### `materials.json`
```json
[
  {
    "id": "lindinium",
    "idItem": 5927,
    "name": "Lindinium",
    "unitBasis": "scu",
    "qualityBands": [585, 618, 729, 853, 930, 954, 1000],
    "qualityBandsFetchedAt": 1788000000000
  }
]
```
- `id` — a short, stable, unique slug. Craftables reference materials by this —
  don't change it once something depends on it.
- `idItem` — the UEX catalog item ID (Catalog Admin fills this in when you
  search and pick a material). `null` if there's no UEX listing for it.
- `unitBasis` — `"scu"` for most raw/refined materials (1 "Unit" listing ==
  1 SCU), or `"piece"` for things sold as individual items, like gems.
- `qualityBands` — the real, distinct quality values seen in live UEX listings
  the last time Catalog Admin sampled it. This is what Builder's sliders snap
  to. Re-sample occasionally (Catalog Admin → Sample bands) since it can go
  stale as listings turn over. `null` until sampled at least once, in which
  case Builder falls back to a generic 8-point scale.

### `craftables.json`
```json
[
  {
    "id": "pulverizer-lmg",
    "name": "Pulverizer LMG",
    "category": "Weapons",
    "blueprintSlug": "pulverizer-lmg",
    "ingredients": [
      {
        "name": "Lindinium",
        "slot": "Frame",
        "quantityScu": 0.06,
        "quantity": null,
        "affects": ["Recoil Smoothness", "Recoil Handling", "Recoil Kick"]
      }
    ],
    "ingredientsFetchedAt": 1788000000000
  }
]
```
- `blueprintSlug` — from the item's page on `api.star-citizen.wiki/blueprints/<slug>`
  (sometimes a UUID instead of a readable slug — either works).
- `ingredients` — baked in by Catalog Admin from the Wiki API: the resolved
  ingredient list, quantities, which crafting "slot" each occupies, and which
  stats that slot's quality tunes. Re-bake (Catalog Admin → Bake/refresh) if a
  patch changes a recipe. Ingredient `name` is matched against `materials.json`
  by loose name matching at Builder runtime — if a material's name changes,
  update it in both files together.
- Quality *targets* aren't stored here — that's your call per build, chosen
  live in Builder's queue, not an org-wide fixed value.

## Data sources & attribution

- Live prices and item catalog: [UEX Corporation](https://uexcorp.space)'s public
  API (`api.uexcorp.uk`), community-maintained.
- Blueprint ingredients and stat data: the [Star Citizen Wiki API](https://api.star-citizen.wiki),
  built on [`StarCitizenWiki/scunpacked-data`](https://github.com/StarCitizenWiki/scunpacked-data),
  a community datamining project. Not affiliated with Cloud Imperium Games.

This is an internal Eagle Co. tool, not a public product — no warranty, use at
your own risk, don't blame us if a listing disappears before you get there.
