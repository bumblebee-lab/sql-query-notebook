# Setup — the two file install

Everything you need is **two files**. Allow about 15 minutes.

```
Code.gs        the whole backend      (5,650 lines, 16 modules merged)
Index.html     the whole front end    (5,073 lines, CSS + JS merged)
```

A third file, `appsscript.json`, is the manifest. Apps Script creates it for
you — you only edit it, never create it.

You need a Google account. A work (Workspace) account is strongly recommended.

---

## STEP 1 — Create the spreadsheet

1. Go to [sheets.google.com](https://sheets.google.com) → **Blank spreadsheet**.
2. Click the title (top left) and rename it exactly:

   ```
   SQL Query Notebook DB
   ```

3. Leave the empty `Sheet1` alone. Setup deletes it.

Do **not** create any tabs. Setup builds all 14 with the right headers.

---

## STEP 2 — Open the script editor

In that spreadsheet: **Extensions → Apps Script**.

A new browser tab opens with a file called `Code.gs` containing an empty
`myFunction()`.

Rename the project: click **"Untitled project"** at the top left, type
`SQL Query Notebook`, press Enter.

---

## STEP 3 — Paste the backend

The editor already has a file called `Code.gs`. Reuse it.

1. Click anywhere in the code area.
2. Select everything: **Ctrl+A** (Cmd+A on Mac).
3. Delete it.
4. Open `Code.gs` from this folder, select all, copy.
5. Paste into the editor.
6. **Ctrl+S** to save.

It is a large file — give the paste a few seconds to finish before saving.

> **Navigating it later:** the file opens with a table of contents. Every module
> has a banner like `SECTION 07 — Categories.gs`. Press **Ctrl+F** and search
> for `SECTION 07` to jump straight there.

---

## STEP 4 — Paste the front end

1. Next to **Files** on the left, click **+** → **HTML**.
2. Name it exactly `Index` — **no `.html`**, the editor adds that.
3. Delete the placeholder content the editor puts there.
4. Open `Index.html` from this folder, select all, copy, paste.
5. **Ctrl+S**.

You now have exactly two files: `Code.gs` and `Index.html`.

> The name must be `Index` with a capital I. `index` will not work —
> `doGet()` looks for `Index` by name.

---

## STEP 5 — Set the manifest

1. Click **Project Settings** (the gear icon, left sidebar).
2. Tick **"Show 'appsscript.json' manifest file in editor"**.
3. Go back to **Editor**. There is now an `appsscript.json` file.
4. Open it, select all, and replace it with `appsscript.json` from this folder.
5. Change `"timeZone"` if `Asia/Karachi` is not yours.
6. **Ctrl+S**.

This grants the script the access it needs: Sheets, Drive, sending mail, and
reading the signed-in user's email address.

---

## STEP 6 — Run setupSystem()

1. In the toolbar, open the function dropdown (it says `doGet` or `myFunction`).
2. Choose **`setupSystem`**.
3. Click **Run**.

---

## STEP 7 — Authorise

The first run stops and asks for permission.

1. **Review permissions** → pick your account.
2. You will see **"Google hasn't verified this app."** That is expected — it is
   your own private script, not a published add-on.
   Click **Advanced** → **Go to SQL Query Notebook (unsafe)**.
3. Read the list, click **Allow**.

| It asks for | Because |
|---|---|
| See, edit, create and delete your spreadsheets | The database lives in Sheets |
| See, edit, create and delete your Drive files | Folder tree, `.sql` files, attachments, backups |
| Send email as you | Share and update notifications |
| See your primary email address | Identifies who is signed in — the basis of every permission check |
| Manage scripts and triggers | The hourly session cleanup job |
| Connect to an external service | Fetches the `.xlsx` export of the sheet during backup |

Click **Run** again. In the **Execution log** at the bottom you should see:

```
=========================================
 SQL Query Notebook — setup complete
=========================================
Spreadsheet : SQL Query Notebook DB
Sheets created  : Queries, Categories, Databases, Users, Roles, ...
Seeded          : Roles, Permissions, Categories, Databases, Settings,
                  Admin user (you@company.com)
Drive root      : https://drive.google.com/...
```

Switch to the spreadsheet tab — it now has 14 tabs with formatted headers, and
a Drive folder called **SQL Query Notebook** now exists.

> `setupSystem()` is safe to run again whenever you like. It only adds what is
> missing and never overwrites data.

---

## STEP 8 — Load the demo data (optional)

Function dropdown → **`seedDemoData`** → **Run**.

Inserts 10 example queries (`SQL-000001` to `SQL-000010`) so the app has
something to show. Every one is tagged `demo`.

> The demo SQL is illustrative and assumes a courier/logistics schema that will
> not match yours. To remove it later, filter by the `demo` tag, delete to
> Trash, then permanently delete from Trash.

---

## STEP 9 — Deploy

1. Top right: **Deploy → New deployment**.
2. Click the gear next to "Select type" → **Web app**.
3. Fill in:

   | Field | Value |
   |---|---|
   | Description | `v1.0.0` |
   | Execute as | **Me (you@company.com)** |
   | Who has access | **Anyone within `<your organisation>`** |

4. **Deploy** → authorise if asked → **copy the Web app URL**.

```
https://script.google.com/a/macros/yourcompany.com/s/AKfycb.../exec
```

Share **that URL** with your team. Never share the spreadsheet.

### Do not change "Execute as"

"Execute as: Me" is what lets the script read the spreadsheet while your users
have no access to it at all. That is the entire basis of the permission model.

If you set it to "User accessing the web app", every user would need edit access
to the spreadsheet — handing them the whole dataset and letting them rewrite the
permission table.

### Choosing "Who has access"

| Option | When | Effect |
|---|---|---|
| Anyone within `<org>` | ✅ A company | Google sign-in resolves reliably |
| Anyone with a Google account | Mixed / external | Google identity may come back empty; the app falls back to application passwords |
| Only myself | Testing | Nobody else can open it |

---

## STEP 10 — Open it

Open the Web app URL. You should land on the dashboard with your name top right
and an **Admin** badge.

If you see *"Your Google account is not registered"*, the email Google reported
does not match the **Users** tab. Check the spelling there.

---

## STEP 11 — Add your team

**Users → Add User.** Enter name, email and role.

| Role | Can |
|---|---|
| **Admin** | Everything |
| **Manager** | Create, edit and delete any query; restore; export |
| **Editor** | Create and edit their own queries; share |
| **Viewer** | View, search and copy only |

They sign in at the same URL with their own Google account.

---

## STEP 12 — Finish the setup

Run these once each from the function dropdown:

| Function | Does |
|---|---|
| `installTriggers` | Adds the hourly job that expires stale sessions |
| `sendTestNotification` | Confirms email delivery — check your inbox |
| `showConfiguration` | Prints the configuration to the log (no secrets) |

**Add a weekly backup:** in the editor click the **clock icon** (Triggers) →
**Add Trigger** → function `runScheduledBackup`, time-driven, week timer, pick
Sunday 3–4am.

---

## Updating later

Change the code, then:

**Deploy → Manage deployments → ✏️ edit → Version: New version → Deploy.**

The URL stays the same. Users get the new code on their next page load. Your
data is untouched.

> Use **New version** on the *existing* deployment. Creating a *new deployment*
> gives you a different URL and leaves everybody on the old code.

To roll back: same dialog, pick an older version, **Deploy**.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| *"The application is not set up yet"* | Run `setupSystem()` (Step 6) |
| *"Your Google account is not registered"* | Working as intended — add the email under **Users** |
| Blank page after deploying | The HTML file is not named exactly `Index` |
| `Missing sheet "..."` | A tab was renamed or deleted. Re-run `setupSystem()` |
| Sign-in loops back to login | The deployment is not "Execute as: Me". Redeploy |
| Emails not arriving | Daily Gmail quota hit, or notifications off in **Settings**. Run `sendTestNotification()` |
| Changes not showing for users | You saved but did not deploy a **New version** |
| *"Too many requests"* | Rate limit, 180 calls/minute/user. Wait a minute |

---

## Before you let people in

- [ ] Deployment is **Execute as: Me**, access **your organisation**
- [ ] The spreadsheet is shared with **nobody** — check `File → Share`
- [ ] Two-factor authentication is on for the deploying Google account
- [ ] **Users** contains only people who should have access, with correct roles
- [ ] A Viewer account really cannot see New Query, Edit or the Manage section
- [ ] `installTriggers()` has been run
- [ ] A weekly `runScheduledBackup` trigger exists
- [ ] Demo data removed, or knowingly kept

**The single most important line:** anyone with edit access to the spreadsheet
bypasses every permission in this application. Share the web app URL, never the
Sheet.
