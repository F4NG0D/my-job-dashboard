# Design: Interview Prep Tab

**Date:** 2026-03-22
**Status:** Approved
**Scope:** `index.html` only

---

## Summary

Add a 4th tab "Interview Prep" with a left sidebar listing "Interviewing" apps and a right panel showing the generated prep sheet. Add a new "Interviewing" status. Persist generated sheets to Supabase `Interview Prep` table.

---

## New Status: "Interviewing"

- Added to `STATUS_OPTIONS`, `statusCls()`, `statusInfo()`, `normalizeStatusToOption()`
- Value: `'Interviewing'` | CSS class: `s-interview` (same green, label distinguishes)
- Position: between "Submitted – Pending Response" and "Interviewed" in all dropdowns
- Added to both modal selects (static HTML)

---

## Tab Structure

- 4th tab button added to header nav + mobile tab row
- `TAB_ORDER` updated to include `'interviewprep'`
- New panel: `#panel-interviewprep`
- Layout: two-column (`ip-layout`) matching `.cl-layout` pattern
  - **Left (280px):** scrollable sidebar, one card per "Interviewing" app
  - **Right (flex:1):** prep sheet display panel

### Sidebar Cards

Each card shows: company logo initials, company name, role. Clicking selects it (accent border) and shows its prep sheet (or empty state) in the right panel. Card badge: "Sheet ready" (green) if a prep sheet exists, "Generate" (accent) if not.

### Right Panel States

1. **No card selected** — empty state prompt
2. **Card selected, no sheet** — company name + "Generate Prep Sheet" button
3. **Generating** — spinner + status text
4. **Sheet ready** — formatted content + Copy + Delete + Regenerate buttons

---

## Data

### Global state

```javascript
let ALL_PREP_SHEETS = []; // loaded from Supabase on tab open
let activePrepAppId  = null;
```

### Supabase calls

| Operation | Method | Endpoint |
|---|---|---|
| Load all | GET | `/rest/v1/Interview%20Prep?select=*` |
| Save new | POST | `/rest/v1/Interview%20Prep` |
| Delete | DELETE | `/rest/v1/Interview%20Prep?id=eq.{id}` |

POST body: `{ app_id, company, role, content, generated_at }`
DELETE uses `SB_POST_HEADERS` with `method: 'DELETE'`

---

## Claude Prompt

**System:** Antonio's profile + "You are an interview prep assistant for IB roles."
**User:** Company, role, job link (if available). Generate 4 sections:
1. Company Overview — history, culture, recent deals, notable clients
2. Your Pitch — tailored 60-second intro using Antonio's OCBC experience + GPA
3. Expected Q&A — 8 common IB questions with firm-specific suggested answers
4. Questions to Ask — 5 smart, firm-specific questions for the interviewer

**Model:** `claude-sonnet-4-6` | **max_tokens:** 3500

---

## Functions

| Function | Purpose |
|---|---|
| `loadPrepSheets()` | GET all sheets from Supabase, populate `ALL_PREP_SHEETS`, render sidebar |
| `renderPrepSidebar()` | Render sidebar cards from `ALL_APPS` filtered to "Interviewing" |
| `selectPrepCard(appId)` | Highlight card, show sheet or empty state in right panel |
| `generatePrepSheet(appId)` | Call Claude, POST result to Supabase, update state, display |
| `deletePrepSheet(prepId, appId)` | DELETE from Supabase, clear right panel, update badge |
| `renderPrepSheet(content)` | Render markdown-like content into right panel |

---

## CSS

- `.ip-layout` — mirrors `.cl-layout` (two columns, 280px left + flex:1 right)
- `.ip-sidebar` — left column, scrollable, padding
- `.ip-card` — sidebar card, hover + active states
- `.ip-panel` — right column, scrollable
- Responsive: single column below 900px (same breakpoint as `.cl-layout`)

---

## What Does NOT Change

- `loadApps()`, `renderApps()`, Supabase Applications table — untouched
- Cover Letter tab — untouched
- Job Leads tab — untouched
