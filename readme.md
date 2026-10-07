# Acteon Marketing Spend

A single-page web app with two parts:

- **Dashboard**: the year's spend against budget. Data comes from the Marketing workbook (for example the "2026" tab), imported in the browser.
- **Budget planner**: an editable plan for next year, started from this year's forecast, that you can export to Excel for finance.

It runs in **preview mode** until Firebase is connected. In preview mode there's no login and everything is saved only in the browser you're using, which is handy for trying it out.

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole app. Paste your Firebase config near the top. |
| `firestore.rules` | Security rules. Paste them into Firestore > Rules. |
| `netlify.toml` | Security headers for Netlify. Optional, but keep it in the folder. |

## One-time Firebase setup (about 10 minutes)

1. **Create the project.** Go to <https://console.firebase.google.com>, choose **Add project** and name it (for example `acteon-marketing-spend`). Google Analytics isn't needed.
2. **Register a web app.** Go to Project settings > General > Your apps > **Web (</>)**. Copy the `firebaseConfig` values into `window.FIREBASE_CONFIG` near the top of `index.html`. These values are designed to be public; access is protected by the login and the rules.
3. **Turn on login.** Go to Build > Authentication > Get started > Sign-in method, and enable **Email/Password**.
4. **Add users.** Go to Authentication > Users > **Add user** for each person (email + temporary password). There's no self sign-up. People can change their password with "Forgot password?" on the login screen.
5. **Create the database.** Go to Build > Firestore Database > **Create database**, choose Production mode, and pick a US location (for example `nam5`).
6. **Paste the rules.** Go to Firestore > **Rules**, replace everything with the contents of `firestore.rules`, and click **Publish**.
7. **Choose who can import and edit plans.** In Firestore > Data, start a collection called `admins`. Add a document whose **Document ID is the person's email in lowercase** (for example `david.teter@acteongroup.com`). Give it any field, such as `role: "admin"`. Repeat for each admin. Everyone else who signs in gets view-only access.
8. **Allow the Netlify domain.** After the first deploy, go to Authentication > Settings > **Authorized domains** and add your site's domain (for example `acteon-mkt-spend.netlify.app`).

## Deploy

Drag the folder (or just `index.html` + `netlify.toml`) onto **app.netlify.com > Sites > Add new site > Deploy manually**. To update later, open the site > **Deploys** and drag the folder in again.

## Monthly update

1. Save the latest Marketing workbook (.xlsx).
2. Open the app and click **Import data**, then choose the file.
3. Check the preview: tab, year, "Actuals through" month, budgets, and any warnings. Then click **Publish to dashboard**.

Everyone sees the new numbers right away. Each import is kept, so **History > Restore** brings back an earlier version.

**What the importer expects** (the current 2026 tab layout): a header row with JAN–DEC, the row above it marking each month as ACTUAL or Projection, a TOTAL BUDGET column, "… COST CENTER" section rows, GL header rows like `62311-00-03  Brochures`, and "Total … Expenses by month" rows that carry each cost center's budget. Totals are always recalculated from the months; the workbook's own TOTAL cells are only used to flag mismatches.

## Budget planner

- **Create plan:** pick the base year, the plan year, the starting amounts (full-year forecast or blank), an across-the-board % and the monthly spread. Cost-center targets start at the base year's budget × that %.
- **Edit:** type into any month, or type a new full-year total to rescale that row. Use **+ Add line** under a GL account for new spend, **×** to remove a line, and **Adjust shown rows** to apply a % to whatever the filter is showing. Changes autosave to Firebase. If two people edit at the same moment, the last save wins.
- **Targets:** type a total in the bottom row of the cost-center targets table (or in "Total target budget" when creating the plan) and it is split across cost centers by their current share.
- **Ask the planner:** a prompt line above the Scenario lab. Type things like `average Travel to the full 12 months based on previous months`, `spread KOL retainer evenly`, `increase Travel by 10%`, `set Engel to 24000` or `remove Engel`. It previews the change (untick lines to skip them) before applying, and it is undoable and logged.
- **Scenario lab** (has a built-in "How scenarios work" guide): *Shift budget* raises one or more areas at once (%, $, or $ each × qty, optionally as a new named line such as "4 new KOLs") and suggests where to cut; *Hit target* closes a gap to target; *Remove lines* drops line items (or sets them to $0); *Snapshots* saves, compares and restores. Every apply saves a snapshot, so **Undo last scenario** rolls it back.
- **Mark as submitted** locks the plan, and **Reopen for edits** unlocks it.
- **Export to Excel** downloads `Acteon_Marketing_Budget_<year>.xlsx` (Aptos font) with a Summary tab and a full budget tab. Every total is a live formula, so finance can keep editing in Excel.
