# Jobs dashboard v2: multi-user

Plan for turning `/jobs` from a single-person tool (wired to one Google Sheet) into something any Google user can sign in to and use with their own data.

## Goals

- Anyone with a Google account can sign in and get **their own** watchlist, documents and templates.
- Each user's data stays in **their own** Google Drive. The site stores nothing and has no backend.
- Existing v1 users (the original sheet) keep working after a one-time connect step.
- No Google "unverified app" warning and no test-user list to maintain.

New users can set up their sheet, skills and scheduled searches with the [setup prompt](jobs-v2-setup-prompt.md).

## Non-goals

- Sharing one watchlist between several people (each user has their own sheet).
- Server-side automation. The Run log is still filled by the user (or by their own tools).
- Native mobile apps.

## What changes from v1

| Area | v1 | v2 |
|---|---|---|
| Which sheet | `SHEET_ID` constant in `jobs.html` | Chosen per user at first sign-in, remembered in their Drive |
| OAuth scope | `spreadsheets` (every sheet in the account; sensitive) | `drive.file` + `drive.appdata` (only files the app creates or the user picks; non-sensitive) |
| Sheet setup | Created by hand | "Create my watchlist" builds it with tabs, headers, dropdowns and starter rows |
| Personal details | Name hard-coded (print title), menu links to the owner's profiles | Read from a `Profile` tab |
| Starter templates | Owner's own email, cover letter and paragraphs | Neutral templates with placeholders |
| Documents | Paste a Drive link | Upload or pick from Drive; replacing the CV keeps the same link |
| Run log | Filled by the owner's tools | Same, plus an "Add finding" form for people without tools |

## User flows

### First sign-in

1. Sign in with Google (token client, scopes below).
2. Read `config.json` from the user's Drive app data folder.
   - Found: open that spreadsheet and continue to the dashboard.
   - Not found: show the **Connect your watchlist** screen.
