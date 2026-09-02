# SQL Query Notebook

**Organize, Search & Manage Your SQL Queries**

A shared home for your team's SQL, built entirely on free Google
infrastructure — Sheets as the database, Apps Script as the backend, Drive for
files, Gmail for notifications. No server, no hosting bill, no external database.

**The whole application is two files.**

```
Code.gs        the entire backend      201 KB   5,650 lines
Index.html     the entire front end    232 KB   5,073 lines
```

Paste both into the Apps Script editor, run `setupSystem()`, deploy. See
**[SETUP.md](SETUP.md)** — 12 steps, about 15 minutes.

> This is a query *notebook*, not an execution engine. It stores, versions,
> searches and shares SQL. It never connects to a database and never runs your
> SQL. That is deliberate — see [Why no execution](#why-no-execution).

---

## What you get

| | |
|---|---|
| **Store** | Permanent ID per query (`SQL-000001`), category, database label, tags, status, description |
| **Find** | Full-text search across names, descriptions, tags, authors **and the SQL body itself**. `Ctrl+K` from anywhere |
| **Filter** | Category, database, status, tag, author, date ranges — combined, applied server side |
| **Version** | Every edit is an immutable version. View, compare with a line diff, restore |
| **Share** | VIEW or EDIT to named colleagues, with optional expiry and an email notice |
| **Audit** | Append-only log of every login, view, copy, edit, delete, permission change and denied attempt |
| **Control** | 4 built-in roles plus custom ones, across a 21-permission matrix an admin edits live |
| **Anywhere** | One responsive interface for desktop, tablet, Android and iPhone. Light and dark |

**21 screens:** Login · Dashboard · All Queries · My Queries · Favourites ·
Recent · Shared With Me · Trash · Query Details · Create · Edit · Version
History · Version Compare · Categories · Databases · Users · Roles &
Permissions · Activity Logs · Settings · Backup · 404.

---

## Files in this folder

| File | What it is |
|---|---|
| **`Code.gs`** | The whole backend. Paste into Apps Script. |
| **`Index.html`** | The whole front end. Paste into Apps Script as `Index`. |
| **`appsscript.json`** | The manifest — scopes and timezone. Apps Script shows this itself; you replace its contents. |
| **`SETUP.md`** | 12-step install guide. Start here. |
| **`preview.html`** | Open in any browser to click through the interface on demo data. Nothing is saved. Not needed to deploy. |
| `docs/` | The installation manual as an interactive web page, a Word document and a PDF. |
| `README.md` | This file. |

That is the entire install. Three files pasted, one guide, one optional preview.

### Finding your way around `Code.gs`

It opens with a table of contents and each of the 16 modules has a banner.
**Ctrl+F** for `SECTION 07` to jump.

```
SECTION 01  Config          schema, permissions and defaults
SECTION 02  Utils           Utils, Cache, SheetDB, Settings
SECTION 03  ActivityLogs    audit trail and usage tracking
SECTION 04  Auth            identity, sessions, permission gate
SECTION 05  Queries         CRUD, search, filter, paging, dashboard
SECTION 06  Versions        history, diff, restore
SECTION 07  Categories      category master data
SECTION 08  Databases       database labels
SECTION 09  Users           users, roles, permission matrix
SECTION 10  Favorites       per-user favourites
SECTION 11  Sharing         VIEW / EDIT grants
SECTION 12  DriveService    .sql mirrors and attachments
SECTION 13  Backup          CSV / ZIP export and backups
SECTION 14  Notifications   Gmail notifications
SECTION 15  Setup           setupSystem(), seedDemoData(), maintenance
SECTION 16  Code            doGet() and the 57 API endpoints
```

### Finding your way around `Index.html`

```
<style>    the design system — all theming lives in the ~40 CSS variables
           in :root and [data-theme="dark"] at the very top
<body>     one mount point; every screen is drawn by JavaScript
<script>   icons → core (Fmt/Sql/Theme/API/State/Toast/Modal/Drawer)
           → views-queries → views-detail → views-admin → app
```

To restyle, change those variables. Nothing hard-codes a colour.

---

## How it works

```
   Browser                    Apps Script (runs as YOU)         Google
   ───────                    ─────────────────────────         ──────
   Index.html                 Code.gs                            Sheets   14 tabs
   google.script.run(fn) ──▶  doGet()                            Drive    files
                              guard_()  ◀── every call            Gmail    notices
                                │
                                ├─ who are you?   Session.getActiveUser()
                                ├─ what is your role?  Users sheet
                                └─ may you do this?    Permissions sheet
```

**The rule that governs everything:** the browser never decides anything. All
57 API endpoints pass through `guard_()`, which resolves identity server side
and checks permissions. Any `role`, `userId`, `email` or `permissions` field
sent from the browser is ignored.

Hidden buttons are a courtesy. The server check is the control — every hidden
button has a matching server-side refusal.

---

## The database

One spreadsheet, 14 tabs, created for you by `setupSystem()`.

| Tab | Holds |
|---|---|
| **Queries** | The queries themselves |
| **QueryVersions** | Immutable history, one row per version ever |
| **Categories** / **Databases** | Master data. Databases store *labels*, never credentials |
| **Users** / **Roles** / **Permissions** | Who, what role, what that role may do |
| **UserFavorites** | Per-user stars — never a shared flag |
| **QueryShares** | Sharing grants with optional expiry |
| **QueryUsage** / **ActivityLogs** | Usage tracking and the audit trail |
| **Sessions** / **Settings** / **Attachments** | Live sessions, config, Drive files |

IDs never repeat. `SQL-000042` stays retired even after permanent deletion, so
references in tickets and documentation stay meaningful.

---

## Roles

| | Admin | Manager | Editor | Viewer |
|---|:--:|:--:|:--:|:--:|
| View, search, copy, download | ✓ | ✓ | ✓ | ✓ |
| View version history | ✓ | ✓ | ✓ | ✓ |
| Create queries | ✓ | ✓ | ✓ | — |
| Edit own | ✓ | ✓ | ✓ | — |
| Edit anyone's | ✓ | ✓ | — | — |
| Delete to Trash | ✓ | ✓ | — | — |
| Restore from Trash | ✓ | ✓ | — | — |
| Delete permanently | ✓ | — | — | — |
| Restore a version | ✓ | ✓ | — | — |
| Share | ✓ | ✓ | ✓ | — |
| Export | ✓ | ✓ | — | — |
| Users, roles, settings, backups | ✓ | — | — | — |

Admins edit any cell of this live from **Roles & Permissions**. Two are locked:
the Admin role can never lose `MANAGE_ROLES` or `MANAGE_USERS`, and the last
active administrator can never be demoted or deactivated.

---

## Versions

`QueryVersions` keeps one row for every version that has ever existed,
including the live one. Nothing is overwritten.

```
SQL-000001
  v1.0   25-Aug   Haroon   Initial version
  v1.1   29-Aug   Muneeb   Added KPI fields
  v2.0   02-Sep   Haroon   Optimised joins          ← current
```

SQL changed → major bump (`1.4 → 2.0`). Metadata only → minor (`1.4 → 1.5`).

**Restoring is additive.** Restore v1.0 while v3.0 is live and you get a new
**v4.0** carrying v1.0's SQL. v3.0 stays in the timeline. An undo is itself
undoable.

---

## Security

- Server-side authorisation on every call, through one choke point
- Identity resolved by Google, never taken from the browser
- Object-level checks — "may you edit *this* query" is separate from "does your
  role have EDIT_QUERY"
- Every dynamic value HTML-escaped before rendering; the SQL highlighter escapes
  each token individually
- Spreadsheet formula injection neutralised on write and on CSV export
- 30-minute sliding sessions with a warning dialog and instant revocation
- Rate limiting, account lockout, `LockService` on ID allocation and writes
- Upload extension allow-list and size cap checked before Drive is touched
- No credentials anywhere — the app has none to store

**Honest limits.** Application passwords use salted SHA-256 with 12,000 rounds;
Apps Script has no bcrypt. Google SSO is the intended path. And anyone with edit
access to the underlying spreadsheet bypasses the entire permission model —
share the web app URL, never the Sheet.

---

## Limits

Built for roughly **100–500 users** and low tens of thousands of queries.
One `getValues()` per sheet per execution, cached lookups, server-side paging.

| | |
|---|---|
| Apps Script execution | 6 min/call (30 on Workspace) |
| Gmail quota | 100/day consumer, 1,500/day Workspace |
| Practical query count | ~20,000 before lists feel slow |
| Attachment | 10 MB |
| ZIP export | 300 queries |

---

## Why no execution

Running arbitrary SQL against a production warehouse from a shared web app needs
stored database credentials, a sandbox, result-size limits, per-user database
identities and query timeouts. Getting any of that wrong turns a convenience
tool into a data-exfiltration path.

The notebook solves the actual daily problem — *where is that query and which
version is correct* — without taking that risk. People copy the SQL into the
tool they already have permission to run it in.

If you do want a connector later, the permission system is built for it: add
the module, keep credentials in Script Properties, gate it behind a new
`EXECUTE_QUERY` permission.

---

## Putting it on GitHub

Apps Script keeps no browsable history and no way to diff two versions. GitHub
gives you both, plus somewhere the code survives if the Google account is lost.

**These files are safe to publish.** No spreadsheet IDs, no Drive folder IDs, no
script ID, no passwords, no API keys, no company data. Every real ID lives in
Script Properties inside your Apps Script project and never leaves Google.

### The easy way — no command line

1. [github.com/new](https://github.com/new) → name it `sql-query-notebook`
2. Choose **Private** (or Public — the code has no secrets either way)
3. Leave *Add a README*, *.gitignore* and *license* **unticked** — this folder
   already has them, and ticking them creates a conflict
4. **Create repository** → **uploading an existing file**
5. Drag in everything from this folder and **Commit changes**

### With git

```bash
git init -b main
git add .
git commit -m "SQL Query Notebook v1.0.0"
git remote add origin https://github.com/YOUR-USERNAME/sql-query-notebook.git
git push -u origin main
```

### Keeping it current

GitHub and Apps Script do not sync on their own. After changing code in the Apps
Script editor, copy the file back here and commit it — or use
[clasp](https://github.com/google/clasp) to skip the copy-paste entirely:

```bash
npm install -g @google/clasp
clasp login
# create .clasp.json with your Script ID (Project Settings in the editor)
clasp push     # local  -> Apps Script
clasp pull     # Apps Script -> local
```

**Never commit `.clasp.json`** — it holds your Apps Script project ID. The
included `.gitignore` already excludes it, along with `.env`, key files and
credential JSON.

---

## License

MIT. The demo SQL is illustrative and assumes a courier/logistics schema that
will not match yours; every demo query is tagged `demo`.
