# Carmel Data Definitions Manager — Agent Instructions

This document describes the behavior, configuration, and operating instructions for the Carmel Data Definitions Manager application and its embedded AI agent capability (email notification generation).

---

## 1. Application Roles

The app has three entry points selected from the landing screen. Each loads a different `currentType` state (`gov`, `dept`, or `search`) which drives which UI elements, SharePoint filters, and permissions are active.

| Type | Variable | Permissions |
|---|---|---|
| Data Governance | `gov` | Full CRUD, Create/Update modal, AI email notifications |
| Department Reports | `dept` | Full CRUD, no AI email notifications |
| Search Data Elements | `search` | Read-only, no SharePoint writes, no modal access |

**Rule:** The `search` type must never expose `addNewRow`, `deleteRow`, `saveChanges`, or modal functions in its UI. These functions remain defined in code but are not wired to any visible control when `currentType === 'search'`.

---

## 2. Data Model

All records share one schema regardless of type, distinguished by a `DefinitionType` field (`"Governance"` or `"Department"`) when persisted to SharePoint.

```
{
  id, spId, Link, Sort, ReportName, PageLoc,
  Measure, IndustryDef, Definition, Calculation,
  Source, CarmelSame, _status, _orig, _isDup
}
```

- `_status`: one of `live`, `new`, `dirty`, `saving`, `error`, `deleted`
- `_orig`: snapshot used to detect whether a row has unsaved changes
- `_isDup`: computed flag, true when `Measure` (case-insensitive, trimmed) matches another active row

---

## 3. Duplicate Detection Logic

Run via `checkDuplicates()` after any row is added, imported, or has its `Measure` field edited.

**Algorithm:**
1. Filter to rows where `_status !== 'deleted'`
2. Build a map of lowercase, trimmed `Measure` values to row IDs
3. Any key with more than one row ID is a duplicate group
4. Mark all rows in duplicate groups with `_isDup = true`
5. Render a dismissible alert banner listing each duplicated name and occurrence count

**Instruction to any agent modifying this app:** never silently de-duplicate or delete rows automatically. Duplicate detection is advisory only — a human must resolve duplicates manually.

---

## 4. SharePoint Integration Instructions

### Authentication Flow
1. User provides Site URL, List Name, and Bearer Token
2. Agent/user clicks **Get Digest** → calls `/_api/contextinfo` to retrieve `X-RequestDigest`
3. Digest is required before any write (create/update/delete) operation
4. Digest is **not** required for read (`Load from SP`) operations

### Required List Columns
See `README.md` for the full column-to-type mapping. The internal field `DefinitionType` **must** exist as a text column or all records will collapse into a single unfiltered view.

### Write Operation Rules
- **Create**: POST to `/items` with no `X-HTTP-Method` override
- **Update**: POST to `/items(<id>)` with `X-HTTP-Method: MERGE`
- **Delete**: POST to `/items(<id>)` with `X-HTTP-Method: DELETE`
- All writes must include `IF-MATCH: *` to bypass version conflict checks (acceptable here since the app does not support concurrent multi-user editing)

---

## 5. AI Email Notification Agent

This is the only true "agent" behavior in the app — an LLM call that drafts a notification email based on structured form data.

### Trigger Conditions
- Only available when `currentType === 'gov'`
- Only triggered manually via the **✨ Generate Email** button inside the Create/Update modal
- Never triggered automatically on save — generation and sending are explicit user actions

### Prompt Contract
The agent is instructed to:
1. Act as a "data governance communications specialist at Carmel Partners"
2. State whether the action is a create or update
3. Summarize the measure name, definition, and (if present) industry definition comparison
4. Note any required reviewer action
5. Close professionally
6. **Return strictly valid JSON**: `{"subject": "...", "body": "..."}` — no preamble, no markdown fences

### Evaluation Checks for the Email Agent
See `EVALUATION_CHECKS.md` for the full pass/fail criteria. At minimum, any response must:
- Parse as valid JSON after stripping code fences
- Contain non-empty `subject` and `body` string fields
- Not fabricate data not present in the form (no hallucinated metrics, dates, or people)
- Avoid including raw HTML or script tags in the body (defense against injection if email body is later rendered as HTML)

### Failure Handling
If the API call fails or returns unparseable JSON:
- Show a toast: "Email generation failed."
- **Do not block the save action** — the definition must still save to SharePoint/local state even if the email step fails entirely
- Saving and email-sending are intentionally decoupled operations

### Sending
- Performed via Microsoft Graph `POST /me/sendMail`
- Recipients = default list (`datagovernance@carmelpartners.com`) + any manually added addresses
- If no email was generated (`emailSubject` is empty), `sendOutlookEmail()` short-circuits and returns `true` without making a network call — saving never waits on an email that was never drafted

---

## 6. Excel Import / Export Instructions

### Import
- Accepts `.xlsx` / `.xls` only
- Must support multiple header naming conventions (Governance vs. Department column labels) by checking both possible header strings per field (see `loadSheet()` mapping)
- Imported rows always start with `_status: 'new'` — never silently overwrite existing live rows on import

### Export Template
- Must always reflect the **currently selected type's** column labels and example values
- Must include a second sheet (`Column Guide`) with column name, description, required flag, and example — this is non-optional documentation embedded in the file itself

---

## 7. Safe-Editing Instructions for Future Agents/Developers

If another AI agent or developer is asked to modify this codebase, it should:

1. **Never remove the duplicate detection check** — it is a data quality safeguard, not a cosmetic feature
2. **Never auto-send emails without explicit user action** — generation and sending must remain two distinct, user-initiated steps
3. **Never expose write/edit controls in the `search` type view** — this is a hard permission boundary, not just a UI convenience
4. **Always preserve the `_orig` snapshot pattern** when adding new editable fields, so revert/undo functionality continues to work
5. **Always update both `SCHEMA.gov.cols` and `SCHEMA.dept.cols`** when adding a new field — the two types must stay structurally aligned even though labels differ
6. **Test SharePoint write payloads** against the internal field names (`Report_x0020_Name`, etc.) — these encode spaces and must not be changed casually, as SharePoint internal names are fixed at list-column creation time
