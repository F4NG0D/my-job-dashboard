# My Job Dashboard

A personal, AI-powered job search dashboard built to simplify the three biggest friction points in a finance job hunt — finding leads, tracking applications, and writing cover letters.

No subscriptions. No bloated job boards. Just a clean, fast tool built exactly for the way I search.

---

## Why I Built This

Job searching in finance is noisy. You're bouncing between LinkedIn, company career pages, spreadsheets, and ChatGPT tabs — all at once. I wanted one place that:

- **Surfaces relevant leads automatically**, scored against my actual background
- **Tracks every application** with live status updates synced to a database
- **Drafts tailored cover letters** using AI, in seconds, from a resume upload

This dashboard does all three.

---

## Features

### 🔍 Job Leads
- Curated finance job cards (IB, Advisory, FP&A, Asset Management, Private Credit, and more)
- AI-generated fit ratings: **Strong Fit / Good Fit / Stretch**
- Personalized descriptions explaining *why* each role matches your profile
- Filter by category, sort by fit or date, full-text search
- Live search powered by Claude — scrapes 190+ firm career pages and scores results against your profile
- Leads refresh daily via a scheduled task; new ones are flagged with a **NEW** badge

### 📋 My Applications
- Full application tracker backed by **Supabase** (Postgres)
- Add, edit, and delete applications via modal forms
- Update status inline directly from the table dropdown — saves to the database instantly
- Pipeline bar visualizing your funnel: Pending → Interview → Rejected
- Stats strip: total tracked, pending, rejected, interviews, and response rate

### ✍️ Cover Letter Generator
- Upload your resume (PDF, DOCX, or TXT) or paste it directly
- Paste or auto-fetch a job description from any URL
- Hit Generate — Claude writes a tailored, professional cover letter in seconds
- Copy to clipboard or use the fallback prompt for Claude.ai if no API key is set

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML, CSS, JavaScript (zero frameworks) |
| Database | Supabase (PostgreSQL via REST API) |
| AI | Anthropic Claude API (claude-sonnet-4-6) |
| PDF parsing | PDF.js (lazy-loaded) |
| DOCX parsing | Mammoth.js (lazy-loaded) |
| Hosting | GitHub Pages / Vercel |

---

## Getting Started Locally

No build step required. Just serve the files with any static HTTP server:

```bash
# Python (if installed)
python3 -m http.server 8080

# Node.js
npx serve .

# Ruby (ships with macOS)
ruby -e "require 'webrick'; WEBrick::HTTPServer.new(Port: 8080, DocumentRoot: '.').start"
```

Then open `http://localhost:8080` in your browser.

---

## Configuration

**Anthropic API key** (for AI lead search + cover letter generation)

Open the ⚙️ Settings modal in the top-right corner of the app and paste your key. It's stored only in your browser's `localStorage` — never transmitted anywhere except the Anthropic API.

Alternatively, hardcode it in `index.html` for local use only (do not commit to a public repo):
```js
const ANTHROPIC_KEY = 'sk-ant-api03-…';
```

**Supabase** (for application tracking)

The `SUPABASE_URL` and `SUPABASE_ANON` key are already configured in `index.html`. The anon key is safe to be public — it only has the permissions you define in Supabase's Row Level Security policies.

---

## Project Structure

```
My-Job-Dashboard/
├── index.html       # Entire app — UI, styles, and logic in one file
└── jobs.json        # Job leads data, updated by a scheduled task
```

---

## Roadmap

- [ ] "Save to Applications" button directly from a lead card
- [ ] Kanban board view for application pipeline
- [ ] Interview prep notes per company
- [ ] Vercel serverless proxy for public AI access (no user API key required)
- [ ] Analytics dashboard — application trends over time