3. Connect screen offers:
   - **Create my watchlist**: creates a new spreadsheet in the user's Drive (see [Sheet schema](#sheet-schema-v2)), then asks for their name, phone and email to fill the `Profile` tab.
   - **Use an existing sheet**: opens Google Picker filtered to spreadsheets. Picking a file grants `drive.file` access to it. The app checks the schema and offers to upgrade it (see [Migration](#migrating-a-v1-sheet)).
4. Save `{ spreadsheetId, schemaVersion }` to `config.json` in app data and cache it in `localStorage` (keyed by the user's Google account id) for faster loads.

### Returning user

Sign in, read the cached config (falling back to app data), load the sheet. If the sheet was deleted or access was revoked (404/403), clear the config and show the connect screen with a short explanation.

### Switching sheets

Menu item **Switch watchlist** opens the connect screen again. Useful for testing or keeping separate searches.

### Signing out

Revoke the token, clear in-memory data and the cached config for that account. Nothing else is stored locally.

## Auth and Google Cloud setup

### Scopes

- `https://www.googleapis.com/auth/drive.file`: create the watchlist, read and write sheets the app created or the user picked, upload documents. The Sheets API accepts this scope for those files.
- `https://www.googleapis.com/auth/drive.appdata`: store `config.json` in a hidden per-app folder so the choice follows the user across devices.
- Optional: `openid email` to show which account is signed in and to key the local cache.

All of these are non-sensitive, so Google's sensitive-scope verification isn't required. Check Google's current OAuth policy before publishing, as brand verification can still apply (for example when the consent screen shows a logo).

### Cloud Console checklist

- [ ] Enable **Google Sheets API**, **Google Drive API** and **Google Picker API**.
- [ ] OAuth consent screen: user type **External**, add the scopes above, add app name, support email, **privacy policy URL** and home page.
- [ ] Publishing status: **In production** (Testing mode limits sign-in to listed test users).
- [ ] OAuth client (Web): authorised JavaScript origins for production and `http://localhost:8000`.
- [ ] API key for Picker, restricted to the site's HTTP referrers and to the Picker API.
- [ ] Note the Cloud **project number**; Picker needs it as the app ID so picked files are granted to this app.

### Privacy policy

A short page (e.g. `/jobs/privacy`) is required for a production consent screen. It should say: data is read and written in the user's own Google Drive from their browser; nothing is sent to or stored by the site; access can be revoked at myaccount.google.com/permissions.

## Sheet schema v2

One spreadsheet per user. Row 1 is headers. Column order stays the same as v1 so existing tools that write to the sheet keep working.

### `Companies` (A-M, unchanged)

Company, Priority, Status, Role seen, What they do, Found via, Post link, Posted, Contact, Careers page, Last checked, Applied on, Notes.

- Data validation dropdowns: Priority (High/Medium/Low/Avoid), Status (Not checked/No other openings/Has other openings/Applied/Not interested).
- Frozen header row and first column.

### `Run log` (A-G, unchanged)

Date, Company, Type (Job/News/Added/Contract/Freelance), Title, Link, Fit (0-10), Note.

### `Documents` (A-H, unchanged)

Name, Kind (CV/Cover letter/Email/Paragraph), Link, Subject, Body, Updated, Notes, Industry.

Starter rows for a new sheet:

- `CV` (Kind CV, no link yet)
- `Application email` (Kind Email) with neutral text using `[Name]`, `[job title]`, `[Company]`, `[industry paragraph]`, `[one concrete thing from the ad]` and `[Your name]`
- `Cover letter (text)` (Kind Cover letter) in the same style
- One `Paragraph` row per industry (IoT, Fintech, PropTech, Energy tech, Data-intensive) with prompts such as "Describe a project where you handled device data…", for the user to rewrite

### `Profile` (new, key/value in A:B)

| Key | Example | Used for |
|---|---|---|
| Full name | Jane Doe | Print title, `[Your name]` placeholder |
| Phone | 012 345 6789 | `[Your phone]` |
| Email | jane@example.com | `[Your email]` |
| Signature | Kind regards,\nJane Doe | `[Signature]` |
| Cover letter file name | `{name} - Cover letter - {company}` | Print/PDF title |
| Schema version | 2 | Migration checks |

Profile values are available to every template as placeholders, so the composer fills them automatically.

## Migrating a v1 sheet

On connect, read the spreadsheet's tab list and header rows:

1. Missing `Profile` tab: create it, ask for name/phone/email, set `Schema version` to 2.
2. Missing `Documents` tab or `Industry` column: create/extend without touching other columns.
3. Missing `Companies` or `Run log`: offer to create them with headers.
4. Headers that don't match: show which tab and column, and stop rather than guess (other tools may depend on the layout).

The original sheet becomes a normal v2 sheet after this. Opening it in v2 with `drive.file` requires picking it once through Picker.

## Code changes in `jobs.html`

### Config and API layer

- Replace `const SHEET_ID` with a runtime `spreadsheetId` set after connect; build `API` from it.
- New `loadConfig()` / `saveConfig()` using Drive app data (`files.list?spaces=appDataFolder`, multipart create/update of `config.json`).
- `api()` keeps the 401 handling. A 403/404 on the sheet clears the config and returns to the connect screen.

### Connect screen

- New signed-in state before the dashboard: **Create my watchlist** / **Use an existing sheet**, with progress and error notices in the same style as the sign-in card.
- `createWatchlist()`: `spreadsheets.create` with the four tabs, header rows, frozen rows, data validation and starter rows in one request, then write `config.json`.
- `pickWatchlist()`: load `https://apis.google.com/js/api.js`, then `google.picker` with the API key, OAuth token, app ID and a spreadsheet view.

### Profile and placeholders

- Load `Profile!A:B` alongside the other tabs into a `profile` map.
- Composer fills `[Your name]`, `[Your phone]`, `[Your email]` and `[Signature]` from the profile before user-typed fields.
- Print title comes from `Cover letter file name`, with no name in the code.
- New **/ profile** section (or a panel in `/ documents`) to edit these values, using the same find-row-by-key write pattern.

### Documents

- **Upload CV / Upload file**: Drive multipart upload into a `Job search` folder the app creates, then write the link and date to `Documents`.
- **Replace file**: `files.update` with new media, so the Drive link stays the same and downloads always get the latest version.
- **Pick from Drive**: Picker, for files the user already has.
- Keep "Update link" for pasting links to files outside the app's access.

### Run log

- **Add finding** form on `/ news` and `/ contract & freelance` (Date defaults to today, Type preset by tab), appending to `Run log`.
- Empty states explain that the Run log is where findings live and how to add them.

### Generic site chrome

- Remove the owner's GitHub, LinkedIn and email from the `/jobs` menu, or show the signed-in user's email instead.
- Neutral page title, sign-in copy and meta.
- Industry hints (`INDUSTRY_HINTS`) stay in code as defaults; a future option is a `Hints` column on paragraph rows.

### Unchanged

Write safety (re-read and find the row by name before every write), `textContent`-only rendering, `http(s)` link checks, theme handling, loaders and layout.

## Hosting

Two options:

1. **Keep it at `acsono.co.za/jobs`** with neutral copy. Quickest; the site's top bar still says acsono.
2. **Standalone** repo and GitHub Pages site (or a subdomain such as `jobs.acsono.co.za`) with its own name and privacy page. Cleaner for sharing and lets the personal site stay personal.

Recommendation: build v2 in place, then move it to a subdomain before inviting people. The OAuth client just needs the new origin added.

## Security and privacy

- Access token kept in memory only; never written to storage.
- `drive.file` limits access to the watchlist, uploaded documents and anything the user explicitly picks.
- Add a Content Security Policy meta tag allowing only `self`, the Tailwind CDN, Google Fonts, `accounts.google.com`, `apis.google.com`, `sheets.googleapis.com` and `www.googleapis.com`.
- All sheet text still rendered with `textContent`; links still restricted to `http(s)`.
- No analytics on `/jobs` (v1 already excludes Google Tag Manager).

## Testing

- Two or three Google accounts: new user (create), v1 sheet owner (pick + migrate), user who revokes access mid-session.
- Picker on desktop and mobile browsers; pop-up blockers.
- Expired token (about an hour) during a write.
- A sheet another tool edits and re-sorts while the dashboard is open.
- Light/dark, phone width, reduced motion.

## Milestones

1. **Config + scopes**: runtime spreadsheet id, Drive app data config, `drive.file`/`drive.appdata`, connect screen with "Use an existing sheet" via Picker. The owner's current sheet works through it.
2. **Create watchlist**: `spreadsheets.create` with schema, validation and neutral starter rows; `Profile` tab and placeholders; print title from profile.
3. **Documents upload**: Drive folder, upload, replace file, pick from Drive.
4. **Run log form** and empty states.
5. **Generic chrome + privacy page**, CSP, consent screen to production, optional move to a subdomain.

## Open questions

- Should the connect step also offer "Make a copy of a template sheet" (one shared public template) instead of building the sheet in code?
- Keep v1's `spreadsheets` scope as an opt-in for power users whose tools write to other sheets?
- Is a per-user `Hints` column worth it, or are the built-in industry hints enough?
- Standalone name and domain for the shared version.
