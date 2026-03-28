# Food Section Redesign — Design Spec

**Date:** 2026-03-28
**Status:** Approved by user

---

## Goal

Replace the current food category headers and food item card grid in the Builder tab's "Step 3 — Add Food" section with a compact row + pill header layout (Option C). Multi-select is already implemented in the JS (`selectedFood` is a `Set`) — this is primarily a visual redesign with targeted JS updates.

---

## Current State

- Category headers: `.food-cat-header` — plain uppercase text + ▼ arrow, subtle divider line, very low visual weight
- Food items: `.addin-card` in a 2-column grid — bordered rectangles, purple selected state
- Selection feedback: purple border/background on selected cards, `selectedFoodTags` strip below

---

## New Design

### Category Pill Headers

Replace `.food-cat-header` + `.food-cat-divider` with a pill-style row:

```
[ BOWLS   1 selected ]           9 items ▲
```

- **Container** (`id="catpill_{ci}"`): `background: var(--cream)`, `border: 0.5px solid var(--light)`, `border-radius: 8px`; when open: `border-radius: 8px 8px 0 0`
- **Category name** (`.cat-name`): 10px, 700 weight, 1.5px letter-spacing, uppercase, `color: var(--mid)`; when open: `color: var(--dark)` — **dark mode override needed** (see Dark Mode section)
- **"N selected" badge** (`.cat-sel-badge`, `id="catbadge_{ci}"`): teal pill (`background: var(--teal); color: #ffffff`); `display: none` when N = 0, `display: inline-block` when N > 0. Teal is unchanged in dark mode — no dark override needed.
- **Right side** (`.cat-right`, `id="arrow_{ci}"`): item count text + `▲`/`▼`; 10px, `color: var(--mid)`. Keep `id="arrow_{ci}"` to preserve compatibility with existing `toggleFoodCat` chevron logic.
- **Collapsed + has selection**: badge remains visible
- **Exclusive open**: only one category open at a time — `toggleFoodCat` must close all other open categories when opening one

### Item Rows

Replace the `.addin-card` 2-col grid (inside food categories) with full-width rows inside `.item-list`:

```
  ○  Acai Bowl                    560
```

- **List container** (`.item-list`): contains all rows for a category; hidden by default (`display: none`), shown when category is open (`.item-list.open { display: block; }`)
- **Row** (`.item-row`, `id="foodrow_{ci}_{ii}"`): flex row, full width, `padding: 9px 12px`, `border-bottom: 0.5px solid var(--light)`
- **Circular checkbox** (`.item-check`): 16px circle, `border: 1.5px solid var(--light)`, `background: var(--white)`; contains `<span class="item-radio-check">✓</span>` hidden by default
- **Item name** (`.item-name`): 13px, `color: var(--dark)`
- **Calorie badge** (`.item-cal`): 11px, `background: var(--light)`, `color: var(--mid)`, `border-radius: 4px`, `padding: 2px 7px`
- **Last row**: no border-bottom

> **Important:** The search view (`searchFood()`) still generates `.addin-card` + `.food-selected` markup. Do NOT remove those CSS classes — they are still used in the search render path.

### Selected State (teal)

When an item is selected:
- Row background: `#e8f5f3` (light) / `#0a2e28` (dark)
- Item name: `color: var(--teal)`
- Calorie badge: `background: #d0eeea` (light) / `#0d3d32` (dark), `color: var(--teal)`
- Circular checkbox: `background: var(--teal)`, `border-color: var(--teal)`, white `✓` inside (`.item-radio-check { display: block }`)

> **Note:** This changes the food selection color from the current purple (`#6a1b9a`) to teal. Orange remains for smoothie add-ins; teal now covers both filter selection and food selection.

### Selection Tags Strip

The existing `#selectedFoodTags` strip below the food section remains, updated to match the teal theme:
- Tag background: `#0d3d32`, `color: #4db6ac`, `border-radius: 12px`
- ✕ button to deselect individual items (already functional)

### Light Mode Colors

| Element | Value |
|---------|-------|
| Pill background | `var(--cream)` |
| Pill border | `0.5px solid var(--light)` |
| Category name (open) | `color: var(--dark)` |
| Item row background | `var(--white)` |
| Item row divider | `0.5px solid var(--light)` |
| Selected row bg | `#e8f5f3` |
| Selected text | `var(--teal)` |
| Selected cal badge | `background: #d0eeea` |

### Dark Mode Colors

