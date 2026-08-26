# Tab Board — Monday board view

A sticky-note board that sits on your CRM board. Each note is a pinned deal;
salesmen drag browser tabs onto it instead of leaving forty tabs open.

## What lives where

**On the Monday item** (survives the app, visible in Monday, searchable, exportable):
- `long_text_mm6khzqt` — Salesman Notes, plain text
- `long_text_mm6k5rkn` — Tab Board Links, JSON:

```json
{"v":1,"links":[
  {"u":"https://www.linkedin.com/in/mike-rosen-ops","d":"2026-08-24"},
  {"u":"https://app.companycam.com/projects/8841","t":"Before photos — main bay","d":"2026-08-22"}
]}
```

`t` only appears when someone renamed the link; otherwise the name is derived
from the URL at display time. Because the links live on the item, they belong to
the deal — every salesman on that deal sees the same set.

**In app storage, per user** (`monday.storage.instance`, key `tabboard:<userId>`):
- which deals are pinned, and their order
- folded / set-aside state
- `touchedAt` per pinned deal — drives the day badge
- the Loose Ends links and its note

So two salesmen pinning the same deal share the links but keep their own board.

## Deploying

1. Push this folder to GitHub, import it to Vercel as a static project
   (no build command, output directory `.`). Note the deployment URL.
2. In Monday: **Developers → Create app → Add feature → Board View**.
3. Set the Board View URL to your Vercel URL.
4. Under **Permissions / OAuth scopes**, enable:
   `boards:read`, `boards:write`, `users:read`, `me:read`, and app storage.
5. Install the app on your account, then add the view to the CRM board:
   **+ (tab bar) → Apps → Tab Board**.

## Configuration

Everything adjustable is in the `CFG` block at the top of `index.html`:

| Key | Meaning |
|---|---|
| `NOTES_COL` / `LINKS_COL` | the two long-text column ids |
| `STATUS_COL` | leave `null` to auto-detect the first status column |
| `CLOSED_LABELS` | labels that trigger the clearing ribbon |
| `CLEAR_AFTER` | days a closed deal stays before falling off (3) |
| `WARM_AT` / `HOT_AT` | days untouched before the badge goes amber / red (21 / 42) |
| `SHOW_LINKS` | links shown before the "more" triangle (3) |
| `MAX_ITEMS` | deals pulled from the board (500) |

## Notes for whoever picks this up next

- **Adding a tab:** drag the icon from the *address bar*, not the tab itself —
  Chrome keeps tab drags for its own reordering and never offers them to the page.
  The Paste button is the fallback.
- **Closing the tab** after a drop is not possible from inside Monday. That needs
  a browser extension.
- **Page titles** can't be fetched from a URL (CORS), so names are derived from
  the URL and renamed by hand with the pencil.
- **The board owner picker** appears only for board owners and account admins.
  Viewing someone else's board is read only.
- **Reordering, folding and setting aside are deliberately not "touches"** — only
  adding, renaming or removing a link, or editing the note, resets the day badge.
- The links column is capped by Monday's long-text limit. Ten or so links per deal
  is comfortable; if someone hits the ceiling the save fails and the rail shows
  "Too long to save" rather than silently dropping data.
