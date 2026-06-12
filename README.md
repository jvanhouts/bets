# bets

A simple static page that shows a table of bets grouped by match.

## Data model

- **`matches`** — an event you bet on (e.g. "Portugal vs Nigeria"), with `match_date` and a
  nullable `due_time`.
- **`bets`** — individual bets attached to a match (`odds`, `amount`, `w_outcome`, `win_lose`,
  `result`).
- **`sources`** — bookmakers, a **shared** entity: one row per name (`name` is unique), reused
  across bets.
- **`bet_sources`** — many-to-many join linking bets to sources. The link is owned by the bet,
  so adding "bet365" to five bets creates five links to the single shared `bet365` source row.

The table shows one **flat row per match**: name and date come from the match, while odds is
the average of its bets and amount / W outcome / result are sums. The Source column shows the
first source as a badge, with a `+N` count when the match's bets carry more than one source.
When signed in, click a row to expand and edit the individual bets (and their sources) behind it;
the source input offers existing source names so you reuse rather than duplicate them.

## Data source

Data comes from Supabase when configured, with a built-in fallback dataset otherwise:

1. Copy `.env.example` to `.env` and fill in `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY`.
2. Fresh setup: run `supabase/schema.sql` then `supabase/seed.sql` in the Supabase SQL editor.

Upgrading an existing database instead of a fresh setup — run, in order, whichever steps you
haven't applied yet:

- `supabase/migration.sql` — split a flat `bets` table into `matches` + `bets` (+ a per-bet
  `sources` table). If `matches` already exists, use `supabase/migration-sources.sql` to just
  add the per-bet sources table.
- `supabase/migration-sources-shared.sql` — reshape the per-bet `sources` table into a shared
  `sources` entity + `bet_sources` join table (preserves links).

Types are validated with zod (`src/lib/schema.ts`). If Supabase is unreachable or returns data
that doesn't match the schema, the app falls back to the static dataset in `src/lib/bets.ts`.

## Editing

Cells are click-to-edit, but only after signing in: writes are restricted to authenticated
users by row-level security, so visitors get a read-only table. Create a user in the Supabase
dashboard (Authentication → Users → Add user), then use "Sign in to edit" on the page.

- Edit a match's **name** and **date** inline on its row; edit its **due time** in the expanded panel.
- Expand a row to edit, add, or delete the individual **bets** (the aggregated columns aren't
  directly editable since they're sums/averages).
- Edits save per field on Enter/blur; Escape cancels.

## Commands

```sh
bun run dev      # start dev server
bun run build    # type-check and build to dist/
bun run preview  # preview the production build
```
