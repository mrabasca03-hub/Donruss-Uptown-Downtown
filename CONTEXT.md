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

## Price history / change tracking
Each card has a `history` array: `[{ date: "YYYY-MM-DD", raw, psa9, psa10 }, ...]`, sorted
ascending by date. `ensureHistorySeed()` runs on every load (and after "Reset to seed data") and
stamps a baseline snapshot dated today onto any card that doesn't have one yet — this is what
makes tracking start immediately for every card, not just ones a user happens to open and save.
Saving a card (add or edit) separately upserts today's date with the new values, overwriting
same-day entries rather than duplicating them.

The table has a "Δ vs" period selector (7 / 30 / 60 / 365 / 730 days, persisted to localStorage
like the fee) and three dedicated, sortable columns — Δ Raw / Δ PSA 9 / Δ PSA 10 — whose headers
show the active period (e.g. "Δ Raw (30D)"). For the selected period, `valueAtOrBefore()` finds
the most recent history entry dated on or before `today - period` and compares it to the current
price; if no entry is that old yet, the cell shows "—". Critically, this lookup does **not**
require an exact-day match — it just wants the closest known point at or before the cutoff — so
sparse, irregular history (e.g. only month-end snapshots) works fine and is the practical way to
backfill real change data instead of needing a price logged for every single day.

Because there's no live feed, meaningful non-"—" change data depends on backfilling past prices,
not just waiting for real time to pass from the auto-seeded baseline. The Edit-card modal has a
"Price history" section (`+ Add price point`) where dated raw/psa9/psa10 entries can be added or
removed manually — e.g. pulled from SportsCardsPro's own historical price chart for that card.
Sourcing/verifying real historical prices per card is manual effort (same "don't guess" caution
as images below), so backfilling is best done in trial batches rather than all cards at once.

Automated fetching of SportsCardsPro (or PSA/Sports Card Investor/Card Ladder) is not viable from
Claude Code in this environment — WebFetch gets 403s from all of them, the sandboxed Browser pane
is policy-blocked on these domains, and eBay search results only sparsely/inconsistently surface
real sale prices. The user's own browser is unaffected by these blocks and is the only reliable
way to pull real prices; Claude formats whatever values the user hands back.

Column order in the table is: Raw, PSA 9, PSA 10, Profit $, Profit %, then the three Δ (change)
columns, then Gem Rate / Avg Grade — profit comes before change so the "is this worth attention"
metric reads left-to-right before the "why/when" context.

Each card's name has a small "↗" link to its best-effort SportsCardsPro page, built by
`sportsCardsProUrl()` from a URL pattern verified against ~30 real card pages (year/set slug,
parallel slug, player-name slug, card number). It is **not guaranteed correct** for every card —
slugs for uncommon parallels or name formats (suffixes, initials) are inferred, not individually
verified, so some links may 404. That's an acceptable, low-stakes failure mode (worst case: use
the site's own search) — unlike price data, a wrong link isn't a data-integrity problem.

## Thumbnails
Every card renders a generated placeholder thumbnail (inline SVG, no external
requests): player initials + a short parallel tag (BASE/GOLD/W.PAN/B.PAN/…),
colored gold for Uptown and crimson for Downtown to match the existing badge
palette. Generated at render time from `player`/`set`/`parallel` — nothing is
stored, so it applies automatically to every card, including new ones. If a
card has a manually verified `image` URL, that photo overlays on top of the
placeholder (`onerror` falls back to removing the broken image and exposing
the placeholder again). This exists specifically so no one has to source or
verify 100+ individual card photos to get thumbnails.

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
- `image` fields are intentionally blank on seed data — real photo thumbnails
  were not bulk-added because verifying the correct photo per card/parallel
  from scraped data risked mismatches. Only add an image URL when a specific
  one has been manually verified; the generated placeholder (see below)
  covers the rest.

## UI / feature summary
- Vanilla HTML/CSS/JS, single file, no build step, no dependencies beyond
  Google Fonts (Fraunces, JetBrains Mono, Inter).
- Dark navy theme (`--bg` #0A0E1A); white accent for Uptown, blue (#4C7EFF)
  accent for Downtown, used in badges and generated thumbnails. Green/red
  (profit/loss) are unrelated semantic colors and unchanged by this palette.
- Sortable columns (click header), filters for Set (All/Uptown/Downtown) and
  Type (All/Base/Variant), editable grading-fee input (default $25).
- Add/Edit modal for individual cards; "Reset to seed data" restores
  `SEED_CARDS` and wipes localStorage edits.
- "Δ vs" period selector (7D/30D/60D/1Y/2Y) shows each price's % change under
  Raw/PSA 9/PSA 10; per-card price history is editable in the Add/Edit modal
  (see "Price history / change tracking" above).
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
