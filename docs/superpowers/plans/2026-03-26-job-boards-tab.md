# Job Boards Tab Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the broken "Job Leads" scraping tab with a clean "Job Boards" tab that displays 10 curated job board links as large clickable cards with real favicons.

**Architecture:** All changes are in `index.html` (single-file static site). The work is purely subtractive + HTML replacement — no new JS logic, no new dependencies. Tasks proceed in this order: HTML shell updates first, then JS cleanup, then the large scraping pipeline removal.

**Tech Stack:** Vanilla HTML/CSS/JS, no framework, no build step. Google Favicon CDN for site icons.

---

## Spec Reference

`docs/superpowers/specs/2026-03-26-job-boards-tab-design.md`

---

## File Modified

- `index.html` — all changes are in this file

---

## Task 1: Update tab buttons and remove header elements

**Files:**
- Modify: `index.html:904-906` (desktop tab button)
- Modify: `index.html:920` (last-updated element — remove)
- Modify: `index.html:929` (mobile tab button)

- [ ] **Step 1: Update desktop tab button label and icon**

  Find (lines 904-906):
  ```html
  <button class="tab-btn active" onclick="switchTab('leads')" id="btn-leads">
    🔍 Job Leads <span class="tab-pill" id="pill-leads">0</span>
  </button>
  ```

  Replace with:
  ```html
  <button class="tab-btn active" onclick="switchTab('leads')" id="btn-leads">
    🌐 Job Boards
  </button>
  ```

- [ ] **Step 2: Remove the `#last-updated` element**

  Find (line 920):
  ```html
        <div class="last-updated" id="last-updated"></div>
  ```
  Delete this line entirely.

- [ ] **Step 3: Update mobile tab button label and icon**

  Find (line 929):
  ```html
    <button class="filter-btn active" onclick="switchTab('leads')"        id="mbtn-leads">🔍 Job Leads</button>
  ```

  Replace with:
  ```html
    <button class="filter-btn active" onclick="switchTab('leads')"        id="mbtn-leads">🌐 Job Boards</button>
  ```

- [ ] **Step 4: Verify in browser**

  Open `index.html` in a browser. The first tab should read "🌐 Job Boards". No counter pill. The header timestamp is gone.

---

## Task 2: Replace panel-leads HTML with job boards card grid

**Files:**
- Modify: `index.html:938-989` (entire panel-leads inner content)

