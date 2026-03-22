# "+ Track" Button on Lead Cards — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a `+ Track` button to each job lead card that opens the Add Application modal pre-filled with that lead's details.

**Architecture:** All changes are in a single file (`index.html`). New CSS goes in the `<style>` block. A new JS function `addLeadToApps(id)` handles pre-filling. The card HTML template in `renderLeads()` is updated to render the button. No existing functions are modified.

**Tech Stack:** Vanilla HTML/CSS/JS. No build step. No framework.

---

## Files

| Action | File | Lines |
|--------|------|-------|
| Modify | `index.html` | CSS ~line 271, card HTML ~line 1504, new function ~line 1508 |

---

## Task 1: Add `.track-btn` CSS

**File:** `index.html` — `<style>` block

- [ ] **Step 1: Add `.track-btn` styles after `.apply-btn:hover` (currently line 271)**

  Find this exact block:
  ```css
      .apply-btn:hover { background: var(--accent); color: #fff; }
  ```

  Insert immediately after it:
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
      .track-btn:hover { border-color: var(--accent); color: var(--accent); }
  ```

- [ ] **Step 2: Add responsive rule for `.track-btn` at the phone portrait breakpoint**

  Find this exact line (inside `@media (max-width: 480px)`):
  ```css
        .apply-btn { width: 100%; text-align: center; }
  ```

  Insert immediately after it:
  ```css
        .track-btn { width: 100%; }
        .card-footer .btn-row { width: 100%; }
  ```

---

## Task 2: Add `addLeadToApps(id)` function

**File:** `index.html` — `<script>` block

- [ ] **Step 1: Add the function immediately after `renderLeads()` ends (the closing `}` at ~line 1508)**

  Find this exact line:
  ```javascript
  }

  async function loadLeads() {
  ```

  Insert between them:
  ```javascript

  function addLeadToApps(id) {
    const lead = ALL_LEADS.find(l => l.id === id);
    if (!lead) return;

    // 1. Reset all form fields
    document.getElementById('f-company').value        = '';
    document.getElementById('f-role').value           = '';
    document.getElementById('f-salary').value         = '';
    document.getElementById('f-link').value           = '';
    document.getElementById('f-notes').value          = '';
    document.getElementById('f-date-added').value     = '';
    document.getElementById('f-date-submitted').value = '';
    document.getElementById('f-fetch-url').value      = '';
    document.getElementById('f-fetch-status').textContent = '';
    document.getElementById('f-status').value         = 'Have Not Applied';

    // 2. Pre-fill from lead
    document.getElementById('f-company').value     = lead.company  || '';
    document.getElementById('f-role').value        = lead.title    || '';
    document.getElementById('f-salary').value      = lead.salary   || '';
    document.getElementById('f-link').value        = lead.link     || '';
    document.getElementById('f-date-added').value  = new Date().toISOString().split('T')[0];

    // 3. Open modal
    document.getElementById('modal-form-wrap').style.display  = '';
    document.getElementById('modal-success').style.display    = 'none';
    document.getElementById('modal-submit-btn').disabled      = false;
    document.getElementById('modal-submit-btn').textContent   = 'Save Application';
    document.getElementById('add-modal').classList.add('open');
  }

  ```

---

## Task 3: Add `+ Track` button to the lead card HTML

**File:** `index.html` — inside `renderLeads()`, the card footer template (~line 1504)

- [ ] **Step 1: Update the card footer to include the `+ Track` button**

  Find this exact line inside the `renderLeads()` template literal:
  ```javascript
          ${j.link?`<a class="apply-btn" href="${j.link}" target="_blank" rel="noopener">Search & Apply →</a>`:''}
  ```

  Replace it with:
  ```javascript
          <div class="btn-row" style="display:flex;gap:8px;align-items:center;flex-wrap:wrap">
            <button class="track-btn" onclick="addLeadToApps('${j.id}')">+ Track</button>
            ${j.link?`<a class="apply-btn" href="${j.link}" target="_blank" rel="noopener">Search & Apply →</a>`:''}
          </div>
  ```

---

## Task 4: Manual verification

- [ ] **Step 1: Open the site in a browser** (open `index.html` directly or via `http://localhost` if serving)

- [ ] **Step 2: Verify the button renders on every lead card** — a `+ Track` button should appear to the left of "Search & Apply →"

- [ ] **Step 3: Click `+ Track` on a lead with all fields** — modal opens; company, role, salary, link, and date added are pre-filled; status = "Have Not Applied"; date submitted = blank

- [ ] **Step 4: Click `+ Track` on a lead with no salary** — modal opens; salary field is blank

- [ ] **Step 5: Click `+ Track`, cancel, then click `+ Track` on a different lead** — second lead's data replaces the first, no stale values

- [ ] **Step 6: Save an application via `+ Track`** — application appears in My Applications tab after clicking refresh

- [ ] **Step 7: Toggle light mode** — `+ Track` button renders correctly in both themes

- [ ] **Step 8: Verify mobile layout** — resize browser to 375px wide; both buttons should stack vertically and be full width

---

## Task 5: Commit

- [ ] **Step 1: Stage and commit the single changed file**

  ```
  git add index.html
  git commit -m "feat: add + Track button to job lead cards"
  ```

  Commit via GitHub Desktop: stage `index.html`, commit with message above, push to `main`.

---

## Notes for implementer

- `ALL_LEADS` is the global array holding all loaded leads. `addLeadToApps` looks up by `lead.id` — safer than passing the full JSON object through an `onclick` attribute.
- Do NOT call `openAddModal()` directly — it sets date fields to today and would overwrite values we just pre-filled.
- Do NOT modify `submitNewApp()`, `openAddModal()`, `resetModal()`, or any modal HTML.
- After pushing to GitHub, the live site updates in ~1–2 minutes.
