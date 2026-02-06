# Day 6 — Merge Form responses into canonical risk register

Goal
Move new risk submissions from the Google Form Responses sheet into artifacts/risk-register.csv, assign unique IDs, and avoid duplicates.

Summary
Option 1 — Manual (recommended for beginners)
- Quick and safe. Copy rows from the Form Responses sheet, create new IDs, paste into artifacts/risk-register.csv, commit.

Option 2 — Semi-automated (recommended when you have many rows)
- Use a small local script that reads exported CSV from the Form Responses sheet and appends to artifacts/risk-register.csv while auto-generating IDs.

Pre-checks (mandatory)
1. Open `artifacts/risk-register.csv` and note the highest existing ID (numeric). For example, if the last ID is 12, remember 12.
2. Ensure the Form Responses sheet has Score and Priority columns computed (Day 5).
3. Ensure Form Responses sheet has a "Processed" column (checkbox or text) you can use to mark exported rows.

Column mapping (canonical `artifacts/risk-register.csv`)
ID,Title,Description,Category,Likelihood (1-5),Impact (1-5),Score (LxI),Priority,Owner,Mitigation,Status,DateIdentified,Notes

Google Form → Sheet typical mapping (Form Responses sheet columns)
Timestamp,Title,Description,Category,Likelihood,Impact,Owner,Mitigation,Status,Notes,Score,Priority,Processed

OPTION 1 — Manual merge (30–40 minutes first time, ~10–15 min typical)
1. Identify new rows:
   - In your Form Responses sheet, filter where Processed is unchecked or blank.
2. Compute new IDs in the sheet:
   - In a new column (call it NewID), set the header `NewID`.
   - Suppose the highest existing ID in artifacts file is 12. In the NewID cell for first response row (row 2), put:
     `=ROW()-1 + 12`
     - Explanation: ROW()-1 gives 1 for row 2, 2 for row 3, etc. Adding 12 makes IDs 13, 14, ...
   - Fill down the NewID column for all unprocessed rows.
   - Alternatively, set NewID = CONCATENATE("R-", ROW()-1 + 12) if you prefer a prefix.
3. Prepare rows for copy:
   - Create a small export sheet (new tab) where you assemble columns matching the canonical order:
     - Column A: NewID (or NewID stripped of prefix)
     - Column B: Title
     - Column C: Description
     - Column D: Category
     - Column E: Likelihood
     - Column F: Impact
     - Column G: Score (from sheet)
     - Column H: Priority (from sheet)
     - Column I: Owner
     - Column J: Mitigation
     - Column K: Status
     - Column L: DateIdentified (use Timestamp from form or =TODAY())
     - Column M: Notes
   - Use formulas to map correct cells into the export tab so copy will match the artifacts CSV order exactly.
4. Copy & append:
   - Select the assembled export rows → Copy → Open your local `artifacts/risk-register.csv` in a CSV editor (or plain text editor / Excel) → Paste as new rows at the end.
   - Verify formatting (no extra commas breaking the CSV). If using Excel, save as CSV.
5. Mark processed:
   - In the Form Responses sheet, mark those rows as Processed = TRUE (check the box or write "yes").
6. Commit to Git:
   - See git steps below.
7. NOTE: If you make a mistake, revert locally (git checkout -- artifacts/risk-register.csv) and re-run steps.

OPTION 2 — Semi-automated (export + local script)
- Steps:
  1. In the Form Responses sheet, filter unprocessed rows and File → Download → Comma-separated values (.csv) → save as `new_responses.csv`.
  2. Run the small Python script `scripts/append_responses.py` (provided in repo) which:
     - Reads artifacts/risk-register.csv and new_responses.csv
     - Finds highest ID from artifacts
     - Generates new numeric IDs, maps columns, appends rows, writes updated artifacts/risk-register.csv
     - Optionally writes a backup `artifacts/risk-register.backup.csv` before change
  3. Open artifacts/risk-register.csv and confirm.
  4. Mark rows processed in the sheet.
  5. Commit to Git.

Safety tips
- Always open the updated `artifacts/risk-register.csv` and verify scores and priorities before committing.
- Keep a backup copy before bulk appends: `cp artifacts/risk-register.csv artifacts/risk-register.backup.csv`
- Use small batches (5–20 rows) until you’re comfortable.

Commit & Branch (example)
- Branch: `day6-merge`
- Commit message: `Day 6: Merge new Form responses into canonical risk-register.csv and add merge instructions`

What to commit
- Updated `artifacts/risk-register.csv` (with new rows appended)
- `docs/merge_instructions.md` (this file)
- Optional: scripts/append_responses.py (if you used the script)

Interview/portfolio note
- In your commit PR description or Day6.md, say what you did: "Merged N new stakeholder risk submissions into artifacts/risk-register.csv, added unique IDs, and updated the Form to mark processed rows. This improves intake traceability and prevents duplicates."
