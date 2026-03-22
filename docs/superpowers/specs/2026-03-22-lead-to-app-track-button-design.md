# Design: "+ Track" Button on Job Lead Cards

**Date:** 2026-03-22
**Status:** Approved
**Scope:** Single-file static site (`index.html`) — Job Leads tab only

---

## Problem

When a user spots a promising job lead, they must manually open the Add Application modal and re-type the company name, role title, salary, and link. This is friction that slows down application tracking.

## Solution

Add a small `+ Track` button to each lead card in the Job Leads tab. Clicking it opens the existing Add Application modal with all available lead fields pre-filled, ready to save in one click.

---

## Architecture

No new infrastructure. Reuses the existing Add Application modal (`#add-modal`) and Supabase POST logic (`submitNewApp()`).

### Changes to `renderLeads()` (HTML generation)

In the card footer, add `+ Track` button alongside the existing `Search & Apply →` link:

```html
<button class="track-btn" onclick="addLeadToApps(${JSON.stringify(j)})">+ Track</button>
```

### New function: `addLeadToApps(lead)`

Exact call sequence — order matters:

1. **Reset the form** by setting all relevant fields to blank/default:
   - `f-company.value = ''`
   - `f-role.value = ''`
   - `f-salary.value = ''`
   - `f-link.value = ''`
   - `f-notes.value = ''`
   - `f-date-submitted.value = ''` (leave blank — not submitted yet)
   - `f-status.value = 'Have Not Applied'`
   - `f-fetch-url.value = ''`
   - `f-fetch-status.textContent = ''`

2. **Pre-fill from lead:**
   - `f-company.value` ← `lead.company`
   - `f-role.value` ← `lead.title`
   - `f-salary.value` ← `lead.salary || ''`
   - `f-link.value` ← `lead.link || ''`
   - `f-date-added.value` ← today's date in `YYYY-MM-DD` format
   - `f-status.value` ← `'Have Not Applied'`

3. **Show modal** by resetting the modal view state and opening the overlay:
   - `document.getElementById('modal-form-wrap').style.display = ''`
   - `document.getElementById('modal-success').style.display = 'none'`
   - `document.getElementById('modal-submit-btn').disabled = false`
   - `document.getElementById('modal-submit-btn').textContent = 'Save Application'`
   - `document.getElementById('add-modal').classList.add('open')`

> **Why not call `openAddModal()` directly?** `openAddModal()` sets both date fields to today and clears the status — it would overwrite pre-filled values. We replicate only what we need.

### New CSS: `.track-btn`

Styled as a secondary outlined button — visually distinct from "Search & Apply →":

```css
.track-btn {
  background: transparent;
  color: var(--text-muted);
  border: 1.5px solid var(--border);
  border-radius: 8px;
  padding: 7px 13px;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  transition: border-color 0.15s, color 0.15s;
}
.track-btn:hover {
  border-color: var(--accent);
  color: var(--accent);
}
```

---

## Data Flow

```
User clicks "+ Track" on lead card
  → addLeadToApps(lead) called with full lead object
  → Form fields reset to blank/defaults
  → Fields pre-filled from lead (company, title→role, salary, link, date_added=today, status="Have Not Applied")
  → Modal opened (overlay visible, form-wrap visible, success hidden)
  → User reviews, edits if needed, clicks "Save Application"
  → submitNewApp() → Supabase POST
  → App added to Applications tab
```

---

## What Does NOT Change

- `openAddModal()`, `closeAddModal()`, `resetModal()` — untouched
- `submitNewApp()` / Supabase logic — untouched
- Modal HTML structure — untouched
- All other tabs (My Applications, Cover Letter) — untouched
- `jobs.json` or lead data structures — untouched

---

## Field Mapping

| Lead field   | Form field        | Notes                          |
|-------------|-------------------|--------------------------------|
| `company`   | `f-company`       | Direct copy                    |
| `title`     | `f-role`          | Lead uses `title`, form uses `role` |
| `salary`    | `f-salary`        | Optional — blank if absent     |
| `link`      | `f-link`          | Optional — blank if absent     |
| today       | `f-date-added`    | Always set to current date     |
| (blank)     | `f-date-submitted`| Left blank — not submitted yet |
| (blank)     | `f-notes`         | Left blank                     |
| "Have Not Applied" | `f-status` | Default — user can change in modal |

---

## Edge Cases

- **No link on lead:** `f-link` left blank (optional in form)
- **No salary on lead:** `f-salary` left blank (optional in form)
- **Duplicate tracking:** No deduplication — user manages duplicates in Applications tab
- **JSON in onclick:** `JSON.stringify(j)` must handle special characters; this is consistent with how the existing codebase renders inline JS handlers

---

## Testing

1. Click `+ Track` on a lead card with all fields (company, role, salary, link) → modal opens with all four pre-filled, status = "Have Not Applied", date added = today, date submitted = blank
2. Click `+ Track` on a lead with no salary → modal opens, salary field blank
3. Click `+ Track`, then cancel, then click `+ Track` on a different lead → second lead's fields correctly replace the first (no stale values)
4. Save the application → appears in My Applications tab after refresh
5. "Search & Apply →" button still works independently
6. Light and dark themes both render `.track-btn` correctly
