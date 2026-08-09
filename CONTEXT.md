# Donruss Optic Uptown/Downtown Ledger — Project Context

## What this is
A single-file HTML dashboard (`index.html`) that tracks profit potential on
Panini Donruss Optic **Uptown** and **Downtown** insert cards (2024–2025,
football) by comparing raw (ungraded) market price against PSA 10 market
price. Hosted on GitHub Pages, no backend — all data lives inline in the
HTML file's `SEED_CARDS` array and edits persist via the browser's
localStorage.

## Data model
Each card object has:
- `id` — unique string, e.g. `"up-12"` (Uptown) or `"dt-7"` (Downtown)
- `set` — `"Uptown"` or `"Downtown"`
- `player`, `year`, `parallel` (e.g. `"Base"`, `"Gold /10"`, `"White Pandora /25"`,
  `"Black Pandora /25"`), `cardNumber`
- `raw`, `psa9`, `psa10` — prices in USD, or `null` if no reliable sale exists
- `gemRate`, `avgGrade` — PSA population data, left `null` until manually
  logged (there's no automated way to pull PSA's population report)
- `image` — optional URL, blank by default (never bulk-populate this; see
  "Known limitations" below)

Derived at render time (not stored): `profit = psa10 - raw - fee`,
`profitPct`, `cardType` (`"Base"` if parallel is empty/"Base", else `"Variant"`).

## Data source & methodology
All prices sourced from **SportsCardsPro** (sportscardspro.com), which
aggregates completed eBay/marketplace sales through a proprietary,
recency-weighted algorithm (blends most-recent sale, median, average; filters
outliers). This is a **snapshot pulled manually**, not a live feed — there is
no API integration. To refresh prices, pull the "Most Expensive" sort view
for the relevant year/set page on SportsCardsPro, e.g.:
`sportscardspro.com/console/football-cards-2025-panini-donruss-optic-uptown?sort=highest-price`

Uptown's premium parallel is **White Pandora /25**; Downtown's is
**Black Pandora /25**. Most seed entries are paired: each player appears
once as Base and once as the premium parallel where both had priced sales,
so raw-vs-premium profit is directly comparable.

## Known limitations (don't "fix" these by guessing)
- Several low-population parallels (some Golds, /1 Vinyls) have no PSA 10 or
  PSA 9 price because too few graded copies have sold — left blank
  intentionally, not a bug.
- `gemRate` / `avgGrade` require manually checking PSA's population report
  (not fetchable automatically) — all currently `null`.
- `image` fields are intentionally blank on seed data — thumbnails were not
  bulk-added because verifying the correct photo per card/parallel from
  scraped data risked mismatches. Only add an image URL when a specific one
  has been manually verified.

## UI / feature summary
- Vanilla HTML/CSS/JS, single file, no build step, no dependencies beyond
  Google Fonts (Fraunces, JetBrains Mono, Inter).
- Dark theme; gold accent (#C9A227) for Uptown, crimson (#C74B4B) for
  Downtown, used in badges.
- Sortable columns (click header), filters for Set (All/Uptown/Downtown) and
  Type (All/Base/Variant), editable grading-fee input (default $25).
- Add/Edit modal for individual cards; "Reset to seed data" restores
  `SEED_CARDS` and wipes localStorage edits.
- Data persists in the visiting browser's localStorage — this is
  per-browser, not synced across devices, and not visible to me (Claude in
  chat) unless the user tells me what they changed.

## Ongoing workflow
1. User asks Claude (in claude.ai chat) to pull fresh prices for specific
   cards/sets from SportsCardsPro.
2. Claude (chat) returns updated values or a fully regenerated `index.html`.
3. User hands that to Claude Code in this repo with an instruction like
   "update index.html with these values, commit, and push."
4. GitHub Pages auto-redeploys from the `main` branch, root folder.

Live site: https://mrabasca03-hub.github.io/Donruss-Uptown-Downtown/
