# Tab Board — Project Context

> **Retired 2026-10-09.** The sticky notes moved into Deal Kanban as its NOTES
> view (the deal-kanban repo). This app is now a one-time hand-over page: when
> anyone opens it, it posts each person's board from Monday app storage to
> Deal Kanban's /api/notes/import, then says where the notes went. The full old
> app is in git history (commit 9aa3cc2 and before). Everything below describes
> the old app.

## What this is

A sticky-note board view that sits on Provident LED's Monday **CRM (Deals)**
board. Each note is a pinned deal; salesmen (Ari, Chuny, Shragie) drag browser
tabs onto it instead of leaving dozens of tabs open. `README.md` has the full
explanation of what is stored where; read it before changing storage.

- Repo: `github.com/ysteinberg1/tab-board`
- Hosting: Vercel, static project (no build step, output directory `.`),
  auto-deploys on push to `main`
- The whole app is one file: `index.html`. The PNGs are logos.
- Runs inside Monday as a **Board View** app feature, via the Monday SDK.

## Where data lives

- **On the Monday item** (shared by everyone on the deal):
  Salesman Notes `long_text_mm6khzqt`, Tab Board Links `long_text_mm6k5rkn` (JSON).
- **In Monday app storage, per user** (`tabboard:<userId>`): which deals are
  pinned and their order, folded and set-aside state, `touchedAt`, Loose Ends.

Column ids, the sales team list, the sticky-note group and all thresholds are
in the `CFG` block near the top of `index.html`. Change them there, not inline.
For anything else about the CRM board, use the `monday-provident` skill.

## Things that will bite

- **Monday API version:** this app calls `monday.api()` from inside the board
  view, so Monday applies its own **older** API version. Queries written for the
  current schema (e.g. `items_page`) can fail here, and `me { is_admin }` works
  here even though the current schema dropped it. See the "Why the embedded view
  does not use the SDK for data" section in `..\deal-kanban\CLAUDE.md`. Don't
  copy tab-board queries into the Next.js apps, or the reverse, without checking.
- **Don't load the whole board.** Pinned deals load by id; the search index is
  capped by `CFG.MAX_ITEMS`. Pulling 800+ items on every open was slow.
- **Never silently drop a pinned deal** because it isn't loaded yet. Past bugs
  here lost pins (past the 25th, and for deals not yet fetched).
- **Only real edits count as a "touch"**: adding, renaming or removing a link,
  or editing the note. Reordering, folding and setting aside don't reset the
  day badge.
- The links column is limited by Monday's long-text size; a failed save must
  show "Too long to save", never drop links quietly.

## Related projects

- **deal-kanban** copied tab-board's look and its `muted()` colour transform.
- **field-sales-map**: the map pin on a card opens the deal there
  (`SALES_MAP` in `index.html`).

## Making a change

Edit `index.html`, commit, push to `main`. Vercel republishes in about a
minute; hard-refresh the Monday view (Ctrl+Shift+R) to see it.
