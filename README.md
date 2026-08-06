# Wedding Ops Board

Shared dashboard for 6 September 2026. Static front end on GitHub Pages,
Supabase for shared state. No login — you type your name once and every
change you make is stamped with it.

## Live backend

| | |
|---|---|
| Project | `wedding-ops` |
| Ref | `ixtynzfktgfwjwlvdaho` |
| URL | https://ixtynzfktgfwjwlvdaho.supabase.co |
| Region | us-east-1 |
| Key in `index.html` | publishable (`sb_publishable_...`) — safe to commit |

Tables: `vendors`, `payments`, `tasks`. Storage bucket: `documents`.
RLS is on, with open read/write policies for the `anon` role. Anyone holding
the publishable key can read and edit — that key ships in `index.html`, so
treat the Pages URL as the only thing standing between the board and a
stranger. Don't put anything on it you'd mind being read.

Attribution instead of authentication: the name you type is stored in
`localStorage` and written to `tasks.done_by_name`, `payments.entered_by_name`
and `vendors.updated_by_name` on every change. It tells you who did what.
It does not stop anyone from doing it.

Realtime is enabled on `vendors`, `payments`, `tasks` — a change on one
browser repaints every other open board.

## Deploy

```bash
git init && git add . && git commit -m "wedding board"
git branch -M main
git remote add origin git@github.com:joshpled/wedding.git
git push -u origin main
```

Repo → Settings → Pages → Source `main` / root. That's the whole deploy —
no Supabase config needed on the front end.

## Sharing it

Send the URL. They type a name and they're in.

If you ever want it locked down, the smallest change that helps is a shared
passphrase checked in the browser before `load()` runs. Real security means
putting auth back.

## Documents

Uploads go to a public Supabase Storage bucket, `documents`. 25 MB per file,
1 GB free across the project — hundreds of contracts' worth.

Why Storage and not the alternatives:

- **In the repo** — no browser upload. Every file needs a `git push`, and
  Barbara isn't doing that from her phone.
- **Google Drive** — you already have the folder, but reading it from a static
  page needs OAuth, which is the login you deliberately removed.
- **Supabase Storage** — same project, same key, drag-and-drop from any
  device, public URLs that open straight in a browser tab.

Two consequences worth knowing:

1. **Public means public.** Anyone with a file's URL can open it. Contracts and
   invoices are fine. **Do not upload the filled-in Snyder CC authorization
   form, or anything else with a card number or a signature on it.**
2. **No deletes**, same as payments. Upload a corrected version instead.
   Filenames are prefixed with a timestamp so nothing overwrites anything.

Keep Drive as the archive; this bucket is for what people need to open from
their phone.

## Schema notes

- `tasks.seed_key` marks the 26 seeded tasks. A database trigger blocks
  deleting them, so nobody can wipe the original checklist by accident.
  Tasks added later delete normally.
- `payments` is the source of truth for "paid". Vendor totals never store a
  paid figure — it's summed from payments, so the math can't drift.
- Payments cannot be deleted. There is no delete policy on the table and a
  `before delete` trigger raises regardless, so neither the app nor a stray
  API call can drop one. Wrong entries get corrected in place; an entry that
  never happened gets set to 0. The UI offers **Correct**, not a delete.
- `vendors.total` is null for anything unquoted. Those render as
  "No number yet" rather than zero, so they don't quietly disappear from
  the budget.

## Data provenance

| Vendor | Source |
|---|---|
| Patrick Properties | BEO / Menu Confirmation E30749, 29 Jul 2026 |
| Invitation Only | Service Invoice 00498, 25 Nov 2025 |
| Richard Bell Photography | Invoice 17H1742217, 10 Jun 2026 |
| Dream Day Charleston | Payment confirmation email, 2 Jul 2026 |
| Palmetto Strings | Contract signed 27 Jul 2026, total confirmed verbally |
