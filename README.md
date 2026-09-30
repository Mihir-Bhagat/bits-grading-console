# BITS Pilani Digital – Advanced Grading Console

**CodeForge V1.0 · Debug. Reimagine. Deploy.**

**Live app:** [https://mihir-bhagat.github.io/bits-grading-console/](https://mihir-bhagat.github.io/bits-grading-console/)  
**Repository:** [https://github.com/Mihir-Bhagat/bits-grading-console](https://github.com/Mihir-Bhagat/bits-grading-console)

A single-file, browser-only grading console. Upload an Excel marks sheet, check data quality, fix problems in place, explore grade-boundary changes with impact preview, and export final grades as CSV.

**No backend · No install · Data never leaves the browser.**

---

## Quick start

1. Open the [live app](https://mihir-bhagat.github.io/bits-grading-console/) (or open `index.html` in any modern browser).
2. Enter the **instructor name**.
3. Upload a `.xlsx` / `.xls` file **or** click **Try sample data**.
4. Review the **Data Quality Report** and fix issues in the student records table.
5. Adjust grade ranges — use **What-If** to preview impact before applying.
6. Click **Finalize & Download** → review the summary → **Confirm & Export**.

You receive `grades_<Course>.csv`.

### Expected input

| Column       | Rule                          |
|--------------|-------------------------------|
| `BITS ID`    | Student ID                    |
| `Course`     | Course name                   |
| `Total Marks`| Whole number **0–100**        |

Students who should receive **NC** must **not** appear in the file.

---

## Challenge stages

| Stage        | Goal                                                      | Outcome                                                          |
|--------------|-----------------------------------------------------------|------------------------------------------------------------------|
| **1. Debug**     | Find and fix intentional bugs                         | **12 bugs** fixed and documented                                 |
| **2. Reimagine** | Turn a fragile demo into a tool an instructor can trust | **6 product enhancements** focused on safety, speed, and clarity |
| **3. Deploy**    | Make it publicly usable                               | Static single-file app (GitHub Pages)                            |

---

## Stage 1 — Bugs fixed

| #  | Bug                                                                 | Root cause                                          | Fix                                                                                      |
|----|---------------------------------------------------------------------|-----------------------------------------------------|------------------------------------------------------------------------------------------|
| 1  | Timer stayed at `00:00` for the first second                        | Display updated only on the first interval tick     | Paint immediately, then every second                                                     |
| 2  | Min and Max stat boxes were swapped                                 | HTML `id`s were crossed                             | Corrected the ids                                                                        |
| 3  | Course dropdown listed the same course once per student             | One `<option>` created per data row                 | Unique list via `new Set(...)`                                                           |
| 4  | Second upload kept courses from the first file                      | Options were never cleared                          | Reset the dropdown before adding courses                                                 |
| 5  | Re-selecting the same file did nothing                              | Browser does not fire `change` for an identical value | Clear the input value after reading                                                    |
| 6  | Blank, text, or out-of-range marks silently vanished                | Non-numbers failed every grade comparison           | Only valid 0–100 numbers are graded; the rest are reported                               |
| 7  | Histogram bars overflowed for larger classes                        | Fixed bar height `count × 12`                       | Scale bars against the tallest bin                                                       |
| 8  | Bell curve produced `NaN`/`Infinity` for one student or identical marks | Standard deviation was 0                        | Guard + friendly message; align curve with bar centres                                   |
| 9  | Reported grading time included idle setup time                      | Clock started at page load                          | Start when the first course is opened                                                    |
| 10 | Stats/chart showed `undefined` / `NaN` for an empty course          | No empty-data handling                              | Show `—` and an explanatory message                                                      |
| 11 | File picker only accepted `.xls`                                    | `accept=".xls"`                                     | Accept `.xlsx,.xls`                                                                      |
| 12 | Weak export (commas broke CSV, fixed filename, weak checks)         | No escaping / incomplete eligibility                | CSV-escape all fields, name files `grades_<Course>.csv`, re-check eligibility at export |

The full **Bug Fix Log** (reproduction steps + testing) is submitted separately as PDF/DOC in the required tabular format.

---

## Stage 2 — Enhancements

**Guiding question from the brief:**  
*“What would make an instructor faster, clearer, and safer?”*

Grades are consequential. The focus was **safety and trust**, not decoration.

### 1. Data Quality Gate & Report *(safer)*

On every upload the whole file is scanned.

- Detects: blank marks, non-numeric values, marks outside 0–100, fractional marks, missing BITS IDs, missing courses, duplicate ID+Course pairs, missing required columns, trailing whitespace.
- Shows a readiness banner: **Ready / Review required / Blocked**.
- Lists every issue with row numbers.
- **Rule:** the tool never silently rounds, clamps, or drops the instructor’s data.

### 2. Inline Student Records editor *(faster)*

After the report flags a problem, the instructor does not need to go back to Excel.

- Edit BITS IDs and marks in place for the selected course.
- Add or delete students.
- Search by BITS ID; filter to **issues only**.
- One-click **Round fractional marks**.
- Unlimited **Undo / Redo** (buttons + Ctrl/Cmd+Z / Y).
- Download a **corrected .xlsx**.
- Grades, charts, quality report, and export eligibility all update live from the same data.

### 3. What-If Boundary Explorer + Impact Simulator *(clearer)*

Moving a grade boundary changes real students’ grades.

- Preview a proposed boundary change **before** applying it.
- See the before/after grade distribution and the exact students who move.
- Apply the change, or cancel with no side effects.
- **Undo** the last applied What-If change.
- An impact banner summarises how many students were affected.

### 4. Final Review before export *(safer)*

- Confirmation dialog shows: data readiness, course stats, grade distribution, last impact, and export status.
- Export is **blocked with clear reasons** (not just a greyed-out button) when:
  - Instructor name is missing
  - BITS IDs are missing or duplicated in the course
  - Grade ranges are invalid
- Eligibility is checked **again** at the moment of export, so a UI bug cannot leak an unsafe CSV.

### 5. Try sample data *(easier to evaluate)*

One click loads a three-course demo set that intentionally contains four problems (blank mark, text mark, fractional mark, duplicate ID). Anyone can see the Quality Gate and editor working without preparing a file.

### 6. UX & accessibility polish

- Labelled inputs and ARIA live regions.
- Visible keyboard focus.
- Responsive layout.
- Clear empty-state messages.
- Proper CSV escaping and course-based filenames.
- Dropping a file on the page is ignored so the browser never navigates away.

> **Design decision:** A drag-and-drop upload zone was tried and **removed** — it duplicated the file button and added clutter without solving a real problem.

---

## Architecture notes

A few deliberate design choices keep the app correct under edge cases:

| Concern            | Approach                                                                                                |
|--------------------|---------------------------------------------------------------------------------------------------------|
| Valid marks        | Single function `hasValidMarks()` — only finite numbers in 0–100                                        |
| Grading rule       | Single pure function `assignGradeWithRanges()` used for real grades **and** What-If                     |
| Export safety      | Single function `getExportEligibility()` used by the button, the review screen, **and** the final CSV write |
| Data mutation      | Fractional marks are flagged, never auto-rounded; instructor chooses                                    |
| Session isolation  | Impact / What-If / undo state is reset on file change and course change                                 |

---

## Tech stack

| Area             | Choice                                                          |
|------------------|-----------------------------------------------------------------|
| Language         | Vanilla HTML, CSS, JavaScript — one self-contained `index.html` |
| Excel read/write | [SheetJS](https://sheetjs.com/) (`xlsx`), fully inlined so the app works offline |
| Charts           | HTML5 Canvas (histogram + bell curve)                           |
| Undo             | JSON snapshots of the full dataset                              |
| Hosting          | Static site (GitHub Pages) — no build step                      |

**Why no framework?**  
The brief asks for something anyone can open with nothing to install. Vanilla JS keeps the app dependency-free, fast, and easy for evaluators to review.

---

## How to run

```bash
# No install, no build
git clone https://github.com/Mihir-Bhagat/bits-grading-console.git
cd bits-grading-console
# Open index.html in Chrome / Edge / Firefox / Safari
```

### Deploy (GitHub Pages)

1. Push `index.html` and `README.md` to a public repository.
2. **Settings → Pages → Source:** Deploy from a branch → `main` / `/ (root)`.
3. Live URL: `https://<username>.github.io/bits-grading-console/`

---

## Testing performed

- Clean file → readiness **Ready**, export allowed
- Messy file (blank / text / fractional / duplicate) → issues reported; fix in editor → banner turns green → export
- Two sequential uploads
- Re-selecting the same file
- Empty course, single student, identical marks, large class
- Invalid / discontinuous grade ranges
- Missing instructor name at export time
- CSV with course names containing commas

---

## Limitations & possible next steps

- Marks are assumed on a **0–100** scale (as specified in the brief).
- Only the **first sheet** of a workbook is read.
- **Future ideas:** configurable grade schemes, multi-course batch export, session save/resume.

---

## AI tools used

**Claude (Anthropic)** — used for code review, implementing features, and drafting documentation.  
All output was reviewed, tested, and owned by me.

---

## Submission (CodeForge form)

| Field              | Value                                                                                                                             |
|--------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Public URL**     | https://mihir-bhagat.github.io/bits-grading-console/                                                                              |
| **GitHub repository** | https://github.com/Mihir-Bhagat/bits-grading-console                                                                           |
| **Top 3 enhancements** | 1. Data Quality Gate & Report · 2. Inline Student Records editor · 3. What-If Boundary Explorer + Impact Simulator + Final Review |
| **Bug Fix Log**    | Separate PDF/DOC in the required table format                                                                                     |

---

*Built for BITS Pilani Digital CodeForge V1.0.*  
*This is a challenge prototype — not an official grading tool.*
