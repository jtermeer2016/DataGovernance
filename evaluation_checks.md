# Evaluation Checks — Carmel Data Definitions Manager

This document defines the test cases and pass/fail criteria for validating the application before deployment and after any code change. Organized by subsystem.

---

## 1. Landing & Navigation

| # | Check | Pass Criteria |
|---|---|---|
| 1.1 | Landing screen loads | All 3 cards (Governance, Department, Search) render with correct labels and pills |
| 1.2 | Click Governance card | App shell appears, blue stripe, "🏛️ Data Governance" badge, Editor + Report tabs visible, Search tab hidden |
| 1.3 | Click Department card | App shell appears, green stripe, "📊 Dept. Report Definitions" badge, Editor + Report tabs visible, Search tab hidden |
| 1.4 | Click Search card | App shell appears, purple stripe, "🔍 Read-Only Search" badge, only Search tab visible, Editor/Report tabs hidden |
| 1.5 | Click "Change Type" | Returns to landing screen, clears `rows`, resets `currentType` to null |

---

## 2. Data Governance — Static Data Load

| # | Check | Pass Criteria |
|---|---|---|
| 2.1 | Select Governance | 14 static rows load automatically without requiring SharePoint connection |
| 2.2 | Verify row count | Row count badge shows "14 rows" before any filtering |
| 2.3 | Verify Carmel Same? pre-checks | Rows for "5-Year Rent Growth CAGR", "Blended Weeks Concessions", "Capitalization" show checked box; all others unchecked |

---

## 3. Data Editor — CRUD Operations

| # | Check | Pass Criteria |
|---|---|---|
| 3.1 | Add Row | New blank row appears at bottom, status badge = "New", first input auto-focused |
| 3.2 | Edit a cell | On input, row status flips from "Live" to "Modified" (only if value differs from original) |
| 3.3 | Edit then revert to original value | Row status returns to "Live" (no false-positive dirty state) |
| 3.4 | Delete a row with `spId` | Row hidden from table, status set to `deleted` internally (not removed from array until save) |
| 3.5 | Delete a row without `spId` (unsaved new row) | Row removed from array entirely, no orphaned `deleted` status |
| 3.6 | Revert single row (↩ button) | Only visible on dirty rows; restores `_orig` values, status reverts to `live` or `new` correctly |
| 3.7 | Revert All | Removes all rows without `spId`; restores all `spId` rows to `_orig` and `live` status |

---

## 4. Duplicate Detection

| # | Check | Pass Criteria |
|---|---|---|
| 4.1 | Enter duplicate Measure name | Both rows flagged `_isDup = true`, red highlight, "⚠ Duplicate" badge replaces normal status |
| 4.2 | Alert banner | Appears above table listing duplicate name(s) and occurrence count |
| 4.3 | Resolve duplicate (rename one row) | `_isDup` clears on both rows once names no longer match; banner disappears if no duplicates remain |
| 4.4 | Case insensitivity | "Total Units" and "total units" are correctly detected as duplicates |
| 4.5 | Whitespace insensitivity | "Total Units" and "Total Units " (trailing space) are correctly detected as duplicates |
| 4.6 | Deleted rows excluded | A row marked `deleted` does not count toward duplicate detection for its former duplicate partner |

---

## 5. SharePoint Integration

| # | Check | Pass Criteria |
|---|---|---|
| 5.1 | Missing config on Load | Clicking "Load from SP" with empty Site/List/Token shows error toast, no network call attempted |
| 5.2 | Get Digest success | `requestDigest` populated, success toast shown |
| 5.3 | Get Digest failure (bad token) | Error toast shown, `requestDigest` remains empty string |
| 5.4 | Load filters by type | Only rows where `DefinitionType` matches current `SCHEMA[currentType].typeLabel` are returned |
| 5.5 | Save without digest | Clicking "Save to SharePoint" with no digest shows error toast and does not attempt writes |
| 5.6 | Save new row | POST to `/items` (no method override); on success, `spId` is set on the row and status becomes `live` |
| 5.7 | Save modified row | POST with `X-HTTP-Method: MERGE`; on success, status becomes `live`, `_orig` updated to new snapshot |
| 5.8 | Save deleted row | POST with `X-HTTP-Method: DELETE`; on success, row fully removed from local `rows` array |
| 5.9 | Partial failure | If some rows succeed and others fail, toast reflects mixed result (e.g. "3 saved, 1 failed"); failed rows retain `error` status for retry |

---

## 6. Excel Import / Export

| # | Check | Pass Criteria |
|---|---|---|
| 6.1 | Upload valid .xlsx | Sheet selector appears only if workbook has 2+ sheets; otherwise hidden |
| 6.2 | Import Governance-labeled columns | Columns like "Measure Name" map correctly to `Measure` field |
| 6.3 | Import Department-labeled columns | Columns like "Data Element" map correctly to the same `Measure` field |
| 6.4 | Imported rows default status | All imported rows have `_status: 'new'` |
| 6.5 | Import triggers duplicate check | If imported data contains a name matching an existing row, duplicate flagging fires immediately |
| 6.6 | Download Template | File downloads with correct type-specific headers, 1 sample row, and a second "Column Guide" sheet |
| 6.7 | Template filename | Matches pattern `Carmel_{TypeLabel}_Definitions_Template.xlsx` |