- [ ] **Step 1: Replace the panel-leads content**

  Find the entire block from line 938 through 989:
  ```html
    <!-- ───────────── TAB 1: JOB LEADS ───────────── -->
    <div id="panel-leads" class="tab-panel active">

      <div class="controls">
        ...
      </div>

      <div class="stats-strip">
        ...
      </div>

      <div class="grid" id="leads-grid"></div>
    </div>
  ```

  Replace with:
  ```html
    <!-- ───────────── TAB 1: JOB BOARDS ───────────── -->
    <div id="panel-leads" class="tab-panel active">
      <div style="display:grid;grid-template-columns:1fr 1fr;gap:20px;padding:24px 0">

        <a href="https://www.linkedin.com/jobs/" target="_blank" rel="noopener noreferrer" class="job-board-card">
          <img src="https://www.google.com/s2/favicons?domain=linkedin.com&sz=64" class="job-board-favicon" />
          <div class="job-board-info">
            <div class="job-board-name">LinkedIn Jobs</div>
            <div class="job-board-domain">linkedin.com</div>
          </div>
          <span class="job-board-arrow">↗</span>
        </a>

        <a href="https://fordham.joinhandshake.com/job-search/10683063?page=1&per_page=25" target="_blank" rel="noopener noreferrer" class="job-board-card">
          <img src="https://www.google.com/s2/favicons?domain=joinhandshake.com&sz=64" class="job-board-favicon" />
          <div class="job-board-info">
            <div class="job-board-name">Fordham Handshake</div>
            <div class="job-board-domain">joinhandshake.com</div>
          </div>
          <span class="job-board-arrow">↗</span>
        </a>

        <a href="https://www.efinancialcareers.com/" target="_blank" rel="noopener noreferrer" class="job-board-card">
          <img src="https://www.google.com/s2/favicons?domain=efinancialcareers.com&sz=64" class="job-board-favicon" />
          <div class="job-board-info">
            <div class="job-board-name">eFinancialCareers</div>
            <div class="job-board-domain">efinancialcareers.com</div>
          </div>
          <span class="job-board-arrow">↗</span>
        </a>

        <a href="https://www.wallstreetoasis.com/finance-jobs" target="_blank" rel="noopener noreferrer" class="job-board-card">
          <img src="https://www.google.com/s2/favicons?domain=wallstreetoasis.com&sz=64" class="job-board-favicon" />
          <div class="job-board-info">
            <div class="job-board-name">Wall Street Oasis</div>
            <div class="job-board-domain">wallstreetoasis.com</div>
          </div>
          <span class="job-board-arrow">↗</span>
        </a>

        <a href="https://my.greenhouse.io/jobs" target="_blank" rel="noopener noreferrer" class="job-board-card">
          <img src="https://www.google.com/s2/favicons?domain=greenhouse.io&sz=64" class="job-board-favicon" />
          <div class="job-board-info">
            <div class="job-board-name">Greenhouse</div>
            <div class="job-board-domain">greenhouse.io</div>
          </div>
          <span class="job-board-arrow">↗</span>
        </a>

        <a href="https://www.indeed.com/" target="_blank" rel="noopener noreferrer" class="job-board-card">
          <img src="https://www.google.com/s2/favicons?domain=indeed.com&sz=64" class="job-board-favicon" />
          <div class="job-board-info">
            <div class="job-board-name">Indeed</div>
            <div class="job-board-domain">indeed.com</div>
          </div>
          <span class="job-board-arrow">↗</span>
        </a>

        <a href="https://www.glassdoor.com/Job/" target="_blank" rel="noopener noreferrer" class="job-board-card">
          <img src="https://www.google.com/s2/favicons?domain=glassdoor.com&sz=64" class="job-board-favicon" />
          <div class="job-board-info">
            <div class="job-board-name">Glassdoor</div>
            <div class="job-board-domain">glassdoor.com</div>
          </div>
          <span class="job-board-arrow">↗</span>
        </a>

        <a href="https://www.newgrad-jobs.com/?k=af" target="_blank" rel="noopener noreferrer" class="job-board-card">
          <img src="https://www.google.com/s2/favicons?domain=newgrad-jobs.com&sz=64" class="job-board-favicon" />
          <div class="job-board-info">
            <div class="job-board-name">New Grad Jobs</div>
            <div class="job-board-domain">newgrad-jobs.com</div>
          </div>
          <span class="job-board-arrow">↗</span>
        </a>

        <a href="https://www.buysidecareers.com/" target="_blank" rel="noopener noreferrer" class="job-board-card">
          <img src="https://www.google.com/s2/favicons?domain=buysidecareers.com&sz=64" class="job-board-favicon" />
          <div class="job-board-info">
            <div class="job-board-name">Buyside Careers</div>
            <div class="job-board-domain">buysidecareers.com</div>
          </div>
          <span class="job-board-arrow">↗</span>
        </a>

        <a href="https://www.selbyjennings.com/jobs" target="_blank" rel="noopener noreferrer" class="job-board-card">
          <img src="https://www.google.com/s2/favicons?domain=selbyjennings.com&sz=64" class="job-board-favicon" />
          <div class="job-board-info">
            <div class="job-board-name">Selby Jennings</div>
            <div class="job-board-domain">selbyjennings.com</div>
          </div>
          <span class="job-board-arrow">↗</span>
        </a>

      </div>
    </div>
  ```

- [ ] **Step 2: Add card CSS styles**

  Find the `.last-updated` CSS rule (around line 141) and add the following styles immediately after it:

  ```css
  .job-board-card {
    display: flex;
    align-items: center;
    gap: 18px;
    padding: 28px 22px;
    border: 1px solid var(--border);
    border-radius: 12px;
    background: var(--card);
    text-decoration: none;
    color: var(--text);
    cursor: pointer;
    transition: background 0.15s, border-color 0.15s;
  }
  .job-board-card:hover {
    background: var(--card-hover, var(--surface));
    border-color: var(--accent2);
  }
  .job-board-favicon {
    width: 42px;
    height: 42px;
    border-radius: 8px;
    flex-shrink: 0;
  }
  .job-board-info { flex: 1; min-width: 0; }
  .job-board-name { font-weight: 700; font-size: 16px; }
  .job-board-domain { font-size: 12px; color: var(--text-muted); margin-top: 3px; }
  .job-board-arrow { opacity: 0.35; font-size: 16px; flex-shrink: 0; }
  @media (max-width: 600px) {
    #panel-leads > div[style*="grid-template-columns"] {
      grid-template-columns: 1fr !important;
    }
  }
  ```

