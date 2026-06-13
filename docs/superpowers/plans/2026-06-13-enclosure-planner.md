# Enclosure Planner Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single `index.html` file that replicates the paleo.gg JWE3 Enclosure Planner, loading data from `enclosure_planner.json`.

**Architecture:** Single HTML file with inline `<style>` and `<script>`. All state lives in a plain JS object. Data fetched from `enclosure_planner.json` at load time. No frameworks, no build step.

**Tech Stack:** Vanilla HTML/CSS/JS (ES2020+), Flexbox layout, `fetch()` for data loading.

---

## File Structure

- `index.html` — entire app: markup, styles, script
- `enclosure_planner.json` — already exists (280 records, 32 fields each)

Serve locally with:
```bash
npx serve . -l 3000
# or: python3 -m http.server 3000
```
Then open `http://localhost:3000`.

---

### Task 1: HTML skeleton + 3-panel CSS layout

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create `index.html` with skeleton markup and full CSS**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Enclosure Planner | JWE3</title>
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    :root {
      --bg:        #0d1117;
      --panel:     #161b22;
      --surface:   #21262d;
      --border:    #30363d;
      --text:      #c9d1d9;
      --text-dim:  #8b949e;
      --text-head: #e6edf3;
      --accent:    #58a6ff;
      --green:     #3fb950;
      --red:       #f85149;
      --orange:    #d29922;
      --blue:      #58a6ff;
    }

    body {
      background: var(--bg);
      color: var(--text);
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      font-size: 13px;
      height: 100vh;
      display: flex;
      flex-direction: column;
      overflow: hidden;
    }

    header {
      padding: 12px 16px;
      background: var(--panel);
      border-bottom: 1px solid var(--border);
      display: flex;
      align-items: center;
      gap: 12px;
      flex-shrink: 0;
    }
    header h1 { font-size: 16px; color: var(--text-head); font-weight: 600; }
    header .subtitle { color: var(--text-dim); font-size: 12px; }

    .app {
      display: flex;
      flex: 1;
      overflow: hidden;
    }

    /* ── Sidebar ── */
    #sidebar {
      width: 220px;
      flex-shrink: 0;
      background: var(--panel);
      border-right: 1px solid var(--border);
      overflow-y: auto;
      padding: 12px;
    }

    .filter-group { margin-bottom: 14px; }
    .filter-group h3 {
      font-size: 11px;
      text-transform: uppercase;
      letter-spacing: .6px;
      color: var(--text-dim);
      margin-bottom: 6px;
    }
    .filter-tags { display: flex; flex-wrap: wrap; gap: 4px; }
    .tag {
      padding: 3px 8px;
      border-radius: 12px;
      border: 1px solid var(--border);
      background: var(--surface);
      color: var(--text-dim);
      font-size: 11px;
      cursor: pointer;
      user-select: none;
      transition: background .1s, color .1s, border-color .1s;
    }
    .tag.active {
      background: var(--accent);
      border-color: var(--accent);
      color: #fff;
    }
    #reset-btn {
      width: 100%;
      padding: 6px;
      margin-top: 4px;
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: 4px;
      color: var(--text-dim);
      cursor: pointer;
      font-size: 12px;
    }
    #reset-btn:hover { border-color: var(--accent); color: var(--accent); }

    /* ── Table area ── */
    #table-area {
      flex: 1;
      display: flex;
      flex-direction: column;
      overflow: hidden;
    }

    .table-toolbar {
      padding: 8px 12px;
      border-bottom: 1px solid var(--border);
      display: flex;
      align-items: center;
      gap: 10px;
      flex-shrink: 0;
    }
    .table-toolbar input {
      flex: 1;
      padding: 5px 10px;
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: 4px;
      color: var(--text);
      font-size: 12px;
    }
    .table-toolbar input::placeholder { color: var(--text-dim); }
    .table-toolbar input:focus { outline: none; border-color: var(--accent); }
    .count-badge { color: var(--text-dim); font-size: 11px; white-space: nowrap; }

    .table-scroll { overflow-y: auto; flex: 1; }

    table { width: 100%; border-collapse: collapse; }
    thead th {
      position: sticky;
      top: 0;
      background: var(--panel);
      padding: 6px 8px;
      text-align: left;
      font-size: 11px;
      color: var(--text-dim);
      text-transform: uppercase;
      letter-spacing: .4px;
      border-bottom: 1px solid var(--border);
      cursor: pointer;
      user-select: none;
      white-space: nowrap;
    }
    thead th:hover { color: var(--accent); }
    thead th.sort-asc::after  { content: ' ↑'; color: var(--accent); }
    thead th.sort-desc::after { content: ' ↓'; color: var(--accent); }

    tbody tr { border-bottom: 1px solid var(--border); transition: background .1s; }
    tbody tr:hover { background: var(--surface); }
    tbody tr.in-enclosure { background: #1c2a1c; }
    tbody tr.in-enclosure:hover { background: #243224; }
    tbody td { padding: 5px 8px; }
    tbody td.num { text-align: right; font-variant-numeric: tabular-nums; }

    .variant-badge {
      display: inline-block;
      padding: 1px 5px;
      border-radius: 3px;
      font-size: 10px;
      font-weight: 600;
      text-transform: uppercase;
    }
    .variant-badge.female   { background: #3d1f5c; color: #d2a8ff; }
    .variant-badge.male     { background: #1f3a5c; color: #79c0ff; }
    .variant-badge.juvenile { background: #3d3010; color: #e3b341; }

    .add-btn {
      width: 22px; height: 22px;
      border-radius: 4px;
      border: 1px solid var(--border);
      background: var(--surface);
      color: var(--text-dim);
      cursor: pointer;
      font-size: 14px;
      line-height: 1;
      display: flex; align-items: center; justify-content: center;
      transition: background .1s, color .1s;
    }
    .add-btn:hover  { background: var(--accent); border-color: var(--accent); color: #fff; }
    .add-btn.added  { background: #238636; border-color: #238636; color: #fff; }
    .add-btn.added:hover { background: #da3633; border-color: #da3633; }

    /* ── Enclosure panel ── */
    #enclosure-panel {
      width: 280px;
      flex-shrink: 0;
      background: var(--panel);
      border-left: 1px solid var(--border);
      overflow-y: auto;
      padding: 14px;
      display: flex;
      flex-direction: column;
      gap: 12px;
    }

    .panel-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .panel-header h2 { font-size: 14px; font-weight: 600; color: var(--text-head); }
    .panel-header .dino-count { color: var(--text-dim); font-size: 12px; }

    .enc-list { display: flex; flex-direction: column; gap: 4px; }
    .enc-chip {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 5px 8px;
      background: var(--bg);
      border-radius: 4px;
      font-size: 12px;
    }
    .enc-chip .remove-btn {
      background: none;
      border: none;
      color: var(--text-dim);
      cursor: pointer;
      font-size: 14px;
      line-height: 1;
      padding: 0 2px;
    }
    .enc-chip .remove-btn:hover { color: var(--red); }

    .stat-block {
      background: var(--bg);
      border-radius: 6px;
      padding: 10px;
    }
    .stat-block .label {
      font-size: 10px;
      text-transform: uppercase;
      letter-spacing: .6px;
      color: var(--text-dim);
      margin-bottom: 6px;
    }
    .stat-block .big-num {
      font-size: 20px;
      font-weight: 600;
      color: var(--accent);
      font-variant-numeric: tabular-nums;
    }
    .stat-block .note { font-size: 10px; color: var(--text-dim); margin-top: 2px; }

    .cohab-row {
      display: flex;
      align-items: center;
      gap: 6px;
      font-size: 11px;
      margin-bottom: 4px;
    }
    .cohab-dot {
      width: 8px; height: 8px;
      border-radius: 50%;
      flex-shrink: 0;
    }
    .cohab-dot.hostile  { background: var(--red); }
    .cohab-dot.prey     { background: var(--orange); }
    .cohab-dot.friendly { background: var(--green); }
    .cohab-dot.pack     { background: var(--blue); }
    .cohab-dot.unknown  { background: var(--border); }

    .cohab-label.hostile  { color: var(--red); }
    .cohab-label.prey     { color: var(--orange); }
    .cohab-label.friendly { color: var(--green); }
    .cohab-label.pack     { color: var(--blue); }
    .cohab-label.unknown  { color: var(--text-dim); }

    .food-row {
      display: flex;
      justify-content: space-between;
      font-size: 12px;
      margin-bottom: 3px;
    }
    .food-row .food-name { color: var(--text-dim); }
    .food-row .food-val  { font-variant-numeric: tabular-nums; }

    #clear-btn {
      width: 100%;
      padding: 7px;
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: 4px;
      color: var(--text-dim);
      cursor: pointer;
      font-size: 12px;
    }
    #clear-btn:hover { border-color: var(--red); color: var(--red); }

    .empty-msg { color: var(--text-dim); font-size: 12px; text-align: center; padding: 20px 0; }

    .pre-alpha {
      font-size: 10px;
      color: var(--orange);
      background: #2d1f00;
      border: 1px solid #5a3e00;
      border-radius: 4px;
      padding: 4px 8px;
    }
  </style>
</head>
<body>

<header>
  <div>
    <h1>Enclosure Planner</h1>
    <div class="subtitle">Plan your enclosure and optimize cohabitation · JWE3</div>
  </div>
</header>

<div class="app">

  <aside id="sidebar">
    <!-- rendered by JS -->
  </aside>

  <div id="table-area">
    <div class="table-toolbar">
      <input id="search" type="text" placeholder="Search dinosaurs…">
      <span class="count-badge" id="count-badge">Loading…</span>
    </div>
    <div class="table-scroll">
      <table id="dino-table">
        <thead id="table-head"></thead>
        <tbody id="table-body"></tbody>
      </table>
    </div>
  </div>

  <aside id="enclosure-panel">
    <!-- rendered by JS -->
  </aside>

</div>

<script>
// ── State ──────────────────────────────────────────────────────────────────
const state = {
  all: [],          // full dataset
  filters: {        // active filter sets per group
    diet: new Set(),
    era: new Set(),
    nest_size: new Set(),
  },
  search: '',
  sort: { col: 'Name', dir: 'asc' },
  enclosure: [],    // array of record keys currently added
};

// ── Boot ───────────────────────────────────────────────────────────────────
function renderTable() {}    // stubs — replaced in Tasks 3 & 4
function renderEnclosure() {}

fetch('./enclosure_planner.json')
  .then(r => r.json())
  .then(data => {
    state.all = data;
    buildSidebar();
    renderTable();
    renderEnclosure();
  });
</script>
</body>
</html>
```

- [ ] **Step 2: Serve and verify layout**

```bash
cd /Users/nicholas.bradley/Documents/Claude/Projects/enclosure
npx serve . -l 3000
```

Open http://localhost:3000. Expected: dark 3-panel frame, header visible, "Loading…" badge. No errors in console.

- [ ] **Step 3: Commit**

```bash
git init  # if not already a git repo
git add index.html
git commit -m "feat: HTML skeleton and 3-panel CSS layout"
```

---

### Task 2: Filter sidebar

**Files:**
- Modify: `index.html` — replace the `// ── Boot` script section with expanded JS

- [ ] **Step 1: Add helper constants and `buildSidebar()` after the `fetch` block inside `<script>`**

Add these constants and functions inside the `<script>` tag, before `fetch(...)`:

```js
// ── Constants ──────────────────────────────────────────────────────────────
const NEST_SIZE_LABEL = { 1: 'Small', 2: 'Medium', 3: 'Large' };

const TERRAIN_COLS = [
  'Arid','Barren','Cover','Deep Water','Open Water',
  'Pasture','Water','Wetland'
];
const FOOD_COLS = [
  'Ground Fiber','Tall Fiber','Fish','Ground Fruit','Tall Fruit',
  'Ground Leaf','Tall Leaf','Meat','Ground Nut','Tall Nut','Prey','Shark'
];

// ── Sidebar ────────────────────────────────────────────────────────────────
function buildSidebar() {
  const diets    = [...new Set(state.all.map(d => d.diet))].sort();
  const eras     = [...new Set(state.all.map(d => d.era))].sort();
  const nestSizes = [...new Set(state.all.map(d => d.nest_size))].sort((a,b) => a-b);

  const groups = [
    { key: 'diet',      label: 'Diet',      values: diets },
    { key: 'era',       label: 'Era',       values: eras },
    { key: 'nest_size', label: 'Nest Size', values: nestSizes,
      labelFn: v => NEST_SIZE_LABEL[v] || v },
  ];

  const sidebar = document.getElementById('sidebar');
  sidebar.innerHTML = groups.map(g => `
    <div class="filter-group">
      <h3>${g.label}</h3>
      <div class="filter-tags">
        ${g.values.map(v => `
          <span class="tag"
            data-group="${g.key}"
            data-value="${v}"
            onclick="toggleFilter('${g.key}','${v}',this)">
            ${g.labelFn ? g.labelFn(v) : v}
          </span>`).join('')}
      </div>
    </div>
  `).join('') + `<button id="reset-btn" onclick="resetFilters()">Reset filters</button>`;
}

function toggleFilter(group, value, el) {
  const s = state.filters[group];
  const parsed = isNaN(value) ? value : Number(value);
  if (s.has(parsed)) { s.delete(parsed); el.classList.remove('active'); }
  else               { s.add(parsed);    el.classList.add('active'); }
  renderTable();
}

function resetFilters() {
  Object.values(state.filters).forEach(s => s.clear());
  document.querySelectorAll('.tag.active').forEach(t => t.classList.remove('active'));
  state.search = '';
  document.getElementById('search').value = '';
  renderTable();
}
```

- [ ] **Step 2: Wire search input** — add this after `buildSidebar()` inside the `fetch` callback:

```js
document.getElementById('search').addEventListener('input', e => {
  state.search = e.target.value.toLowerCase();
  renderTable();
});
```

- [ ] **Step 3: Reload browser, verify filter tags render and clicking highlights them**

Expected: Diet, Era, Nest Size groups visible with toggle buttons. Reset button present. Clicking a tag turns it blue. (Table doesn't filter yet — `renderTable()` is a stub.)

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: filter sidebar with toggle tags"
```

---

### Task 3: Dinosaur table — render, sort, filter

**Files:**
- Modify: `index.html` — add `filteredDinos()`, `renderTable()`, `renderTableHead()`, `renderTableBody()`, `dominantTerrain()`

- [ ] **Step 1: Add filtering and sorting logic**

Add inside `<script>` before `fetch(...)`:

```js
// ── Data helpers ───────────────────────────────────────────────────────────
function dominantTerrain(d) {
  let best = null, bestVal = -1;
  TERRAIN_COLS.forEach(col => {
    if (d[col] != null && d[col] > bestVal) { bestVal = d[col]; best = col; }
  });
  return { col: best, val: bestVal > 0 ? bestVal : null };
}

function filteredDinos() {
  return state.all.filter(d => {
    if (state.search && !d.Name.toLowerCase().includes(state.search)) return false;
    for (const [group, active] of Object.entries(state.filters)) {
      if (active.size === 0) continue;
      if (!active.has(d[group])) return false;
    }
    return true;
  });
}

function sortedDinos(list) {
  const { col, dir } = state.sort;
  return [...list].sort((a, b) => {
    let av = a[col], bv = b[col];
    // numeric nulls sort last
    if (col === 'dominant_terrain') {
      av = dominantTerrain(a).val ?? -1;
      bv = dominantTerrain(b).val ?? -1;
    }
    if (av == null) av = dir === 'asc' ? Infinity : -Infinity;
    if (bv == null) bv = dir === 'asc' ? Infinity : -Infinity;
    if (typeof av === 'string') return dir === 'asc' ? av.localeCompare(bv) : bv.localeCompare(av);
    return dir === 'asc' ? av - bv : bv - av;
  });
}
```

- [ ] **Step 2: Add `renderTableHead()` and `renderTableBody()`**

Add inside `<script>` before `fetch(...)`:

```js
// ── Table render ───────────────────────────────────────────────────────────
const TABLE_COLS = [
  { key: '',                 label: '',          numeric: false }, // add button
  { key: 'Name',             label: 'Name',      numeric: false },
  { key: 'variant',          label: 'Variant',   numeric: false },
  { key: 'Base Appeal',      label: 'Appeal',    numeric: true  },
  { key: 'Base Dominance',   label: 'Dominance', numeric: true  },
  { key: 'Security Rating',  label: 'Security',  numeric: true  },
  { key: 'dominant_terrain', label: 'Top Terrain', numeric: true },
];

function renderTableHead() {
  const { col, dir } = state.sort;
  document.getElementById('table-head').innerHTML = `<tr>${
    TABLE_COLS.map(c => {
      if (!c.key) return `<th></th>`;
      const cls = c.key === col ? (dir === 'asc' ? 'sort-asc' : 'sort-desc') : '';
      return `<th class="${cls}" onclick="setSort('${c.key}')">${c.label}</th>`;
    }).join('')
  }</tr>`;
}

function setSort(col) {
  if (state.sort.col === col) {
    state.sort.dir = state.sort.dir === 'asc' ? 'desc' : 'asc';
  } else {
    state.sort.col = col;
    state.sort.dir = 'asc';
  }
  renderTable();
}

function renderTableBody(list) {
  document.getElementById('table-body').innerHTML = list.map(d => {
    const key = d.Name + '__' + d.variant;
    const added = state.enclosure.includes(key);
    const dt = dominantTerrain(d);
    return `<tr class="${added ? 'in-enclosure' : ''}">
      <td>
        <button class="add-btn ${added ? 'added' : ''}"
          title="${added ? 'Remove from enclosure' : 'Add to enclosure'}"
          onclick="toggleEnclosure('${key}')">
          ${added ? '−' : '＋'}
        </button>
      </td>
      <td>${d.Name}</td>
      <td><span class="variant-badge ${d.variant}">${d.variant}</span></td>
      <td class="num">${d['Base Appeal'] ?? ''}</td>
      <td class="num">${d['Base Dominance'] ?? ''}</td>
      <td class="num">${d['Security Rating'] ?? ''}</td>
      <td class="num">${dt.val != null ? dt.val.toLocaleString() + ' <span style="color:var(--text-dim);font-size:10px">' + dt.col + '</span>' : ''}</td>
    </tr>`;
  }).join('');
}

function renderTable() {
  const list = filteredDinos();
  const sorted = sortedDinos(list);
  document.getElementById('count-badge').textContent = `${list.length} Found`;
  renderTableHead();
  renderTableBody(sorted);
}
```

- [ ] **Step 3: Reload browser, verify:**
  - Table shows 280 rows on load
  - Clicking a column header sorts ascending then descending
  - Typing in search filters rows live
  - Clicking a Diet filter tag reduces the count
  - Reset button restores 280

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: dinosaur table with sort, search, and filtering"
```

---

### Task 4: Add/remove dinos + enclosure list

**Files:**
- Modify: `index.html` — add `toggleEnclosure()` and `renderEnclosure()`

- [ ] **Step 1: Add `toggleEnclosure()` and basic `renderEnclosure()`**

Add inside `<script>` before `fetch(...)`:

```js
// ── Enclosure state ────────────────────────────────────────────────────────
function toggleEnclosure(key) {
  const i = state.enclosure.indexOf(key);
  if (i === -1) state.enclosure.push(key);
  else          state.enclosure.splice(i, 1);
  renderTable();
  renderEnclosure();
}

function clearEnclosure() {
  state.enclosure = [];
  renderTable();
  renderEnclosure();
}

function enclosureDinos() {
  return state.enclosure.map(key => {
    const [name, variant] = key.split('__');
    return state.all.find(d => d.Name === name && d.variant === variant);
  }).filter(Boolean);
}

// ── Enclosure panel ────────────────────────────────────────────────────────
function renderEnclosure() {
  const panel = document.getElementById('enclosure-panel');
  const dinos = enclosureDinos();

  const listHtml = dinos.length === 0
    ? `<div class="empty-msg">Click ＋ to add dinosaurs</div>`
    : dinos.map(d => {
        const key = d.Name + '__' + d.variant;
        return `<div class="enc-chip">
          <span>${d.Name} <span class="variant-badge ${d.variant}">${d.variant}</span></span>
          <button class="remove-btn" onclick="toggleEnclosure('${key}')" title="Remove">✕</button>
        </div>`;
      }).join('');

  panel.innerHTML = `
    <div class="panel-header">
      <h2>My Enclosure</h2>
      <span class="dino-count">${dinos.length} dino${dinos.length !== 1 ? 's' : ''}</span>
    </div>
    <div class="enc-list">${listHtml}</div>
    ${dinos.length > 0 ? renderSizeBlock(dinos) : ''}
    ${dinos.length >= 2 ? renderCohabBlock(dinos) : ''}
    ${dinos.length > 0 ? renderFoodBlock(dinos) : ''}
    ${dinos.length > 0 ? `<button id="clear-btn" onclick="clearEnclosure()">Clear Enclosure</button>` : ''}
  `;
}

function renderSizeBlock(dinos) { return `<div class="stat-block"><div class="label">Enclosure Size</div><div class="big-num">—</div><div class="note">Coming in next step</div></div>`; }
function renderCohabBlock(dinos) { return `<div class="stat-block"><div class="label">Cohabitation</div><div class="note">Coming in next step</div></div>`; }
function renderFoodBlock(dinos)  { return `<div class="stat-block"><div class="label">Food Requirements</div><div class="note">Coming in next step</div></div>`; }
```

- [ ] **Step 2: Reload browser, verify:**
  - Clicking `＋` on a table row: row turns green, button flips to `−`, dino appears in right panel
  - Clicking `−` (or ✕ in panel): removes from both places
  - Panel shows dino count; Clear button wipes everything

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add/remove dinos to enclosure with live panel sync"
```

---

### Task 5: Enclosure size estimate

**Files:**
- Modify: `index.html` — replace `renderSizeBlock()` stub

- [ ] **Step 1: Replace `renderSizeBlock()` with real implementation**

Find and replace the stub `renderSizeBlock` function:

```js
function renderSizeBlock(dinos) {
  const total = dinos.reduce((sum, d) => {
    const { val } = dominantTerrain(d);
    return sum + (val ?? 0);
  }, 0);
  return `
    <div class="stat-block">
      <div class="label">Enclosure Size</div>
      <div class="big-num">${total.toLocaleString()} m²</div>
      <div class="note">Sum of dominant terrain per dino</div>
      <div class="pre-alpha" style="margin-top:6px">⚠ Pre-alpha: size estimates may be inaccurate</div>
    </div>`;
}
```

- [ ] **Step 2: Reload browser, verify:** Adding a single Acrocanthosaurus (female) should show 13,361 m² (its Pasture value). Adding a second dino should increase the total.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: enclosure size estimate"
```

---

### Task 6: Cohabitation panel

**Files:**
- Modify: `index.html` — replace `renderCohabBlock()` stub

- [ ] **Step 1: Add reaction mapping constants** (before `fetch(...)` in `<script>`):

```js
const REACTION_MAP = {
  fight:     { label: 'Hostile',  cls: 'hostile'  },
  hunt:      { label: 'Prey',     cls: 'prey'      },
  socialize: { label: 'Friendly', cls: 'friendly'  },
  packhunt:  { label: 'Pack',     cls: 'pack'      },
};
const REACTION_SEVERITY = { fight: 3, hunt: 2, packhunt: 1, socialize: 0 };

function worstReaction(a, b) {
  // a's reaction to b's uuid, and b's reaction to a's uuid
  const aUuid = a.Name.toLowerCase().replace(/\s+/g, '');
  const bUuid = b.Name.toLowerCase().replace(/\s+/g, '');
  const ra = a.reaction?.[bUuid];
  const rb = b.reaction?.[aUuid];
  // pick the more severe direction
  const sev = r => REACTION_SEVERITY[r] ?? -1;
  if (ra == null && rb == null) return null;
  if (sev(ra) >= sev(rb)) return ra;
  return rb;
}
```

- [ ] **Step 2: Replace `renderCohabBlock()` stub:**

```js
function renderCohabBlock(dinos) {
  const pairs = [];
  for (let i = 0; i < dinos.length; i++) {
    for (let j = i + 1; j < dinos.length; j++) {
      const reaction = worstReaction(dinos[i], dinos[j]);
      pairs.push({ a: dinos[i], b: dinos[j], reaction });
    }
  }

  const rows = pairs.map(p => {
    const r = p.reaction ? REACTION_MAP[p.reaction] : { label: 'Unknown', cls: 'unknown' };
    return `<div class="cohab-row">
      <span class="cohab-dot ${r.cls}"></span>
      <span style="flex:1;color:var(--text-dim)">${p.a.Name} + ${p.b.Name}</span>
      <span class="cohab-label ${r.cls}">${r.label}</span>
    </div>`;
  }).join('');

  return `
    <div class="stat-block">
      <div class="label">Cohabitation</div>
      ${rows}
    </div>`;
}
```

- [ ] **Step 3: Reload browser, verify:**
  - Add Allosaurus (female) + Allosaurus (male) → should show **Pack** (packhunt)
  - Add Brachiosaurus (female) → Allosaurus + Brachiosaurus should show **Prey** or **Hostile**
  - Unknown pairs show "Unknown" in grey

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: cohabitation pairs with Hostile/Prey/Friendly/Pack labels"
```

---

### Task 7: Food requirements panel

**Files:**
- Modify: `index.html` — replace `renderFoodBlock()` stub

- [ ] **Step 1: Replace `renderFoodBlock()` stub:**

```js
function renderFoodBlock(dinos) {
  const totals = {};
  dinos.forEach(d => {
    FOOD_COLS.forEach(col => {
      if (d[col] != null && d[col] > 0) {
        totals[col] = (totals[col] ?? 0) + d[col];
      }
    });
  });

  const entries = Object.entries(totals).sort((a, b) => b[1] - a[1]);
  if (entries.length === 0) return `
    <div class="stat-block">
      <div class="label">Food Requirements</div>
      <div class="note">No food data for selected dinos</div>
    </div>`;

  const rows = entries.map(([col, val]) => `
    <div class="food-row">
      <span class="food-name">${col}</span>
      <span class="food-val">${val.toLocaleString()}</span>
    </div>`).join('');

  return `
    <div class="stat-block">
      <div class="label">Food Requirements</div>
      ${rows}
    </div>`;
}
```

- [ ] **Step 2: Reload browser, verify:**
  - Adding a carnivore (e.g. Allosaurus female): should show **Prey: 5** or similar
  - Adding a herbivore (e.g. Brachiosaurus female): should show Tall Leaf / Ground Leaf values
  - Values add up correctly when multiple dinos are added

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: food requirements aggregation in enclosure panel"
```

---

### Task 8: Polish and final wiring

**Files:**
- Modify: `index.html` — minor UX improvements

- [ ] **Step 1: Add "no results" empty state to the table**

Inside `renderTableBody()`, replace the current function body with:

```js
function renderTableBody(list) {
  if (list.length === 0) {
    document.getElementById('table-body').innerHTML =
      `<tr><td colspan="${TABLE_COLS.length}" class="empty-msg">No dinosaurs match the current filters</td></tr>`;
    return;
  }
  document.getElementById('table-body').innerHTML = list.map(d => {
    const key = d.Name + '__' + d.variant;
    const added = state.enclosure.includes(key);
    const dt = dominantTerrain(d);
    return `<tr class="${added ? 'in-enclosure' : ''}">
      <td>
        <button class="add-btn ${added ? 'added' : ''}"
          title="${added ? 'Remove from enclosure' : 'Add to enclosure'}"
          onclick="toggleEnclosure('${key}')">
          ${added ? '−' : '＋'}
        </button>
      </td>
      <td>${d.Name}</td>
      <td><span class="variant-badge ${d.variant}">${d.variant}</span></td>
      <td class="num">${d['Base Appeal'] ?? ''}</td>
      <td class="num">${d['Base Dominance'] ?? ''}</td>
      <td class="num">${d['Security Rating'] ?? ''}</td>
      <td class="num">${dt.val != null ? dt.val.toLocaleString() + ' <span style="color:var(--text-dim);font-size:10px">' + dt.col + '</span>' : ''}</td>
    </tr>`;
  }).join('');
}
```

- [ ] **Step 2: Add diet display name helper**

The `diet` field values are raw strings like `diet_carnivore`. Add a formatter before `fetch(...)`:

```js
function formatDiet(raw) {
  return raw ? raw.replace('diet_', '').replace(/_/g, ' ') : raw;
}
function formatEra(raw) {
  return raw ? raw.replace('era_', '').replace(/_/g, ' ') : raw;
}
```

Then update `buildSidebar()` to use `formatDiet` / `formatEra` as `labelFn` for those groups:

```js
const groups = [
  { key: 'diet',      label: 'Diet',      values: diets,     labelFn: formatDiet },
  { key: 'era',       label: 'Era',       values: eras,      labelFn: formatEra  },
  { key: 'nest_size', label: 'Nest Size', values: nestSizes, labelFn: v => NEST_SIZE_LABEL[v] || v },
];
```

- [ ] **Step 3: Final browser verification — full walkthrough:**
  1. Load page → 280 rows, filter tags render with readable names (e.g. "carnivore" not "diet_carnivore")
  2. Search "rex" → filters to T-Rex variants
  3. Click Era "late cretaceous" → count drops, reset restores 280
  4. Add Allosaurus female + Allosaurus male → panel shows: enclosure size, Pack cohabitation, Prey food
  5. Add Brachiosaurus female → cohabitation shows 3 pairs, food adds leaf values
  6. Clear enclosure → panel empties, table rows deselect
  7. No console errors throughout

- [ ] **Step 4: Final commit**

```bash
git add index.html
git commit -m "feat: polish — empty state, readable filter labels, final wiring"
```

---

## Done

Run with:
```bash
npx serve /Users/nicholas.bradley/Documents/Claude/Projects/enclosure -l 3000
```
Open http://localhost:3000.
