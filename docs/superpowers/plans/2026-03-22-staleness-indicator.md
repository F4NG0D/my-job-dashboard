# Staleness Indicator — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement task-by-task.

**Goal:** Auto-mark "Submitted – Pending Response" apps as stale after 45 days (display-only, amber), and add a manual "Stale – No Response" status option that saves to Supabase.

**Architecture:** All changes in `index.html`. Auto-stale is computed in `renderApps()` via `isAutoStale(app)` — never writes to Supabase. Manual stale is a new value in `STATUS_OPTIONS` and both modal selects. Both cases render the same amber select styling via `.s-stale` class.

**Tech Stack:** Vanilla HTML/CSS/JS, no build step.

---

## Files

| Action | File | Areas |
|--------|------|-------|
| Modify | `index.html` | CSS vars (~line 17–50), CSS rules (~line 344–380), filter bar HTML (~line 937), modal selects (~line 1127, ~line 1201), STATUS_OPTIONS (~line 1325), status helpers (~line 1333–1394), renderApps row template (~line 1628–1682) |

---

## Task 1: CSS — add `--status-stale` variable + `.s-stale` rules

- [ ] **Step 1: Add `--status-stale` to dark theme `:root` block**

  Find:
  ```css
      --status-notapplied: #818384;
    }
  ```
  Replace with:
  ```css
      --status-notapplied: #818384;
      --status-stale: #f59e0b;
    }
  ```

- [ ] **Step 2: Add `--status-stale` to light theme block**

  Find:
  ```css
      --status-notapplied: #6e6e73;
    }
  ```
  Replace with:
  ```css
      --status-notapplied: #6e6e73;
      --status-stale: #e08c00;
    }
  ```

- [ ] **Step 3: Add `.s-stale` status badge rule after `.s-notyet`**

  Find:
  ```css
      .s-notyet   { background: rgba(129,131,132,0.15); color: var(--status-notapplied); }
  ```
  Replace with:
  ```css
      .s-notyet   { background: rgba(129,131,132,0.15); color: var(--status-notapplied); }
      .s-stale    { background: rgba(245,158,11,0.15);  color: var(--status-stale); }
  ```

- [ ] **Step 4: Add `.status-select.s-stale` rule after `.status-select.s-notyet`**

  Find:
  ```css
      .status-select.s-notyet    { background: rgba(129,131,132,0.10); color: var(--status-notapplied); border-color: rgba(129,131,132,0.3); }
  ```
  Replace with:
  ```css
      .status-select.s-notyet    { background: rgba(129,131,132,0.10); color: var(--status-notapplied); border-color: rgba(129,131,132,0.3); }
      .status-select.s-stale     { background: rgba(245,158,11,0.10);  color: var(--status-stale);      border-color: rgba(245,158,11,0.35); }
  ```

---

## Task 2: HTML — filter bar + modal selects

- [ ] **Step 1: Add "⏰ Stale" filter button after the "📌 Not Applied" button**

  Find:
  ```html
          <button class="filter-btn" data-scat="Not Yet Applied"                    onclick="setAppFilter('Not Yet Applied')">📌 Not Applied</button>
  ```
  Replace with:
  ```html
          <button class="filter-btn" data-scat="Not Yet Applied"                    onclick="setAppFilter('Not Yet Applied')">📌 Not Applied</button>
          <button class="filter-btn" data-scat="Stale"                              onclick="setAppFilter('Stale')">⏰ Stale</button>
  ```

- [ ] **Step 2: Add stale option to Add Application modal `#f-status`**

  Find:
  ```html
            <option value="Offer Extended - In Progress">Offer Extended – In Progress</option>
          </select>
        </div>
        <div class="form-cols">
          <div class="form-row">
            <label>Date Added</label>
  ```
  Replace with:
  ```html
            <option value="Offer Extended - In Progress">Offer Extended – In Progress</option>
            <option value="Stale - No Response">⏰ Stale – No Response</option>
          </select>
        </div>
        <div class="form-cols">
          <div class="form-row">
            <label>Date Added</label>
  ```

- [ ] **Step 3: Add stale option to Edit Application modal `#ef-status`**

  Find:
  ```html
            <option value="Offer Extended - In Progress">Offer Extended – In Progress</option>
          </select>
        </div>
        <div class="form-cols">
          <div class="form-row">
            <label>Date Added</label>
            <input type="date" id="ef-date-added" />
  ```
  Replace with:
  ```html
            <option value="Offer Extended - In Progress">Offer Extended – In Progress</option>
            <option value="Stale - No Response">⏰ Stale – No Response</option>
          </select>
        </div>
        <div class="form-cols">
          <div class="form-row">
            <label>Date Added</label>
            <input type="date" id="ef-date-added" />
  ```

---

## Task 3: JS — STATUS_OPTIONS + status helpers

- [ ] **Step 1: Add stale entry to `STATUS_OPTIONS`**

  Find:
  ```javascript
    { value: 'Offer Extended - In Progress', cls: 's-offer',     label: 'Offer Extended' },
  ];
  ```
  Replace with:
  ```javascript
    { value: 'Offer Extended - In Progress', cls: 's-offer',     label: 'Offer Extended' },
    { value: 'Stale - No Response',          cls: 's-stale',     label: '⏰ Stale – No Response' },
  ];
  ```