- [ ] **Step 3: Verify in browser**

  Open `index.html`. The Job Boards tab should show 10 large cards in a 2-column grid. Each card has the site's favicon, name, and domain. Clicking any card opens the site in a new tab. On narrow window width, the grid should collapse to 1 column.

- [ ] **Step 4: Commit**

  ```
  git add index.html
  git commit -m "feat: replace Job Leads tab with Job Boards card grid"
  ```

---

## Task 3: Remove leads state, constants, and the INLINE_JOBS blob

**Files:**
- Modify: `index.html:1369` (LEADS_URL)
- Modify: `index.html:1391-1392` (INLINE_JOBS blob + INLINE_LAST_UPDATED)
- Modify: `index.html:1395` (ALL_LEADS)
- Modify: `index.html:1397` (activeLeadFilter)
- Modify: `index.html:1406-1416` (CAT_COLORS + catColor)

- [ ] **Step 1: Remove `LEADS_URL`**

  Find and delete (line 1369):
  ```js
  const LEADS_URL = 'jobs.json';
  ```

- [ ] **Step 2: Remove INLINE_JOBS blob and INLINE_LAST_UPDATED**

  Find and delete (lines 1391-1392):
  ```js
  window.INLINE_JOBS = [{"id":"goldman-sachs-ib-new-analyst-20260313",...}];
  window.INLINE_LAST_UPDATED = "2026-03-13T01:05:58";
  ```
  (The INLINE_JOBS line is very long — the entire single-line JSON array.)

- [ ] **Step 3: Remove `ALL_LEADS` state variable**

  Find and delete (line 1395):
  ```js
  let ALL_LEADS = [];
  ```

- [ ] **Step 4: Remove `activeLeadFilter` state variable**

  Find and delete (line 1397):
  ```js
  let activeLeadFilter = 'All';
  ```

- [ ] **Step 5: Remove `LEVER_FIRMS`**

  Search for `LEVER_FIRMS` in the file. If it appears as a `const` declaration (it may already be deleted as dead code, or may exist as a standalone constant), delete that declaration line.

- [ ] **Step 6: Remove CAT_COLORS and catColor**

  Find and delete (lines 1405-1416, including the comment):
  ```js
  // ─── Category colours ──────────────────────────────────────────────────────
  const CAT_COLORS = {
    'Investment Banking': '#4f6ef7',
    'Corporate Finance':  '#94a3b8',
    'FP&A':               '#a78bfa',
    'Markets / S&T':      '#46d160',
    'Advisory':           '#f59e0b',
    'Private Credit':     '#fb7185',
    'Asset Management':   '#34d399',
    'Corporate Dev':      '#60a5fa',
  };
  const catColor = c => CAT_COLORS[c] || '#8b90b0';
  ```

- [ ] **Step 6: Verify no console errors**

  Open `index.html`, open browser DevTools (F12 → Console). There should be no `ReferenceError` for any of the removed variables.

---

## Task 4: Update `updateHeaderCount()` and remove `loadLeads()` startup call

**Files:**
- Modify: `index.html:1570-1583` (updateHeaderCount function)
- Modify: `index.html:3781` (loadLeads startup call — line number will shift slightly after prior removals)

- [ ] **Step 1: Remove the `leads` branch from `updateHeaderCount()`**

  Find (inside updateHeaderCount, lines 1572-1574):
  ```js
  function updateHeaderCount() {
    const el = document.getElementById('header-count');
    if (activeTab === 'leads') {
      el.textContent = `${ALL_LEADS.length} lead${ALL_LEADS.length !== 1 ? 's' : ''}`;
      el.style.color = 'var(--accent2)';
    } else {
      const submitted = ALL_APPS.filter(a => {
        const s = (a.status||'').toLowerCase();
        return s.includes('submit') || s.includes('pending') || s.includes('reject') || s.includes('interview');
      }).length;
      el.textContent = `${submitted} submitted`;
      el.style.color = 'var(--accent2)';
    }
  }
  ```

  Replace the entire function with:
  ```js
  function updateHeaderCount() {
    const el = document.getElementById('header-count');
    if (activeTab === 'leads') {
      el.textContent = '10 boards';
      el.style.color = 'var(--accent2)';
    } else {
      const submitted = ALL_APPS.filter(a => {
        const s = (a.status||'').toLowerCase();
        return s.includes('submit') || s.includes('pending') || s.includes('reject') || s.includes('interview');
      }).length;
      el.textContent = `${submitted} submitted`;
      el.style.color = 'var(--accent2)';
    }
  }
  ```

