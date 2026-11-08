# TEX ZIPPERS (BD) LIMITED — Recipe Calculator v6

Zipper tape dyeing recipe calculator: **Lab to Bulk** and **Machine-to-Machine** conversion, Addition (reshade), printable **Recipe Card** and **Prescription Card**, Recipe Library, users with roles, and cloud sync through **Google Apps Script** and/or **Supabase**.

Everything runs in the browser from one file (`index.html`). Data is stored in the browser's **IndexedDB** and, when cloud sync is on, in your own Google Sheet or Supabase project.

## Files

| File | What it is |
|---|---|
| `index.html` | The whole app |
| `manifest.webmanifest`, `sw.js`, `icon-192.png`, `icon-512.png` | Install as an app + works offline (all must be uploaded) |
| `.nojekyll` | Tells GitHub Pages to serve the files as they are |
| `apps-script/Code.gs` | Backend for Google Apps Script sync (paste into Apps Script — **not** used by GitHub Pages) |
| `supabase/schema.sql` | Backend for Supabase sync (run in the Supabase SQL Editor — **not** used by GitHub Pages) |

## Publish on GitHub Pages

1. Create a new repository on GitHub (for example `recipe-calculator`). It can be **private** if your GitHub plan allows Pages on private repos; otherwise public is fine — **no data or passwords are stored in the repository**.
2. Upload these files to the repository root (Add file → Upload files → Commit): `index.html`, `manifest.webmanifest`, `sw.js`, `icon-192.png`, `icon-512.png`, `.nojekyll`, `README.md`. The `apps-script` and `supabase` folders are optional (they are not used by the website).
3. Repository → **Settings → Pages** → *Build and deployment* → Source: **Deploy from a branch** → Branch: **main**, folder **/ (root)** → Save.
4. After a minute the site is live at `https://<your-username>.github.io/<repository>/`.
5. Open it on the first computer → create the **Admin** account → save the recovery key.

To update later, upload the new `index.html` (and bump `CACHE` in `sw.js`).

**Install as an app:** open the site in a normal (not Incognito) Chrome/Edge window → install icon in the address bar, or menu ⋮ → *Install app*. On Android: Chrome menu ⋮ → *Install app* / *Add to Home screen*. Data in the browsers and in the cloud is not affected.

## Users and roles

* The first account is the **Admin**. Admin adds users in **Users & Roles**, gives each a role and a temporary password; the user must choose their own password at first sign-in.
* Default roles — change them any time in the permission table, or add your own roles:

| Permission | Admin | Manager | Chemist | Viewer |
|---|:-:|:-:|:-:|:-:|
| Edit calculator inputs | ✓ | ✓ | ✓ | |
| Save / update recipes | ✓ | ✓ | ✓ | |
| Print cards | ✓ | ✓ | ✓ | ✓ |
| Delete recipes | ✓ | ✓ | | |
| Export (Excel / PDF / backup) | ✓ | ✓ | ✓ | |
| Import a library backup | ✓ | ✓ | | |
| Edit suggestion lists | ✓ | ✓ | | |
| Run cloud sync | ✓ | ✓ | ✓ | |
| Cloud sync settings, force push / pull | ✓ | | | |
| Activity log | ✓ | ✓ | | |
| Users & roles | ✓ | | | |
| Company header, full backup / restore, danger zone | ✓ | | | |

* Buttons a role may not use are hidden, and every action is checked again when it runs.
* The last active Admin cannot be disabled, deleted or demoted.
* **Activity Log** records sign-ins (including failed ones), saves, updates, deletes, prints, user and role changes and sync actions.
* Passwords are stored only as salted PBKDF2-SHA256 hashes (150,000 rounds). This is a privacy lock for a shared workplace, not bank-grade security: anyone with full control of a browser can read its stored data.

## Cloud sync

Choose **Apps Script**, **Supabase**, or **Both** in **Cloud Sync → Settings**. With *Both*, every change is written to both clouds, so either one is a live backup of the other.

How the sync keeps data safe:

* Each record has one permanent id. A record is replaced only by a **newer** version — sync can never duplicate recipes or bring back an old copy.
* Deletes are kept as tombstones so they reach every device.
* Only one sync runs at a time (also across browser tabs). Requests use POST with a timeout; failures retry automatically (20 s → 15 min backoff) and changes made offline wait as *pending* until the internet is back.
* Devices pull "everything after position N" using a server counter, so nothing is skipped or received twice.
* **Full re-sync** downloads everything again (safe). **Force push** / **Force pull** overwrite one side with the other — use only to repair.

### A. Google Apps Script

1. Create a Google Sheet → **Extensions → Apps Script**.
2. Paste `apps-script/Code.gs`, Save.
3. Run the function **`setupSyncToken`** once, allow permissions, copy the token from the Execution log.
4. **Deploy → New deployment → Web app** — Execute as **Me**, Who has access **Anyone** → copy the URL ending in **`/exec`**.
5. In the app: Cloud Sync → Apps Script → paste the URL and token → **Test connection** → **Save sync settings**.

After changing `Code.gs`, use *Manage deployments → Edit → New version* so the same URL keeps working.

### B. Supabase

1. Create a Supabase project → **SQL Editor**.
2. Open `supabase/schema.sql`, change `CHANGE-ME-TO-A-LONG-RANDOM-KEY` to your own long secret, run the whole file.
3. **Project Settings → API**: copy the **Project URL** and the **anon / publishable** key.
4. In the app: Cloud Sync → Supabase → URL, key and your sync key → **Test connection** → **Save**.

The tables are locked by Row Level Security; the public key can only call the three sync functions, and each one checks the sync key.

### New computer / phone

Open the site → on the first screen choose **Restore from cloud** (or **Restore from backup file**) → sign in with your existing username. Each device keeps its own cloud settings, so enter them once per device.

## Backups

* **Settings → Full Backup** (Admin): one JSON file with recipes, users, roles, lists, company header and activity log. Restore only adds records that are newer than what the device has.
* **Recipe Library**: JSON backup, readable HTML backup, PDF and Excel exports.

## Moving from v5

Opening v6 in the same browser moves the old v5 data automatically: the saved recipes, suggestion lists, last-backup date and the old login (it becomes the **Admin**). The old v5 data is left in place as a safety copy.

## Changelog

**v6.0**
* Company header is now **TEX ZIPPERS (BD) LIMITED** on every card, QR code and export (editable in Settings).
* Multi-user sign-in with roles and a permission matrix, forced password change for new users, Activity Log.
* Storage moved to IndexedDB (localStorage fallback), with automatic migration from v5.
* Cloud sync: Google Apps Script and Supabase (either or both), auto sync, offline queue, retry, force push / pull, restore on a new device.
* Save on a loaded job now **updates** it (no more duplicate copies); **Save as new** added. New job clears the calculator without reloading.
* Full backup / restore of every table. Installable app (PWA) and offline support.
* Bug fixes: names containing quotes broke input boxes and the library buttons (escaping fixed; library actions no longer use inline code); tape quantity 0 showed *Infinity*; default date and file names used UTC (wrong date before 6 am in Bangladesh); comma decimals (0,5) were ignored; imported/old jobs with missing fields could crash loading and exports; print mode could stay stuck if the browser never reported the end of printing; validation could crash on non-text fields.
