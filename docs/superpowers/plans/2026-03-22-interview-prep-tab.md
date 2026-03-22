# Interview Prep Tab — Implementation Plan

> **For agentic workers:** Use superpowers:executing-plans to implement task-by-task.

**Goal:** Add a 4th "Interview Prep" tab with sidebar card list + right prep sheet panel. New "Interviewing" status. Prep sheets generated on demand via Claude and persisted to Supabase.

**Architecture:** All changes in `index.html`. New tab mirrors Cover Letter tab pattern. New Supabase table `Interview Prep` already created by user.

**Tech Stack:** Vanilla HTML/CSS/JS, Supabase REST, Claude API (claude-sonnet-4-6).

---

## Task 1: CSS

Add to `<style>` block.

### Step 1 — `--status-interviewing` CSS vars

After `--status-stale` in dark theme:
```css
      --status-interviewing: #06b6d4;
```
After `--status-stale` in light theme:
```css
      --status-interviewing: #0891b2;
```

### Step 2 — `.s-interviewing` pill + select rules

After `.s-stale` badge rule:
```css
    .s-interviewing { background: rgba(6,182,212,0.15); color: var(--status-interviewing); }
```
After `.status-select.s-stale` rule:
```css
    .status-select.s-interviewing { background: rgba(6,182,212,0.10); color: var(--status-interviewing); border-color: rgba(6,182,212,0.35); }
```

### Step 3 — Interview Prep layout CSS

After the `.cl-status` rule (end of Cover Letter CSS section):
```css
    /* ── Interview Prep tab ── */
    .ip-layout {
      display: grid; grid-template-columns: 280px 1fr; gap: 20px;
      padding: 24px 32px; max-width: 1280px; margin: 0 auto;
    }
    @media (max-width: 900px) { .ip-layout { grid-template-columns: 1fr; } }
    .ip-sidebar { display: flex; flex-direction: column; gap: 10px; }
    .ip-sidebar-title { font-size: 13px; font-weight: 700; color: var(--text-muted); text-transform: uppercase; letter-spacing: 0.06em; margin-bottom: 4px; }
    .ip-card {
      background: var(--surface); border: 1px solid var(--border);
      border-radius: 12px; padding: 14px 16px; cursor: pointer;
      transition: border-color 0.15s, background 0.15s;
    }
    .ip-card:hover { border-color: var(--accent); }
    .ip-card.active { border-color: var(--accent); background: var(--accent-soft); }
    .ip-card-header { display: flex; align-items: center; gap: 10px; margin-bottom: 8px; }
    .ip-card-logo {
      width: 34px; height: 34px; border-radius: 8px;
      background: var(--surface2); border: 1px solid var(--border);
      display: flex; align-items: center; justify-content: center;
      font-weight: 800; font-size: 11px; color: var(--accent); flex-shrink: 0;
    }
    .ip-card-company { font-weight: 700; font-size: 13px; }
    .ip-card-role { font-size: 11px; color: var(--text-muted); margin-top: 1px; }
    .ip-badge-ready { font-size: 10px; font-weight: 700; padding: 2px 8px; border-radius: 4px; background: rgba(70,209,96,0.15); color: var(--strong); }
    .ip-badge-generate { font-size: 10px; font-weight: 700; padding: 2px 8px; border-radius: 4px; background: var(--accent-soft); color: var(--accent); }
    .ip-panel {
      background: var(--surface); border: 1px solid var(--border);
      border-radius: 12px; padding: 20px; display: flex; flex-direction: column; gap: 12px; min-height: 400px;
    }
    .ip-panel-header { display: flex; align-items: center; justify-content: space-between; flex-wrap: wrap; gap: 8px; }
    .ip-panel-title { font-size: 15px; font-weight: 700; }
    .ip-panel-meta { font-size: 12px; color: var(--text-muted); margin-top: 2px; }
    .ip-panel-actions { display: flex; gap: 8px; flex-wrap: wrap; }
    .ip-output {
      width: 100%; background: var(--surface2); border: 1px solid var(--border);
      border-radius: 8px; padding: 14px; font-size: 13px; color: var(--text);
      outline: none; resize: vertical; min-height: 480px; font-family: inherit;
      line-height: 1.7; flex: 1;
    }
    .ip-generate-btn {
      width: 100%; background: linear-gradient(135deg, var(--accent), var(--accent2));
      border: none; border-radius: 10px; padding: 14px 24px;
      font-size: 15px; font-weight: 700; color: #fff; cursor: pointer;
      transition: opacity 0.15s; margin-top: 4px;
    }
    .ip-generate-btn:hover { opacity: 0.9; }
    .ip-generate-btn:disabled { opacity: 0.5; cursor: not-allowed; }
    .ip-empty {
      display: flex; flex-direction: column; align-items: center; justify-content: center;
      flex: 1; color: var(--text-muted); text-align: center; padding: 40px 20px; gap: 12px;
    }
    .ip-empty-icon { font-size: 40px; }
    .ip-status { font-size: 12px; color: var(--accent2); min-height: 18px; }
```