- [ ] **Step 2: Remove `loadLeads()` startup call**

  Find (near the bottom of the script, after `loadApps()`):
  ```js
  loadLeads();
  loadApps();
  ```

  Replace with:
  ```js
  loadApps();
  ```

- [ ] **Step 3: Verify no console errors on page load**

  Reload `index.html` in the browser. Console should be clean — no `ReferenceError: loadLeads is not defined` or `ALL_LEADS is not defined`.

---

## Task 5: Remove Tab 1 JS functions block (setLeadFilter through loadLeads/refreshLeads)

**Files:**
- Modify: `index.html:1585-1719` (entire `// ─── TAB 1: LEADS` section)

The block runs from the `// ─── TAB 1: LEADS` comment (line 1585) up to — but not including — the `// ─── TAB 2: APPLICATIONS` comment (line 1720). It contains: `setLeadFilter`, `renderLeads`, `addLeadToApps`, `loadLeads`, `refreshLeads`.

- [ ] **Step 1: Delete the entire TAB 1: LEADS JS section**

  Find and delete everything from:
  ```js
  // ─── TAB 1: LEADS ──────────────────────────────────────────────────────────
  function setLeadFilter(cat) {
  ```

  Up to (but not including):
  ```js
  // ─── TAB 2: APPLICATIONS ───────────────────────────────────────────────────
  ```

- [ ] **Step 2: Verify no console errors**

  Reload in browser. Check DevTools console. All other tabs (My Applications, Cover Letter Generator, Interview Prep) should still work normally.

- [ ] **Step 3: Commit**

  ```
  git add index.html
  git commit -m "chore: remove Job Leads JS functions and dead state/constants"
  ```

---

## Task 6: Remove the scraping pipeline (IB_FIRMS_DIRECTORY + all scraping functions)

This is the largest removal — roughly 600 lines covering the entire job scraping and AI scoring pipeline.

**Files:**
- Modify: `index.html:2963-3153` (IB_FIRMS_DIRECTORY constant)
- Modify: `index.html:3157-~3760` (buildJobSearchURL, resolveLeadLink, sanitizeLeadLinks, fetchFromTheMuse, fetchFromIndeedRSS, fetchFromGreenhouse, fetchFromLever, fetchFromIndeedByCompany, fetchFromFirmCareerPages, evaluateJobsWithClaude, searchLiveJobLeads)

- [ ] **Step 1: Remove `IB_FIRMS_DIRECTORY`**

  Find and delete from:
  ```js
  const IB_FIRMS_DIRECTORY = [
  ```
  Through the closing `];` at line 3153 (including the comment line above it if present).

- [ ] **Step 2: Remove all scraping functions**

  Find and delete each of the following functions in order (they appear sequentially in the file):
  - `function buildJobSearchURL(company, title)` — starts line ~3157
  - `function resolveLeadLink(...)` — starts line ~3163
  - `function sanitizeLeadLinks(leads)` — starts line ~3188
  - `async function fetchFromTheMuse()` — starts line ~3196
  - `async function fetchFromIndeedRSS()` — starts line ~3232
  - `async function fetchFromGreenhouse()` — starts line ~3323
  - `async function fetchFromLever()` — starts line ~3362
  - `async function fetchFromIndeedByCompany()` — starts line ~3405
  - `async function fetchFromFirmCareerPages()` — starts line ~3451
  - `async function evaluateJobsWithClaude(rawJobs)` — starts line ~3568
  - `async function searchLiveJobLeads()` — starts line ~3635 through the end of the function

  Delete the entire contiguous block from `function buildJobSearchURL` through the closing `}` of `searchLiveJobLeads`.

- [ ] **Step 3: Final browser verification**

  Reload `index.html`. Check:
  1. DevTools console is clean (no errors)
  2. "Job Boards" tab shows all 10 cards with favicons
  3. Clicking a card opens the correct URL in a new tab
  4. "My Applications" tab loads and shows the Supabase-backed table
  5. "Cover Letter Generator" tab is functional
  6. "Interview Prep" tab is functional
  7. Theme toggle (dark/light) still works
  8. Resize window narrow — cards collapse to 1 column

- [ ] **Step 4: Final commit**

  ```
  git add index.html
  git commit -m "chore: remove scraping pipeline and IB_FIRMS_DIRECTORY (~600 lines)"
  ```
