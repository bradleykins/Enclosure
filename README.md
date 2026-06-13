# JWE3 Enclosure Planner

A local/hosted tool for planning dinosaur enclosures in Jurassic World Evolution 3.

**[Live demo](https://bradleykins.github.io/enclosure/)**

## Features

- **280 dinosaurs** across 98 species (female, male, juvenile variants)
- **Compatibility column** — shows Friendly / Neutral / Mixed / Hostile / Diff. Habitat against your current enclosure, using full cohabitation data from paleo.gg DinoDB
- **Fit score (0–100)** — how much of a candidate's terrain + food needs are already covered by your enclosure
- **Lock-in score (%)** — how many remaining dinos become hostile if you add this one
- **Pack size controls** — ± buttons to increase/decrease count per species; all stats scale correctly
- **Enclosure size estimate** — area, perimeter, and fence segment count matching paleo.gg's formula
- **Terrain & food requirements** — aggregated totals for your current enclosure
- **Cohabitation panel** — per-pair relationship breakdown
- **Filters** — Habitat (Land/Sea/Air), Food type, Nest Size
- **Sort** — Name, Appeal, Size, Fit, Lock-in

## Running locally

```bash
npx serve . -l 3000
# or
python3 -m http.server 3000
```

Then open http://localhost:3000.

## Hosting on GitHub Pages

1. Push to GitHub
2. Settings → Pages → Deploy from branch → `main` → `/ (root)`
3. Live at `https://bradleykins.github.io/enclosure/`

