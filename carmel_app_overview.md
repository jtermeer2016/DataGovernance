# Carmel Data Definitions Manager
### Web Application Overview & Requirements Document
**Prepared for:** Carmel Partners — Data Governance Team
**Last Updated:** June 2026

---

## 1. Purpose

The Carmel Data Definitions Manager is a centralized, browser-based web application designed to create, manage, maintain, and publish standardized data definitions across the Carmel Partners organization. It serves as the single source of truth for data elements used in enterprise reporting, Power BI dashboards, and departmental analytics — replacing fragmented spreadsheets and disconnected documentation with a unified, governed platform.

---

## 2. Business Problem

Carmel Partners maintains data definitions across multiple sources including Excel workbooks, Power BI reports, and SharePoint lists. Without a centralized tool, data governance teams and department users face:

- Inconsistent definitions across reports and teams
- No visibility into whether Carmel's definition aligns with the industry standard
- No structured process for creating, updating, or communicating definition changes
- No self-service lookup for end users who need to understand what a measure means

---

## 3. User Types & Access Levels

| User Type | Access | Entry Point |
|---|---|---|
| **Data Governance Team** | Full create, edit, delete, import, export, and SharePoint sync | "Data Governance" card |
| **Department Report Users** | Full create, edit, delete, import, export, and SharePoint sync for department-scoped records | "Department Report Definitions" card |
| **End Users (Read-Only)** | Search and view definitions only — no add, edit, or delete capabilities | "Search Data Elements" card |

---

## 4. Core Features

### 4.1 Landing Screen
- Role-based entry point with three distinct cards
- Visual differentiation by color: navy (Governance), green (Department), purple (Search)
- Branding: Carmel Partners logo and color scheme throughout

### 4.2 Data Governance Module
- Manage enterprise-wide, canonical data definitions
- Restricted to the Data Governance team
- Dedicated **Create New** and **Update Existing** actions via a structured modal form
- All changes trigger an **AI-generated notification email** sent via Microsoft 365 / Outlook to a configurable recipient list (default team address + ability to add recipients per action)
- Pre-loaded with 14 static data elements sourced from the Carmel Data Dictionary

### 4.3 Department Report Definitions Module
- Manage data elements tied to departmental reports and dashboards
- Same editor interface as Governance with department-specific column labels
- Separate SharePoint list scope via `DefinitionType` field

### 4.4 Unified Data Schema
Both modules share a common field structure to ensure alignment:

| Unified Field | Governance Label | Department Label |
|---|---|---|
| Link | Link | Link |
| Sort | Sort | Sort |
| Report Name | Report Name | Report Name |
| Page / Location | Page Name | Page / Location |
| Measure / Data Element | Measure Name | Data Element |
| Industry Definition | Industry Definition | Industry Definition |
| Definition | Definition | Data Definition |
| Calculation | Calculation | Measure / Calc |
| Source | Source | Source / Link |
| Carmel Same? | Carmel Same? | Carmel Same? |
| Type *(system field)* | Governance | Department |

### 4.5 Data Editor
- Inline editable table with per-cell inputs and textareas
- Row-level status tracking: Live, New, Modified, Saving, Error
- Duplicate detection: real-time flagging when the same measure name is entered more than once, with a banner alert and row highlighting
- Revert individual rows or all changes at once
- Delete rows (soft-delete for SharePoint-backed records)

### 4.6 Excel Import & Export
- Drag-and-drop or click-to-upload `.xlsx` / `.xls` files
- Multi-sheet workbook support with sheet selector
- Intelligent column mapping supports both Governance and Department field naming conventions
- **Export Template**: downloads a pre-formatted `.xlsx` file with correct headers, a sample row, and a "Column Guide" instruction sheet — tailored to the selected module type

### 4.7 SharePoint Integration
- Connects to Carmel Partners SharePoint via REST API
- Loads existing list items filtered by `DefinitionType` (Governance or Department)
- Supports create, update (MERGE), and delete operations
- Request Digest authentication flow for authorized writes
- Row-by-row progress tracking with success/error status per record

### 4.8 Data Dictionary Report Tab
- Read-only tabular report view of all loaded definitions
- Filters by Report Name, Page / Location, Source, and "Carmel Same?" status
- Full-text search across measures and definitions
- Clickable 🔗 links for URL-based sources and page locations
- Printable layout
- Disclaimer footer with data refresh policy language

### 4.9 End User Search (Read-Only)
- Dedicated search experience for all organization users
- Keyword search highlights matched terms across measure names, definitions, industry definitions, calculations, and sources
- Filter by Report, Source, and Carmel Same? status
- Card-based results layout showing:
  - Measure name with link icon
  - Industry Definition vs. Carmel Definition side-by-side
  - Calculation in a code-style block
  - Report, Source, and Page tags
  - "Same as Industry" badge where applicable
- No edit, add, delete, or save controls visible

### 4.10 AI-Powered Email Notifications (Governance Only)
- Triggered on Create or Update actions in the Governance module
- Uses Claude AI (Anthropic API) to generate a professional, context-aware notification email based on the definition details
- Email preview shown before sending
- Sent via Microsoft Graph API (Microsoft 365 / Outlook)
- Default recipient list configurable; additional recipients can be added per notification

---

## 5. Technology Stack

| Component | Technology |
|---|---|
| Frontend | Single-page HTML / CSS / Vanilla JavaScript |
| Excel Parsing | SheetJS (XLSX.js) via CDN |
| SharePoint Integration | SharePoint REST API (v1) |
| Email | Microsoft Graph API (`/me/sendMail`) |
| AI Email Generation | Anthropic Claude API (claude-sonnet-4) |
| Hosting | Browser-based artifact / deployable as static HTML |

---

## 6. Data Governance & Quality Controls

- **Duplicate Detection**: Flags matching measure names in real time during manual entry and on Excel import
- **Industry Definition Alignment**: "Carmel Same?" checkbox on every record to track when Carmel's definition matches the industry standard
- **Type Segregation**: All records include a `DefinitionType` field to keep Governance and Department data distinguishable within a shared SharePoint list
- **Audit Trail via Notifications**: Every Governance create/update generates a dated email notification, creating a lightweight audit trail of changes
- **Template-Enforced Structure**: Downloadable Excel templates enforce correct column structure before import

---

## 7. SharePoint List Requirements

To fully connect the application, the SharePoint list should include the following columns:

| Column Name (Internal) | Type |
|---|---|
| `Title` | Single line of text (maps to Measure Name) |
| `Link` | Single line of text (URL) |
| `Sort` | Number |
| `Report_x0020_Name` | Single line of text |
| `Page_x0020_Name` | Single line of text |
| `Measure_x0020_Name` | Single line of text |
| `Industry_x0020_Definition` | Multiple lines of text |
| `Definition` | Multiple lines of text |
| `Calculation` | Multiple lines of text |
| `Source` | Single line of text |
| `CarmelSame` | Yes/No (Boolean) |
| `DefinitionType` | Single line of text (`Governance` or `Department`) |

---

## 8. Future Considerations

- Role-based authentication (Azure AD / SSO) to enforce access tiers without manual selection
- Version history per data element to track definition changes over time
- Approval workflow for new/updated Governance definitions before publishing
- Promotion flow: elevate a Department definition to a Governance definition
- Integration with Power BI to surface definitions inline within reports
- Expanded static data load from the full Carmel Data Dictionary workbook