| Element | Value |
|---------|-------|
| Pill background | `#222` |
| Pill border | `#333` |
| Category name (open) | `color: #f0f0f0` (hardcoded — `var(--dark)` is not remapped in dark mode) |
| `.cat-sel-badge` | `var(--teal)` fill, `#ffffff` text — unchanged, no override needed |
| Item list background | `#1e1e1e` |
| Item row background | `#222` |
| Item row divider | `#272727` |
| Item name | `#d0d0d0` |
| Cal badge background | `#2a2a2a` |
| Selected row bg | `#0a2e28` |
| Selected text | `#4db6ac` |
| Selected cal badge | `background: #0d3d32; color: #4db6ac` |
| Checkbox unselected border | `#3a3a3a` |
| Checkbox unselected bg | `#2a2a2a` |

---

## Implementation Scope

**Files changed:** `index.html` only (single-file constraint).

### CSS

- **Add:** `.cat-pill`, `.cat-pill.open`, `.cat-pill-left`, `.cat-name`, `.cat-sel-badge`, `.cat-right`
- **Add:** `.item-list`, `.item-list.open`, `.item-row`, `.item-row.sel`, `.item-check`, `.item-radio-check`, `.item-name`, `.item-cal`
- **Add:** dark mode overrides for all new classes (see Dark Mode Colors table)
- **Remove:** `.food-cat-header`, `.food-cat-divider`, `.food-cat-name`, `.food-cat-arrow`, `body.dark-mode .food-cat-body` — no longer used in category render path
- **Keep (do not remove):** `.food-toggle`, `.food-toggle-btn`, `.food-category`, `.food-items`, `.addin-card`, `.addin-card.food-selected` — still used in search view
- **Update:** `body.dark-mode .addin-card.food-selected` — keep (search view still uses it)

### JS — `renderFoodCategories()`

Generate new markup per category:
```
<div class="food-category" id="foodcat_{ci}">
  <div class="cat-pill" id="catpill_{ci}" onclick="toggleFoodCat({ci})">
    <div class="cat-pill-left">
      <span class="cat-name">CATEGORY NAME</span>
      <span class="cat-sel-badge" id="catbadge_{ci}" style="display:none">0 selected</span>
    </div>
    <span class="cat-right" id="arrow_{ci}">N items ▼</span>
  </div>
  <div class="item-list" id="items_{ci}">
    <!-- one .item-row per food item -->
    <div class="item-row" id="foodrow_{ci}_{ii}" onclick="selectFood({ci}, {ii}, this)">
      <div class="item-check"><span class="item-radio-check" style="display:none">✓</span></div>
      <span class="item-name">Food Name</span>
      <span class="item-cal">NNN</span>
    </div>
  </div>
</div>
```

- All `.item-list` elements start **without** `open` class (collapsed by default — matches current behavior)
- First category may optionally start open (match current behavior — currently all collapsed)

### JS — `toggleFoodCat(ci)`

- Close all other open categories: remove `open` from all `.item-list` elements and all `.cat-pill` elements; reset all `arrow_*` to `▼` text
- Toggle the clicked category open/closed
- Update `arrow_{ci}` text to `▲` (open) or `▼` (closed)

### JS — `selectFood(ci, ii, rowEl)`

- `rowEl` is now the `.item-row` element (previously `.addin-card`)
- Toggle `sel` class on `rowEl`
- Toggle `.item-check` fill: add/remove teal background and show/hide `.item-radio-check`
- Toggle `selectedFood` Set entry for key `${ci}_${ii}`
- Update badge: read `selectedFood` count for items in category `ci`, update `#catbadge_{ci}` text and `display`
- Update `#selectedFoodTags` (existing logic, reuse)
- Update nutrition display (existing logic, unchanged)

### JS — `reset()` (line ~1650)

Update the existing reset querySelectorAll to also clear new row states:
```js
// Add alongside existing reset logic:
document.querySelectorAll('.item-row').forEach(r => r.classList.remove('sel'));
document.querySelectorAll('.item-check').forEach(c => { c.style.background = ''; c.style.borderColor = ''; });
document.querySelectorAll('.item-radio-check').forEach(c => c.style.display = 'none');
document.querySelectorAll('.cat-sel-badge').forEach(b => { b.style.display = 'none'; b.textContent = '0 selected'; });
```

### JS — `loadFavorite()` (saved orders restore)

`loadFavorite()` currently calls `document.getElementById('foodcard_${ci}_${ii}')` to restore visual state. After redesign, update to call `selectFood(ci, ii, document.getElementById('foodrow_${ci}_${ii}'))` for each saved food item, or re-derive visual state from the `selectedFood` Set after loading.

### What Does NOT Change

- Data arrays (`foodCategories[]`)
- Nutrition calculation (`calculateNutrition()`) — reads from `selectedFood` Set, unchanged
- Search view (`searchFood()`) — still uses `.addin-card` + `.food-selected` markup
- Compare tab food selectors
- `selectedFoodTags` strip (kept, colors updated to teal)
- `selectedFood` Set data structure

---

## Out of Scope

- Search view redesign
- Compare tab
- Smoothie add-ins (Builder Steps 1 & 2)
