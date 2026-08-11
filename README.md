# Smart 1 Sales Builder — Proposals & Insertion Orders (v1)

One dashboard for the whole flow: build a branded proposal for a customer,
save it to a database under a quote number, edit / duplicate / look it up
any time, download it as a Smart 1–branded PDF or a formatted Word doc,
and convert an approved proposal into a real Insertion Order through the
existing IO app — which is **not modified in any way**.

## What's in the folder

| File | What it is |
|---|---|
| `app.py` | Flask backend: database, quote numbers, dashboard stats, branded PDF, Word export, AI routes |
| `templates/index.html` | The whole app: dashboard, guided proposal builder, proposal editor, lookup, convert-to-IO wizard, embedded IO builder tab |
| `requirements.txt` | Python dependencies |
| `render.yaml` | Render blueprint (web service + Postgres database) |

## Run it locally

1. Python 3.10+, then in this folder: `pip install -r requirements.txt`
2. `python app.py`
3. Open `http://localhost:8001`

With no configuration it uses a local SQLite file (`smart1_sales_builder.db`) and
talks to the live IO API at `insertionordersmart.onrender.com` for the AI
features and IO conversion.

## Publish to GitHub (drag-and-drop, no git needed)

1. Go to github.com → **New repository** → name it **smart1-sales-builder** →
   Private → **Create repository**.
2. On the new repo page click **uploading an existing file**.
3. Drag in everything from this folder: `app.py`, `requirements.txt`,
   `render.yaml`, `README.md`, `.gitignore`, `.env.example` — **and the
   `templates` folder** (drag the folder itself so `templates/index.html`
   keeps its path; if your browser flattens it, create the file manually:
   **Add file → Create new file**, type `templates/index.html` as the
   name, and paste the contents).
4. **Commit changes**. `render.yaml` must sit at the repo root — it
   already does in this folder.

Command-line alternative:

```
cd smart1-sales-builder
git init
git add .
git commit -m "Smart 1 Sales Builder v1"
git branch -M main
git remote add origin https://github.com/YOURUSERNAME/smart1-sales-builder.git
git push -u origin main
```

## Deploy on Render (Blueprint)

1. dashboard.render.com → **New → Blueprint**.
2. Connect the **smart1-sales-builder** GitHub repo. Render reads `render.yaml`
   and shows what it will create:
   - a **web service** (`smart1-sales-builder`) — the app itself, with a `/health`
     check and auto-deploy on every commit
   - a **Postgres database** (`smart1-sales-builder-db`) — where quotes, statuses,
     and archived PDFs live, so nothing is lost across deploys
3. It prompts for the one secret it can't invent: **OPENAI_API_KEY**.
   Paste your key to enable the Sales Builder's own AI features, or leave it blank
   — the builder, PDFs, and conversion all work without it.
4. Click **Apply**. First build takes a few minutes; when it's live, open
   the service URL and the dashboard comes up. Updating later is just
   committing to GitHub — Render redeploys automatically.

Plans note: the blueprint uses the `starter` web service and `basic-256mb`
Postgres (Render's smallest paid tiers, a few dollars each). You can
downgrade the web service to `free` in `render.yaml`, but free services
sleep between uses and wake slowly — fine for testing, annoying for
salespeople.

Environment variables (already wired in the blueprint):

- `DATABASE_URL` — from the Render database (quotes survive deploys)
- `IO_API_BASE` — the existing IO app's API (default: insertionordersmart.onrender.com)
- `IO_APP_URL` — what the "Insertion Orders" tab embeds (default: same)
- `OPENAI_API_KEY` *(optional)* — enables "draft a proposal from one
  sentence" on the dashboard and the ✨ AI section rewriter. Everything
  else works without it because the description / audience-estimate /
  landing-review AI calls go to the IO API, which already has its key.

> Note: the "Insertion Orders" tab embeds the IO app in an iframe. If the
> IO app ever refuses to be framed, use the "Open in its own tab" link —
> nothing else depends on the iframe.

## Quote numbers

Quotes are numbered `Q-10200, Q-10201, …` from the Sales Builder's own counter —
the same number family as IOs so they match up later. When a proposal is
converted, the IO's order number comes from the IO app's own
`/api/next-order-number` (the 10200-series Cloudinary counter), exactly
like a hand-built IO, and both numbers are linked in the database.

## The flow (matches the approved process visual)

1. **Build** — guided steps (goals → customer + AI description → landing
   page + AI review → current marketing → exclusions → geo + AI audience
   estimate → audiences → budget → media mix with per-product pricing →
   KPIs → packages) with IO guardrails re-checked on every change
   (search $1,500 floor, short terms, over-split budgets, creative fee).
   Every change autosaves — quotes are never lost.
2. **Edit the document** — the proposal's sections are editable: rename,
   rewrite (or ✨ AI-rewrite), reorder, hide, delete, or add sections.
   Auto sections (reach table, media plan, packages, KPIs) fill from the
   campaign data.
3. **Deliver & save** — branded PDF (`S1M Proposal - Q-##### - Client.pdf`,
   archived in the database with the quote) + formatted Word export.
   Mark it **Sent**.
4. **Decide** — mark **Approved** or **Lost** from the quote drawer. The
   dashboard AI panel nudges on stale quotes and approved-but-unconverted
   proposals.
5. **Convert** — the wizard maps every proposal field to the IO, asks
   ONLY the missing answers (client contact, dates, creative, tracking,
   fees, final landing page), then calls the existing IO API to issue the
   order number, generate the customer + internal PDFs in Cloudinary, and
   (optional checkbox) submit the finished IO to Smart 1 Suite /
   GoHighLevel. The quote is marked **Converted** and linked to the IO #.

## v1 limits worth knowing

- The convert wizard builds a simplified version of the IO's internal
  requirement lists (per-product budgets, geo, audience, landing, KPIs).
  For campaigns that need the full trafficking cards, creative uploads,
  or variable monthly budgets, finish those in the IO builder tab — the
  order number and PDFs are already in place.
- Creative file uploads live in the IO builder (unchanged), not in the
  proposal builder.
- The salesperson name in the top-right is stored per-browser; there are
  no user accounts in v1.