---

## 7. Create / Update Modal (Governance Only)

| # | Check | Pass Criteria |
|---|---|---|
| 7.1 | Modal hidden for Department/Search | `govActionBar` only visible when `currentType === 'gov'` |
| 7.2 | Open in Create mode | Form is empty, mode toggle shows "Create New" active |
| 7.3 | Open in Update mode | Search box for existing records appears; selecting one pre-fills all fields |
| 7.4 | Required field validation | Clicking Save with empty Measure or Definition shows error toast, does not save |
| 7.5 | Duplicate warning in modal | Typing a Measure name matching an existing (different) row shows inline warning banner |
| 7.6 | Update mode excludes self from dup check | Editing a row's own Measure name without changing it does not trigger a false duplicate warning |

---

## 8. AI Email Notification Agent

| # | Check | Pass Criteria |
|---|---|---|
| 8.1 | Generate without required fields | Clicking "Generate Email" with empty Measure/Definition shows error toast, no API call made |
| 8.2 | Successful generation | Preview area shows non-empty Subject and Body, "Generate Email" re-enabled after completion |
| 8.3 | Malformed AI response | If JSON parsing fails, error toast shown ("Email generation failed"), `emailSubject`/`emailBody` remain unset |
| 8.4 | Save without generating email | Definition still saves successfully; no email is sent (`sendOutlookEmail` short-circuits and returns true) |
| 8.5 | Save after generating email | Email is sent via Graph API after the SharePoint save completes; success/failure reflected in final toast |
| 8.6 | Recipient management | Default recipient present on modal open; can add valid emails (must contain "@"); can remove any recipient including defaults |
| 8.7 | Invalid recipient entry | Entering text without "@" and clicking "+ Add" shows error toast, does not add to list |
| 8.8 | No HTML/script injection in body | Generated email body must not be rendered as HTML in the DOM in a way that executes scripts (current implementation uses `.textContent`, not `.innerHTML`, for the preview — confirm this remains true after any UI changes) |

---

## 9. Data Dictionary Report Tab

| # | Check | Pass Criteria |
|---|---|---|
| 9.1 | Tab switch populates filters | Report Name, Page, Source dropdowns populate from current `rows` on every tab switch |
| 9.2 | Filter combination | Selecting multiple filters (Report + Same as Carmel) narrows results using AND logic |
| 9.3 | Search box | Matches against Measure, Definition, and IndustryDef fields (case-insensitive substring match) |
| 9.4 | URL detection | Fields starting with `http://` or `https://` render as clickable 🔗 links; all others render as plain tags |
| 9.5 | Print | Clicking "Print Report" invokes `window.print()` without JS errors |

---

## 10. End User Search (Read-Only)

| # | Check | Pass Criteria |
|---|---|---|
| 10.1 | No write controls present | No Add/Edit/Delete/Save buttons exist anywhere in the `tab-search` DOM subtree |
| 10.2 | Keyword highlighting | Matching search term is wrapped in `<mark>` tags across Measure, Definition, and Industry Definition fields |
| 10.3 | Empty state | Searching a term with no matches shows the "No results found" empty state, hides result cards |
| 10.4 | Filter reset | "Clear filters" button resets search box and all 3 dropdowns, re-renders full unfiltered list |
| 10.5 | Special character safety | Searching a string containing regex special characters (e.g. `.*+?`) does not throw a JS error (verify regex escaping in `renderSearchView`) |
| 10.6 | XSS safety | All rendered fields pass through `esc()` before highlighting; no raw HTML from data is ever injected unescaped |

---

## 11. Cross-Cutting / Regression Checks

| # | Check | Pass Criteria |
|---|---|---|
| 11.1 | No console errors on load | Browser console is clean on initial page load and on every tab/type switch |
| 11.2 | No console errors on full CRUD cycle | Add → Edit → Save → Delete → Save produces no uncaught exceptions |
| 11.3 | State isolation between types | Switching from Governance to Department to Search and back does not leak rows between types (each `selectType` call fully resets `rows`) |
| 11.4 | Responsive table scroll | Data tables scroll horizontally without breaking page layout on narrow viewports |
| 11.5 | No localStorage/sessionStorage usage | Confirm no browser storage APIs are used anywhere in the codebase (in-memory state only, per environment constraints) |

---

## Sign-Off Checklist

Before merging any change to this application, confirm:

- [ ] All checks in sections 1–11 pass manually
- [ ] No new console errors introduced
- [ ] SharePoint field names unchanged unless list schema is also updated
- [ ] `AGENT_INSTRUCTIONS.md` updated if agent/email behavior changes
- [ ] `README.md` updated if SharePoint column requirements change
