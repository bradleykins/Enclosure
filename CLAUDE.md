# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the app

```bash
npx serve . -l 3000
# or
python3 -m http.server 3000
```

Open http://localhost:3000. No build step — `index.html` is the entire app.

## Architecture

Single-file app: `index.html` (1,079 lines) with inline `<style>` and `<script>`. Data loaded at boot via `fetch('./enclosure_planner.json')`.

**Data file:** `enclosure_planner.json` — 280 records (98 species × female/male/juvenile), 32 fields each. Key fields:
- Identity: `Name`, `variant`, `bio_group`, `diet`
- Terrain requirements (nullable int): `Arid`, `Barren`, `Cover`, `Deep Water`, `Open Water`, `Pasture`, `Water`, `Wetland`
- Food requirements (nullable int): `Ground Fiber`, `Tall Fiber`, `Fish`, `Ground Fruit`, `Tall Fruit`, `Ground Leaf`, `Tall Leaf`, `Meat`, `Ground Nut`, `Tall Nut`, `Prey`, `Shark`
- Cohabitation: `cohab_like`, `cohab_dislike`, `cohab_liked_by`, `cohab_disliked_by` — each an array of `[uuid, variants[]]` tuples (variant-specific)

**State:** Single `state` object with `all[]`, `filters{}`, `search`, `sort`, `enclosure{}` (key→count map).

**Three-panel layout:** filter sidebar → sortable/filterable dino table → enclosure builder panel.

## Key logic

**Cohabitation** (`cohabRelation`): derives `friendly/neutral/mixed/hostile/separate_habitat` from `cohab_like`/`cohab_dislike` arrays. Same-species → friendly. Cross-habitat → separate_habitat. Variant-aware via `cohabHas()`.

**Enclosure size formula** (matches paleo.gg): sum ALL terrain+food cols per dino, then `area(n) = base × (1 + (n−1) × 0.14)` for pack size n. Fence blocks = `ceil(4√area / 64)`.

**Fit score** (`resourceScore`): 0–100, % of candidate's terrain+food needs already covered by enclosure profile. Terrain weighted 40%, food 60%.

**Lock-in score** (`lockInScore`): % of remaining same-habitat dinos that would become hostile if this candidate is added.

**Sort priority when enclosure active:** compat tier → fit desc → lock-in asc → user column.

## Re-scraping data

If paleo.gg data needs refreshing, the scraping approach used:
- Enclosure planner dino data (terrain/food stats): Playwright, extract `window.__NEXT_DATA__.props.pageProps.dinos`
- Cohabitation data: Node.js `https.get` on each DinoDB detail page (`/dino-db/{uuid}`), parse `__NEXT_DATA__` from HTML, extract `detail.cohab_like/dislike/liked_by/disliked_by`
