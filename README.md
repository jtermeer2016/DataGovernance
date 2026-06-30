# Carmel Data Definitions Manager

A browser-based tool for creating, managing, and searching standardized data definitions across Carmel Partners — built for Data Governance, Department Reporting teams, and general end users.

## Features

- **Three access modes**: Data Governance (full edit), Department Report Definitions (full edit), and Search Data Elements (read-only for all users)
- **Inline data editor** with live duplicate detection, row status tracking, and revert/undo
- **SharePoint integration** — load and save records via SharePoint REST API
- **Excel import/export** — upload `.xlsx` files or download a pre-formatted template with a column guide
- **AI-generated notification emails** — Governance changes trigger an AI-drafted email sent via Microsoft 365/Outlook
- **Data Dictionary Report** — filterable, printable report view of all definitions
- **Industry vs. Carmel definition comparison** with a "Carmel Same?" indicator

## Getting Started

1. Open `index.html` in any modern browser — no build step or server required
2. From the landing screen, select **Data Governance**, **Department Report Definitions**, or **Search Data Elements**
3. To connect live data, enter your SharePoint site URL, list name, and access token in the Data Editor tab, then click **Get Digest** followed by **Load from SP**

## Tech Stack

- Vanilla HTML / CSS / JavaScript (single file, no build process)
- [SheetJS](https://sheetjs.com/) for Excel import/export
- SharePoint REST API for data persistence
- Microsoft Graph API for email notifications
- Anthropic Claude API for AI-generated email content

## SharePoint List Setup

Your SharePoint list should include these columns:

| Column | Type |
|---|---|
| `Title` | Single line of text |
| `Link` | Single line of text |
| `Sort` | Number |
| `Report_x0020_Name` | Single line of text |
| `Page_x0020_Name` | Single line of text |
| `Measure_x0020_Name` | Single line of text |
| `Industry_x0020_Definition` | Multiple lines of text |
| `Definition` | Multiple lines of text |
| `Calculation` | Multiple lines of text |
| `Source` | Single line of text |
| `CarmelSame` | Yes/No |
| `DefinitionType` | Single line of text (`Governance` or `Department`) |

## Notes

- The Search Data Elements view is read-only by design — no add, edit, or delete controls are exposed
- Governance edits require a SharePoint request digest (click **Get Digest**) before saving
- Email notifications are optional — saving still works if email generation is skipped or fails

## License

Internal use — Carmel Partners.
