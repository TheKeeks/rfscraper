# RF & Microwave — Coverage Map

An interactive market-mapping site for the everythingRF company directory
(~2,598 companies), with a pipeline that refreshes the data weekly.

- **The site** is static HTML/JS, hosted on **GitHub Pages** — no server, free.
- **The data** is produced by a scraper that runs in **GitHub Actions** and is
  written to `companies.json`; the site loads that file.

Pages can't *run* the scraper, but it serves the data file Actions produces.

## Files in this repo

```
rfscraper/
├─ index.html              # the interactive site (served by GitHub Pages)
├─ companies.json          # data the site loads — illustrative SAMPLE, replaced by the weekly run
├─ everythingrf_scraper.py # pulls the everythingRF directory
└─ .github/workflows/refresh.yml   # weekly: scrape -> rebuild companies.json -> commit
```

## Deploy (summary — see the chat walkthrough for click-by-click)

1. Create a repo named `rfscraper` — **Private** recommended (see the gate note).
2. Add the four files above (the site files go at the repo root; the workflow at
   `.github/workflows/refresh.yml`).
3. **Settings → Pages → Deploy from a branch → `main` → `/ (root)`.** Your site is
   the Pages URL it shows.
4. **Settings → Actions → General → Workflow permissions → "Read and write
   permissions"** so the refresh job can commit the data file.
5. **Actions → Refresh coverage data → Run workflow** to pull real data now; after
   that it runs weekly on its own. Pages redeploys on each commit.

## Data refresh

`refresh.yml` runs weekly (Mondays 06:00 UTC). A full scrape is ~2,600 page loads
(~45 min). To make it daily, swap the cron line (commented in the file).

## Password gate — read this

The site asks for a password before loading data. It's stored only as a
**SHA-256 hash**, so the literal string isn't in the source.

**This is obfuscation, not security.** Static hosting can't check a password
server-side, so the gate is just browser JavaScript:

- The page source and `companies.json` are still directly downloadable by anyone who
  looks. On a **public** repo the source is also openly browsable on github.com — so
  the gate is meaningful **only if the repo is Private.** Keep it private.
- Serving Pages from a private repo generally requires a paid GitHub tier (Pro or an
  Enterprise org) — check your plan. On a free personal account, Pages usually
  requires the repo to be public.
- For *actual* access control it has to come from the host: private Pages
  (Enterprise), an edge-auth layer (Cloudflare Access, Netlify/Vercel password
  protection), or an SSO-protected internal site.

To change the password, replace `PW_HASH` in `index.html` with the hash of a new one:

```bash
python3 -c "import hashlib; print(hashlib.sha256(b'YOUR_NEW_PASSWORD').hexdigest())"
```

The gate needs a secure context (https), which GitHub Pages provides. On a local
`file://` preview it auto-opens (only the bundled sample is present locally).

## Brief for the model iterating this (Claude Code / Claude Fable 5)

You have creative latitude over visual design, layout, extra views, and
visualizations. Preserve the three analytical lenses — **Specialist ↔ Platform**
(coverage breadth), **capability adjacency** (product-category overlap), and
**fragmentation** (players per category) — and keep the password gate, the CSV
export, and the sample-data banner.

**Data integrity is non-negotiable:**

1. Use **only** the values present in `companies.json`. Do not invent companies,
   categories, certifications, counts, locations, or any field value.
2. The bundled `companies.json` is **illustrative sample data with fictional company
   names.** It must stay clearly labeled as sample in the UI until the live scrape
   replaces it. Never present sample data as if it were real companies or figures.
3. **No fabricated numbers anywhere.** Every figure shown must be *computed from*
   `companies.json` (counts, % overlaps, player tallies, breadth bands). Do not add
   invented market sizes, revenues, growth rates, valuations, dates, or any metric the
   data does not contain.
4. Missing or empty fields render as "—" or are omitted — never guessed or filled in.
5. Derived metrics are fine and encouraged (breadth band, Jaccard overlap,
   fragmentation read) because they are computed from the data; keep the method
   transparent.
6. If you add a chart or statistic, it must trace back to a field in the data. When in
   doubt, show less.

**Record schema** (each entry in `companies.json`):
`id, name, country, city, state, website, categories[], num_categories,
total_products, certifications[], end_markets[], profile_url`
(`end_markets` is empty in the live pipeline today — reserved for a later enrichment
pass; don't populate it with guesses.)