- [ ] **Step 2: Update `normalizeStatusToOption()` — add stale case before the final fallback**

  Find:
  ```javascript
    if (l.includes('not yet') || l.includes('not applied') || l.includes('have not')) return 'Have Not Applied';
    return 'Have Not Applied';
  }
  ```
  Replace with:
  ```javascript
    if (l.includes('not yet') || l.includes('not applied') || l.includes('have not')) return 'Have Not Applied';
    if (l.includes('stale')) return 'Stale - No Response';
    return 'Have Not Applied';
  }
  ```

- [ ] **Step 3: Update `statusCls()` — add stale case**

  Find:
  ```javascript
    if (l.includes('pending') || l.includes('submitted')) return 's-pending';
    return 's-notyet';
  }
  ```
  Replace with:
  ```javascript
    if (l.includes('stale'))                               return 's-stale';
    if (l.includes('pending') || l.includes('submitted')) return 's-pending';
    return 's-notyet';
  }
  ```

- [ ] **Step 4: Update `statusInfo()` — add stale case**

  Find:
  ```javascript
    if (l.includes('pending') || l.includes('submitted'))
      return { cls:'s-pending',   label:'Pending' };
    if (l.includes('not yet') || l.includes('not applied') || l.includes('have not'))
  ```
  Replace with:
  ```javascript
    if (l.includes('stale'))
      return { cls:'s-stale',     label:'⏰ Stale – No Response' };
    if (l.includes('pending') || l.includes('submitted'))
      return { cls:'s-pending',   label:'Pending' };
    if (l.includes('not yet') || l.includes('not applied') || l.includes('have not'))
  ```

- [ ] **Step 5: Add `isAutoStale()` and `getAppStatusCls()` helpers after `statusInfo()`**

  Find:
  ```javascript
  // ─── Tab switching ─────────────────────────────────────────────────────────
  ```
  Insert immediately before it:
  ```javascript
  function isAutoStale(app) {
    if ((app.status || '').trim() !== 'Submitted - Pending Response') return false;
    if (!app.date_submitted) return false;
    return (Date.now() - new Date(app.date_submitted)) / 86400000 > 45;
  }

  function getAppStatusCls(app) {
    if (isAutoStale(app)) return 's-stale';
    return statusCls(app.status);
  }

  ```

---

## Task 4: JS — `renderApps()` updates

- [ ] **Step 1: Update stats — exclude manual stale from "Pending" count**

  Find:
  ```javascript
    const submitted = ALL_APPS.filter(a=>{const s=(a.status||'').toLowerCase();return s.includes('submit')||s.includes('pending');});
  ```
  Replace with:
  ```javascript
    const submitted = ALL_APPS.filter(a=>{const s=(a.status||'').toLowerCase();return (s.includes('submit')||s.includes('pending'))&&!s.includes('stale');});
  ```

- [ ] **Step 2: Add stale to sort order**

  Find:
  ```javascript
    const statusOrder = {'interview':0,'offer':0,'s-interview':0,'pending':1,'submit':1,'reject':2,'not yet':3};
  ```
  Replace with:
  ```javascript
    const statusOrder = {'interview':0,'offer':0,'s-interview':0,'pending':1,'submit':1,'stale':2,'reject':3,'not yet':4};
  ```

- [ ] **Step 3: Update filter logic to handle 'Stale' filter**

  Find:
  ```javascript
    const matchStatus = activeAppFilter === 'All' ||
      (a.status || '').toLowerCase().includes(activeAppFilter.toLowerCase().split(' ')[0]);
  ```
  Replace with:
  ```javascript
    const matchStatus = activeAppFilter === 'All' ||
      (activeAppFilter === 'Stale'
        ? (isAutoStale(a) || (a.status || '').toLowerCase().includes('stale'))
        : (a.status || '').toLowerCase().includes(activeAppFilter.toLowerCase().split(' ')[0]));
  ```

- [ ] **Step 4: Update row template — use `getAppStatusCls` + add stale title tooltip**

  Find:
  ```javascript
              <select class="status-select ${statusCls(a.status)}"
                data-id="${safeId}"
                onchange="updateAppStatus(this.dataset.id, this.value, this)">
                ${statusOptionsHtml(a.status)}
              </select>
  ```
  Replace with:
  ```javascript
              <select class="status-select ${getAppStatusCls(a)}"
                data-id="${safeId}"
                onchange="updateAppStatus(this.dataset.id, this.value, this)"
                ${isAutoStale(a) ? 'title="No response in 45+ days"' : ''}>
                ${statusOptionsHtml(a.status)}
              </select>
  ```

---

## Task 5: Verify

- [ ] Open `index.html` in browser, go to My Applications tab
- [ ] An app submitted > 45 days ago with "Submitted – Pending Response" status → select turns amber, has tooltip "No response in 45+ days"
- [ ] An app submitted < 45 days ago → no amber, normal blue
- [ ] Edit an app → modal shows "⏰ Stale – No Response" in the status dropdown
- [ ] Set an app to "Stale – No Response" manually → select turns amber, Supabase saves correctly
- [ ] "⏰ Stale" filter button shows both auto-stale and manual stale apps
- [ ] "⏳ Pending" filter does NOT show manual stale apps
- [ ] Light and dark themes both render amber correctly

---

## Task 6: Commit

```
git add index.html
git commit -m "feat: add staleness indicator for pending applications"
```

Push via GitHub Desktop.
