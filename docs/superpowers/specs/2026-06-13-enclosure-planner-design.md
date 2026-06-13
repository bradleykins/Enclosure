# Enclosure Planner — Local Web App Design

**Date:** 2026-06-13
**Status:** Approved for implementation

---

## Overview

A local, single-file HTML app that replicates the paleo.gg JWE3 Enclosure Planner. Runs by opening `index.html` in any browser — no server, no build step. All data is embedded from `enclosure_planner.json`.

---

## Data

`enclosure_planner.json` — 280 records (98 species × female/male/juvenile), 32 fields each:

- Identity: `variant`, `Name`, `bio_group`, `diet`, `era`
- Stats: `Base Appeal`, `Base Dominance`, `Security Rating`
- Terrain requirements (nullable integers): `Arid`, `Barren`, `Cover`, `Deep Water`, `Open Water`, `Pasture`, `Water`, `Wetland`
- Food requirements (nullable integers): `Ground Fiber`, `Tall Fiber`, `Fish`, `Ground Fruit`, `Tall Fruit`, `Ground Leaf`, `Tall Leaf`, `Meat`, `Ground Nut`, `Tall Nut`, `Prey`, `Shark`
- Cohabitation: `nest_size`, `social_group_min`, `social_group_max`, `reaction` (object mapping species uuid → `fight` | `hunt` | `socialize` | `packhunt`)

---

## Layout — 3-Panel

```
┌─────────────┬──────────────────────────────┬──────────────────┐
│  Filters    │  Dinosaur Table              │  My Enclosure    │
│  (sidebar)  │  (scrollable, sortable)      │  (panel)         │
│  ~220px     │  flex-grow                   │  ~280px          │
└─────────────┴──────────────────────────────┴──────────────────┘
```

Single HTML file. CSS layout via flexbox. Dark theme matching paleo.gg (`#0d1117` bg, `#161b22` panels).

---

## Filter Sidebar

Collapsible filter groups, each a set of toggle buttons:

| Group | Values |
|---|---|
| Habitat | Aerial, Aquatic, Terrestrial (derived from `diet` field) |
| Diet | Unique values from `diet` field |
| Era | Unique values from `era` field |
| Nest Size | Small (1), Medium (2), Large (3) mapped from `nest_size` |
| Cohabitation | Filters table to show only dinos compatible with already-added ones |

Reset button clears all filters. Count badge updates live (e.g. "280 Found").

---

## Dinosaur Table

Columns: Name, Variant (icon), Base Appeal, Base Dominance, Security Rating, dominant terrain column (highest non-null terrain value), + button.

- Sortable by any column (click header)
- Search box at top of table for name filtering
- `+` button adds the dino to the enclosure panel; button changes to `−` if already added
- Rows added to the enclosure are highlighted

---

## Enclosure Panel

### Dino list
Each added dino shown as a removable chip: `Name variant ✕`. Clear All button at bottom.

### Enclosure Size
Sum of each dino's dominant terrain value (highest non-null terrain field). Labelled as estimate with a pre-alpha caveat note.

### Cohabitation
Every unique pair of added dinos checked against the `reaction` map:

| Game value | Display label | Color |
|---|---|---|
| `fight` | Hostile | Red |
| `hunt` | Prey | Orange |
| `socialize` | Friendly | Green |
| `packhunt` | Pack | Blue |
| missing | Unknown | Grey |

Lookup is directional (A's reaction to B). If both directions differ, show the more dangerous one. Pairs only shown when 2+ dinos added.

### Food Requirements
Aggregate all non-null food columns across added dinos, sum by type, display as a list. Only show food types with a non-zero total.

---

## Tech Stack

- Single `index.html` — HTML + inline `<style>` + inline `<script>`
- Data loaded via `fetch('./enclosure_planner.json')` (requires serving over HTTP or using `file://` with CORS disabled — note in README)
- No frameworks, no build step
- Dark theme, monospace numbers

---

## Out of Scope

- Saving/sharing enclosures
- Mobile layout
- Images/icons from the CDN
- The "Fish" column (duplicate header in source data — ignore)
