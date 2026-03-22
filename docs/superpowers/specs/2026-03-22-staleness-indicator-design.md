# Design: Staleness Indicator for Applications

**Date:** 2026-03-22
**Status:** Approved
**Scope:** `index.html` only — My Applications tab

---

## Problem

Applications stuck in "Submitted – Pending Response" with no reply can be forgotten. There's no visual cue when an application has gone cold.

## Solution

Two complementary mechanisms:
1. **Auto-stale** — purely visual: if an app's status is `"Submitted - Pending Response"` AND `date_submitted` is more than 45 days ago, render the status pill as "⏰ Stale – No Response" in amber. Nothing is written to Supabase.
2. **Manual status** — a new `"Stale - No Response"` option in all status dropdowns, saved to Supabase when selected.

Both render identically — an amber "⏰ Stale – No Response" pill.

---

## Threshold

**45 days** from `date_submitted`. Only applies when:
- `Application Status` in Supabase is exactly `"Submitted - Pending Response"`, AND
- `date_submitted` is a valid date AND today − date_submitted > 45 days

If `date_submitted` is blank/invalid, auto-stale does NOT trigger (can't compute the window).

---

## Architecture

### 1. New CSS — `.s-stale` status class

Add after the existing `.s-notyet` rule:

```css
.s-stale { background: rgba(245,158,11,0.15); color: var(--status-stale); border-color: rgba(245,158,11,0.35); }
```

Add `--status-stale` to both theme blocks:
- Dark:  `--status-stale: #f59e0b;`
- Light: `--status-stale: #ff9f0a;`

### 2. Helper function — `isAutoStale(app)`

```javascript
function isAutoStale(app) {
  if ((app.status || '').trim() !== 'Submitted - Pending Response') return false;
  if (!app.date_submitted) return false;
  const days = (Date.now() - new Date(app.date_submitted)) / 86400000;
  return days > 45;
}
```

### 3. `renderApps()` — display logic

When building each table row's status cell:
- If `isAutoStale(app)` → render amber pill: `"⏰ Stale – No Response"` with class `s-stale`
- Else if `app.status === 'Stale - No Response'` → same amber pill (manual)
- Else → existing status pill logic unchanged

The inline status `<select>` element (`.status-select`) displays the stored Supabase value unchanged — auto-stale does NOT change the select's displayed value. This keeps the dropdown truthful about what's stored.

### 4. Inline status `<select>` — add new option

In `renderApps()` where the `.status-select` HTML is built, add:

```html
<option value="Stale - No Response">⏰ Stale – No Response</option>
```

alongside the existing options.

### 5. Add/Edit modals — add new option

In both `#add-modal` and `#edit-modal` status `<select>` elements, add:

```html
<option value="Stale - No Response">⏰ Stale – No Response</option>
```

### 6. `normalizeStatusToOption()` — handle new value

The edit modal uses `normalizeStatusToOption()` to map stored status strings to select values. Add a case for `"Stale - No Response"` → `"Stale - No Response"` (exact match, no transformation needed).

### 7. Status styling helpers — `statusClass()` / `statusSelectClass()`

Wherever the codebase maps a status string to a CSS class (e.g. `s-pending`, `s-rejected`), add:
- `"stale"` / `"Stale - No Response"` → `"s-stale"`

### 8. Filter bar — new "⏰ Stale" button

Add a filter button that matches both auto-stale and manual stale:

```html
<button class="filter-btn" data-scat="Stale" onclick="setAppFilter('Stale')">⏰ Stale</button>
```

In `renderApps()` filter logic, when `activeAppFilter === 'Stale'`:
```javascript
isAutoStale(app) || (app.status || '').includes('Stale')
```

### 9. Stats strip

- **Pending count** excludes manual `"Stale - No Response"` apps (they are not submitted-pending)
- Auto-stale apps continue to be counted as Pending (their Supabase value is still `"Submitted - Pending Response"`)
- No new stat counter added — keep the strip minimal

---

## Status Value Reference

| Supabase value | Display label | CSS class | Notes |
|---|---|---|---|
| `Submitted - Pending Response` (≤45d) | ● Pending | `s-pending` | Normal |
| `Submitted - Pending Response` (>45d) | ⏰ Stale – No Response | `s-stale` | Auto-stale display override |
| `Stale - No Response` | ⏰ Stale – No Response | `s-stale` | Manual |
| `Rejected` | Rejected | `s-rejected` | Unchanged |
| `Interviewed` | Interviewed | `s-interview` | Unchanged |
| `Offer Extended - In Progress` | Offer | `s-offer` | Unchanged |
| `Have Not Applied` | Have Not Applied | `s-notyet` | Unchanged |

---

## What Does NOT Change

- Supabase schema — no new columns
- `submitNewApp()` / `submitEditApp()` / `deleteApp()` — untouched
- `loadApps()` — untouched
- `updateAppStatus()` (inline PATCH) — untouched; handles any status string already
- Job Leads tab, Cover Letter tab — untouched
