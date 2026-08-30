# Reading Passport

A reading tracker and rewards app built for my daughter Hannah (age 7). Two parents
(me and her mother, in separate households) both use it; Hannah logs her reading with
help from whichever parent she's with.

## What it is

`index.html` — one self-contained file. No build step, no dependencies, no framework.
Plain CSS and vanilla JS. It's deployed to GitHub Pages and both parents add it to
their phone home screens.

Keep it a single file unless there's a strong reason not to. If you split it up, it
still has to deploy to GitHub Pages as static files with no build step.

## Two views

**Kid view** (default) — points bank in a sticky header, monthly streak panel, books
currently being read, the shelf, finished books, recent reading log, prize shelf.

**Parent dashboard** (link at the bottom of the kid view) — tabs for Library, Add a
Book, Reading Log, Streaks, Prizes & Points, Stats.

## Points system

A book's **base value** = `ceil(pages / pagesPerPoint) + (readingLevel 1-5 × difficultyBonus)`.
Defaults: `pagesPerPoint: 5`, `difficultyBonus: 8`, `completionBonus: 10`. All three are
editable in the parent dashboard.

Points accrue **progressively**: logging pages earns that fraction of the book's base
value immediately. Finishing pays out the remainder plus the completion bonus, so a
finished book always totals `base + bonus` regardless of how it was logged.

**The balance is derived, never accumulated.** `balance()` recomputes from the log,
completions, streak awards, and redemptions every render. This is deliberate — when a
parent corrects a log entry ("she said 50 pages, meant 15"), points and streaks
self-correct. Don't refactor this into an incrementing counter.

## Streaks

Reset every calendar month. `curStreak()` counts consecutive days with log entries,
stopping at the 1st. `bestStreak()` finds the longest run in the month.

Milestones at 5, 10, 20, 30 days:
- 5 days → 10 bonus points (awarded automatically)
- 10 days → $5 cash (parent hands out)
- 20 days → Special Event (parent hands out)
- 30 days → Special Prize (parent hands out)

Milestones lock against **best** streak, not current — a missed day never takes back
an already-earned prize. This is intentional; don't "fix" it.

Real-world prizes appear as a banner in the kid view and in the parent Streaks tab
with a "Mark as given" button.

## Storage and sync

Two modes, controlled by the CONFIG block at the top of the script:

- Both `SUPABASE_URL` and `SUPABASE_KEY` blank → localStorage only, this device.
- Both filled in → Supabase sync. Whole state stored as one JSONB blob in a
  `reading_app` table, row id `hannah`. Pushes debounced 700ms, pulls every 15s.
  Last-write-wins; acceptable since two parents rarely edit simultaneously.

The anon key is public in the repo. Fine for this — no sensitive data — but don't
reuse that Supabase project for anything else.

## Book covers

No live book API yet. Covers are generated procedurally from genre color + title
typography + a watermark emoji. An optional `coverUrl` per book overrides this and
falls back to the generated cover if the image fails to load.

`CATALOG` is a hardcoded array of ~25 common kids' books with real page counts,
grade bands, and fun facts, used for autocomplete when adding a book.

## Things I'd like to do next

1. **Real ISBN lookup.** Biggest win. Type or scan an ISBN, get title, author, page
   count, and cover art automatically. Open Library's API is free and needs no key.
   Would replace most manual entry.
2. **Barcode scanning** from a phone camera so Hannah can scan books off the shelf.
3. Reading level data is currently manual (K-1 through 5th+). Lexile or AR level
   lookup would be better if there's a free source.

## Conventions

- Kid-facing copy is warm and direct, second person, no baby talk. She's 7 and reads
  well. "You read today — streak is safe!" not "Great job little reader!"
- Parent-facing copy is plain and functional.
- Palette: ink navy `#1F2A44`, parchment `#FBF3E1`, marigold `#F2A93B`, coral
  `#EF6F6C`, sage `#5FA98A`, plum `#7C5CBF`. Fonts: Baloo 2 (display), Nunito (body),
  Space Mono (numbers). Don't drift from these.
- Escape all user-entered strings with `esc()` before putting them in HTML.
- Must work on a phone. Test at narrow widths.

## Deploy

Push to `main`. GitHub Pages serves it from the repo root. Live within a couple of
minutes.