---

## Task 2: HTML — Tab buttons + panel

### Step 1 — Add tab button in header nav

After the Cover Letter Generator tab button:
```html
      <button class="tab-btn" onclick="switchTab('interviewprep')" id="btn-interviewprep">
        🎯 Interview Prep
      </button>
```

### Step 2 — Add mobile tab button

After the Cover Letter mobile button:
```html
  <button class="filter-btn"        onclick="switchTab('interviewprep')"  id="mbtn-interviewprep">🎯 Interview Prep</button>
```

### Step 3 — Add panel HTML

After the closing `</div>` of `panel-coverletter`, add:
```html
  <!-- ───────────── TAB 4: INTERVIEW PREP ───────────── -->
  <div id="panel-interviewprep" class="tab-panel">
    <div class="ip-layout">
      <!-- Left sidebar -->
      <div class="ip-sidebar" id="ip-sidebar">
        <div class="ip-sidebar-title">Interviewing</div>
        <div id="ip-cards"></div>
      </div>
      <!-- Right panel -->
      <div class="ip-panel" id="ip-panel">
        <div class="ip-empty" id="ip-empty">
          <div class="ip-empty-icon">🎯</div>
          <div style="font-weight:600;font-size:15px;color:var(--text)">Select a company to prep for</div>
          <div style="font-size:13px">Pick an application from the sidebar to view or generate your interview prep sheet.</div>
        </div>
      </div>
    </div>
  </div>
```

### Step 4 — Add "Interviewing" to Add modal `#f-status`

Between "Submitted – Pending Response" and "Interviewed":
```html
            <option value="Interviewing">Interviewing</option>
```

### Step 5 — Add "Interviewing" to Edit modal `#ef-status`

Between "Submitted – Pending Response" and "Interviewed":
```html
            <option value="Interviewing">Interviewing</option>
```

---

## Task 3: JS — Status helpers + state

### Step 1 — Add to `STATUS_OPTIONS`

Between "Interviewed" and "Offer Extended":
```javascript
  { value: 'Interviewing',               cls: 's-interviewing', label: 'Interviewing' },
```

### Step 2 — `normalizeStatusToOption()` — add case

Before the `'Interviewed'` return:
```javascript
  if (l === 'interviewing')   return 'Interviewing';
```

### Step 3 — `statusCls()` — add case

Before the `'interview'` return:
```javascript
  if (l === 'interviewing')   return 's-interviewing';
```

### Step 4 — `statusInfo()` — add case

Before the `'interview'` / `'offer'` block:
```javascript
  if (l === 'interviewing')
    return { cls:'s-interviewing', label:'Interviewing' };
```

### Step 5 — Update `TAB_ORDER`

```javascript
const TAB_ORDER = ['leads', 'applications', 'coverletter', 'interviewprep'];
```

### Step 6 — Add global state (after existing globals)

```javascript
let ALL_PREP_SHEETS  = [];
let activePrepAppId  = null;
let prepSheetsLoaded = false;
```

---

## Task 4: JS — Interview Prep functions

Add all functions together after the Cover Letter section, before the shared `callClaudeAPI` function.

