# Changelog

All notable changes to the Carmel Data Definitions Manager are documented in this file.

## [1.0.0] — Initial Release

### Added
- Landing screen with role-based entry: Data Governance, Department Report Definitions, Search Data Elements
- Inline data editor with add/edit/delete, row status tracking (Live, New, Modified, Saving, Error)
- SharePoint REST API integration: load, create, update (MERGE), and delete operations with request digest authentication
- Excel import via SheetJS with support for both Governance and Department column naming conventions
- Excel template export with a second "Column Guide" instruction sheet
- Data Dictionary Report tab with filters (Report Name, Page/Location, Source, Carmel Same?) and full-text search
- Carmel Partners logo and branded color scheme across all views

### Added — Data Quality
- Real-time duplicate detection on Measure/Data Element names, with row highlighting and a dismissible alert banner
- Industry Definition field added alongside Carmel's own Definition for side-by-side comparison
- "Carmel Same?" checkbox to track whether Carmel's definition matches the industry standard
- Link column (first column) supporting direct URLs to source records, rendered as a 🔗 icon when populated

### Added — Data Governance Module
- Dedicated Create New / Update Existing modal restricted to the Governance type
- Searchable "Update Existing" lookup to find and pre-fill an existing record
- AI-generated notification emails (via Anthropic Claude API) drafted from form data on create/update
- Email preview (subject + body) before sending
- Configurable recipient list with a default governance address, plus ability to add/remove recipients per notification
- Email delivery via Microsoft Graph API (Microsoft 365 / Outlook)
- Save and email-send are decoupled — a failed or skipped email never blocks saving the definition

### Added — Search Data Elements Module
- Fully read-only search experience for general end users
- Keyword search across Measure, Definition, Industry Definition, Calculation, and Source fields
- Search term highlighting in results
- Filters for Report, Source, and Carmel Same? status
- Card-based result layout with tags for Report, Source, and Page/Location
- No add, edit, delete, or save controls present anywhere in this view

### Added — Static Data
- Pre-loaded 14 Data Governance records sourced from the existing Carmel Data Dictionary Power BI report, available immediately without requiring a SharePoint connection

### Fixed
- Resolved `SyntaxError: Invalid or unexpected token` and `ReferenceError: selectType is not defined` caused by fragmented incremental script updates; consolidated into a single clean rewrite of the application script

### Documentation
- Added `README.md` with setup instructions and SharePoint list schema reference
- Added `AGENT_INSTRUCTIONS.md` describing agent behavior rules for the AI email feature and safe-editing guidelines for future contributors
- Added `EVALUATION_CHECKS.md` with a full manual test plan across 11 subsystem categories
- Added application overview document covering purpose, user types, features, tech stack, and future considerations

---

## [Unreleased] — Under Consideration

- Role-based authentication (Azure AD / SSO) to enforce access tiers automatically
- Version history per data element
- Approval workflow for new/updated Governance definitions
- Promotion flow: Department definition → Governance definition
- Inline Power BI integration to surface definitions within reports
- Full static data load from the complete Carmel Data Dictionary workbook
