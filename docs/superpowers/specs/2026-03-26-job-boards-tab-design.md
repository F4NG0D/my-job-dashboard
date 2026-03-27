# Job Boards Tab — Design Spec

**Date:** 2026-03-26
**Status:** Approved

## Summary

Replace the existing "Job Leads" tab (which scraped live job postings via APIs and AI scoring) with a simple "Job Boards" tab that displays 10 curated job board links as large clickable cards. Each card opens the target site in a new tab.

## Motivation

The Job Leads tab never reliably surfaced quality IB/finance roles due to scraping limitations and noisy API results. A curated directory of job boards gives faster, more reliable access to where the best roles actually appear.

## Layout

- **Tab name:** "Job Boards" (was "Job Leads")
- **Tab icon:** 🌐 (replacing 🔍)
- **Layout:** 2-column CSS grid, cards stack downward
- **Card size:** Larger than standard — generous padding, slightly larger font
- **Card contents:**
  - Favicon logo (via `https://www.google.com/s2/favicons?domain=<domain>&sz=64`), 40×40px
  - Site name (bold, ~16px)
  - Domain label (muted, ~12px)
  - External link arrow (↗) right-aligned, muted
- **Interaction:** entire card is clickable, opens URL in new tab (`target="_blank" rel="noopener"`)
- **Hover state:** subtle highlight matching the site's existing card hover style

## Job Boards List

| Name | URL | Domain (for favicon) |
|---|---|---|
| LinkedIn Jobs | https://www.linkedin.com/jobs/ | linkedin.com |
| Fordham Handshake | https://fordham.joinhandshake.com/job-search/10683063?page=1&per_page=25 | joinhandshake.com |
| eFinancialCareers | https://www.efinancialcareers.com/ | efinancialcareers.com |
| Wall Street Oasis | https://www.wallstreetoasis.com/finance-jobs | wallstreetoasis.com |
| Greenhouse | https://my.greenhouse.io/jobs | greenhouse.io |
| Indeed | https://www.indeed.com/ | indeed.com |
| Glassdoor | https://www.glassdoor.com/Job/ | glassdoor.com |
| New Grad Jobs | https://www.newgrad-jobs.com/?k=af | newgrad-jobs.com |
| Buyside Careers | https://www.buysidecareers.com/ | buysidecareers.com |
| Selby Jennings | https://www.selbyjennings.com/jobs | selbyjennings.com |

## What Gets Removed

**HTML:**
- Search bar and filter buttons (All, IB, Corp Finance, FP&A, etc.)
- Sort dropdown
- Refresh button
- Stats strip (Strong Fit, Good Fit, etc.)
- `pill-leads` counter element
- `#last-updated` element in the sticky header — remove entirely (no other tab populates it)

**JS Functions:**
- `searchLiveJobLeads()`, `fetchFromFirmCareerPages()`, `resolveLeadLink()`, `sanitizeLeadLinks()`
- `renderLeads()`, `refreshLeads()`, `setLeadFilter()`
- `loadLeads()` — and its startup call at page load
- `addLeadToApps()`
- `fetchFromTheMuse()`, `fetchFromIndeedRSS()`, `fetchFromGreenhouse()`, `fetchFromIndeedByCompany()`
- `fetchFromLever()`
- `evaluateJobsWithClaude()`
- `buildJobSearchURL()`

**State / Constants:**
- `activeLeadFilter` state variable
- `LEADS_URL` constant
- `IB_FIRMS_DIRECTORY` constant
- `LEVER_FIRMS` constant (referenced by `fetchFromLever` but never defined — dead reference)
- `CAT_COLORS` object
- `catColor` helper function (only used by `renderLeads()`)
- `window.INLINE_JOBS` / `window.INLINE_LAST_UPDATED` inline data blob (and the large hardcoded JSON)

**Function modifications (not full removal):**
- `updateHeaderCount()` — remove the `if (activeTab === 'leads')` branch that reads `ALL_LEADS.length`; replace with a static label (e.g., "10 boards") or remove the branch entirely

## What Stays the Same

- Tab button position (first tab, default active)
- `switchTab('leads')` ID kept as `leads` to avoid breaking tab switching logic
- Header and mobile tab row button IDs (`btn-leads`, `mbtn-leads`) kept — only label text and icon updated
- All other tabs (My Applications, Cover Letter Generator, Interview Prep) unchanged

## Implementation Notes

- Favicons are loaded client-side via Google's favicon CDN — no backend needed
- No JS logic required for this tab; pure static HTML
- Cards implemented as `<a href="..." target="_blank" rel="noopener noreferrer">` for accessibility
- **Mobile:** grid collapses to 1 column on mobile via `@media` breakpoint (consistent with existing responsive patterns in the codebase)
- Both dark and light themes handled automatically via existing CSS custom properties — no new theme variables needed