```javascript
// ─── TAB 4: INTERVIEW PREP ─────────────────────────────────────────────────

async function loadPrepSheets() {
  try {
    const res = await fetch(
      `${SUPABASE_URL}/rest/v1/Interview%20Prep?select=*&order=created_at.desc`,
      { headers: SB_HEADERS }
    );
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    ALL_PREP_SHEETS = await res.json();
  } catch(e) {
    console.warn('Could not load prep sheets:', e.message);
    ALL_PREP_SHEETS = [];
  }
  prepSheetsLoaded = true;
  renderPrepSidebar();
}

function renderPrepSidebar() {
  const interviewing = ALL_APPS.filter(a =>
    (a.status || '').toLowerCase() === 'interviewing'
  );
  const cards = document.getElementById('ip-cards');
  if (!cards) return;

  if (!interviewing.length) {
    cards.innerHTML = `<div style="font-size:13px;color:var(--text-muted);padding:12px 0">
      No applications set to "Interviewing" yet.<br>Update a status in My Applications to get started.
    </div>`;
    return;
  }

  cards.innerHTML = interviewing.map(a => {
    const sheet = ALL_PREP_SHEETS.find(s => s.app_id === a.id);
    const badge = sheet
      ? `<span class="ip-badge-ready">Sheet ready</span>`
      : `<span class="ip-badge-generate">Generate</span>`;
    const isActive = a.id === activePrepAppId;
    return `<div class="ip-card${isActive ? ' active' : ''}" onclick="selectPrepCard('${(a.id||'').replace(/'/g,"\\'")}')">
      <div class="ip-card-header">
        <div class="ip-card-logo">${initials(a.company)}</div>
        <div>
          <div class="ip-card-company">${a.company}</div>
          <div class="ip-card-role">${a.role}</div>
        </div>
      </div>
      ${badge}
    </div>`;
  }).join('');
}

function selectPrepCard(appId) {
  activePrepAppId = appId;
  renderPrepSidebar(); // re-render to update active state
  const app    = ALL_APPS.find(a => a.id === appId);
  const sheet  = ALL_PREP_SHEETS.find(s => s.app_id === appId);
  const panel  = document.getElementById('ip-panel');
  if (!app || !panel) return;

  if (sheet) {
    renderPrepSheet(app, sheet);
  } else {
    panel.innerHTML = `
      <div class="ip-panel-header">
        <div>
          <div class="ip-panel-title">${app.company} — ${app.role}</div>
          <div class="ip-panel-meta">No prep sheet yet</div>
        </div>
      </div>
      <div class="ip-status" id="ip-status"></div>
      <button class="ip-generate-btn" id="ip-gen-btn" onclick="generatePrepSheet('${(appId||'').replace(/'/g,"\\'")}')">
        🎯 Generate Interview Prep Sheet
      </button>`;
  }
}

function renderPrepSheet(app, sheet) {
  const panel = document.getElementById('ip-panel');
  const genDate = sheet.generated_at
    ? new Date(sheet.generated_at).toLocaleDateString('en-US', {month:'short', day:'numeric', year:'numeric'})
    : '';
  const escapedAppId = (app.id||'').replace(/'/g,"\\'");
  const escapedPrepId = (sheet.id||'').replace(/'/g,"\\'");
  panel.innerHTML = `
    <div class="ip-panel-header">
      <div>
        <div class="ip-panel-title">${app.company} — ${app.role}</div>
        <div class="ip-panel-meta">Generated ${genDate}</div>
      </div>
      <div class="ip-panel-actions">
        <button class="cl-copy-btn" onclick="copyPrepSheet()">Copy</button>
        <button class="cl-copy-btn" onclick="generatePrepSheet('${escapedAppId}')">Regenerate</button>
        <button class="btn-danger" style="padding:6px 14px;font-size:12px" onclick="deletePrepSheet('${escapedPrepId}','${escapedAppId}')">🗑 Delete</button>
      </div>
    </div>
    <div class="ip-status" id="ip-status"></div>
    <textarea class="ip-output" id="ip-output" readonly>${sheet.content || ''}</textarea>`;
}

function copyPrepSheet() {
  const el = document.getElementById('ip-output');
  if (!el) return;
  navigator.clipboard.writeText(el.value)
    .then(() => { const s = document.getElementById('ip-status'); if(s) { s.textContent = '✓ Copied!'; setTimeout(()=>{ s.textContent=''; }, 2000); } })
    .catch(() => {});
}

async function generatePrepSheet(appId) {
  const app = ALL_APPS.find(a => a.id === appId);
  if (!app) return;

  const btn    = document.getElementById('ip-gen-btn');
  const status = document.getElementById('ip-status');
  if (btn) { btn.disabled = true; btn.textContent = 'Generating…'; }
  if (status) status.textContent = 'Calling Claude — this may take 20–30 seconds…';

  const jobContext = app.link ? `\nJob URL: ${app.link}` : '';
  const prompt = `Generate a comprehensive interview prep sheet for the following role:

Company: ${app.company}
Role: ${app.role}${jobContext}

${ANTONIO_PROFILE}

Structure your response with exactly these four sections, each with a clear heading:

1. COMPANY OVERVIEW
Provide the firm's history, what they do, their culture, recent notable deals, and key clients. Be specific and research-level detailed.

2. YOUR PITCH
Write a tailored 60–90 second verbal intro script Antonio should practice. Reference his OCBC deal experience, GPA, and specific skills relevant to this firm.

3. EXPECTED QUESTIONS & SUGGESTED ANSWERS
List 8 common IB/finance interview questions with concise, tailored answers that tie back to Antonio's background and this specific firm.

4. QUESTIONS TO ASK THEM
List 5 smart, firm-specific questions Antonio can ask the interviewer that show he's done his research.

Be detailed, specific, and practical — this is a real interview prep tool.`;

  try {
    const data = await callClaudeAPI(
      [{ role: 'user', content: prompt }],
      3500,
      'You are an expert investment banking interview coach preparing a candidate for interviews. Be detailed, specific, and practical.'
    );
    const content = data.content[0].text.trim();
    const now     = new Date().toISOString();

    // Check if a sheet already exists for this app — overwrite if so
    const existing = ALL_PREP_SHEETS.find(s => s.app_id === appId);
    let savedSheet;

    if (existing) {
      // PATCH existing record
      const res = await fetch(
        `${SUPABASE_URL}/rest/v1/Interview%20Prep?id=eq.${encodeURIComponent(existing.id)}`,
        { method: 'PATCH', headers: SB_POST_HEADERS, body: JSON.stringify({ content, generated_at: now }) }
      );
      if (!res.ok) throw new Error(`Save failed: HTTP ${res.status}`);
      existing.content      = content;
      existing.generated_at = now;
      savedSheet = existing;
    } else {
      // POST new record
      const res = await fetch(
        `${SUPABASE_URL}/rest/v1/Interview%20Prep`,
        { method: 'POST', headers: { ...SB_POST_HEADERS, 'Prefer': 'return=representation' },
          body: JSON.stringify({ app_id: appId, company: app.company, role: app.role, content, generated_at: now }) }
      );
      if (!res.ok) throw new Error(`Save failed: HTTP ${res.status}`);
      const rows = await res.json();
      savedSheet = rows[0] || { id: Date.now().toString(), app_id: appId, company: app.company, role: app.role, content, generated_at: now };
      ALL_PREP_SHEETS.push(savedSheet);
    }

    renderPrepSidebar();
    renderPrepSheet(app, savedSheet);

  } catch(e) {
    if (status) status.textContent = `⚠ ${e.message === 'NO_API_KEY' ? 'Add your API key in Settings' : e.message}`;
    if (btn) { btn.disabled = false; btn.textContent = '🎯 Generate Interview Prep Sheet'; }
  }
}

async function deletePrepSheet(prepId, appId) {
  if (!confirm('Delete this prep sheet? This cannot be undone.')) return;
  try {
    const res = await fetch(
      `${SUPABASE_URL}/rest/v1/Interview%20Prep?id=eq.${encodeURIComponent(prepId)}`,
      { method: 'DELETE', headers: SB_POST_HEADERS }
    );
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    ALL_PREP_SHEETS = ALL_PREP_SHEETS.filter(s => s.id !== prepId);
    activePrepAppId = null;
    renderPrepSidebar();
    const panel = document.getElementById('ip-panel');
    if (panel) panel.innerHTML = `<div class="ip-empty"><div class="ip-empty-icon">🗑</div><div style="color:var(--text);font-weight:600">Prep sheet deleted.</div></div>`;
  } catch(e) {
    alert(`Could not delete: ${e.message}`);
  }
}
```

---

## Task 5: JS — switchTab() + init

### Step 1 — Handle interviewprep tab in `switchTab()`

In `switchTab()`, after the `if (tab === 'coverletter')` block, add:

```javascript
  if (tab === 'interviewprep' && !prepSheetsLoaded) {
    loadPrepSheets();
  }
```

---

## Task 6: Verify

- [ ] 4th tab appears in header and mobile nav
- [ ] "Interviewing" appears in all status dropdowns, styled teal
- [ ] Switching to Interview Prep tab loads prep sheets from Supabase
- [ ] "Interviewing" apps appear as cards in sidebar
- [ ] Clicking a card with no sheet shows "Generate" button
- [ ] Clicking Generate calls Claude and saves to Supabase
- [ ] Sheet renders in right panel with Copy / Regenerate / Delete
- [ ] Delete removes from Supabase and clears panel
- [ ] Light mode renders correctly

---

## Task 7: Commit

```
git add index.html
git commit -m "feat: add Interview Prep tab with Supabase-persisted prep sheets"
```

Push via GitHub Desktop.
