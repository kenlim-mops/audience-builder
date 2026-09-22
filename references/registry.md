# HubSpot Audience Registry (Notion) — the source of truth

The registry is the informational source so anyone can understand what audiences exist, who made them, their definition, where they've been used, and their linked UTMs — **without opening HubSpot or the HubSpot MCP**.

- **Database:** "HubSpot Audience Registry" under *Marketing Home / Runpod MOPS Hub*
- **Database URL:** https://app.notion.com/p/37c68a47c0f1424cafd870a9a8f06918
- **Data source (collection) ID:** `bdae8ac8-9285-4798-85c6-aa228b35648d`
  (use as `data_source_id` when creating entries, and as the target of `query_data_sources` for lookups)

## Field map

| Column | Type | Notes |
| --- | --- | --- |
| `Name` | title | Audience name, e.g. `pmm_serverless-launch_dormant-30d` |
| `Audience ID` | auto-increment (`AUD-#`) | **read-only** — assigned by Notion. This is the key UTM Builder v2 references. |
| `Status` | select | `Active` / `Archived` / `Proposed` |
| `List type` | select | `Dynamic (active)` / `Static` |
| `Description` | text | Plain-English who-is-in-this |
| `Criteria` | text | Filter logic, including the compliance branch |
| `Requestor` | text | Name/email of the invoker (free text — PMM need not be a Notion user) |
| `Owner` | person | Notion user responsible for upkeep (optional) |
| `Products` | multi-select | `Pods`,`Serverless`,`Cross-product`,`Lifecycle`,`Event`,`Paid media`,`Other` |
| `Count at creation` | number | Live count at creation time |
| `Compliance applied` | checkbox | `__YES__` when the compliance intersection was applied |
| `HubSpot List ID` | text | |
| `HubSpot List URL` | url | |
| `UTM campaign IDs` | text | UTM Builder v2 campaign IDs (comma-separated) |
| `UTM links` | text | Full UTM-stamped destination URLs |
| `Linear ticket` | url | Auto-created MOP audit ticket |
| `Source` | select | `PMM self-serve (skill)` / `MOPS manual` / `Migrated` |
| `Where used / performance` | text | Campaigns/sends used in + performance notes |
| `Last used` | date | |
| `Created` / `Last updated` | timestamps | **read-only** |

## Lookups (LOOKUP mode)
Use `query_data_sources` against `bdae8ac8-9285-4798-85c6-aa228b35648d`. Filter/scan on `Name`, `Products`, `Requestor`, `Status`, `Criteria`. For "overlap", match on `Products` + keyword overlap in `Criteria`/`Description`.

## Creating an entry (BUILD step B7)
Use `notion-create-pages` with `parent: { data_source_id: "bdae8ac8-9285-4798-85c6-aa228b35648d" }`. Property value formats:
- Checkbox: `"Compliance applied": "__YES__"`
- Multi-select: `"Products": ["Serverless","Lifecycle"]`
- Date: use expanded keys — `"date:Last used:start": "2026-09-21"`, `"date:Last used:is_datetime": 0`
- Do **not** set `Audience ID`, `Created`, `Last updated` (read-only). Read `Audience ID` back after creation to get the `AUD-#`.

After creating, fetch the new page to capture the assigned `AUD-#` and the page URL (for the Linear ticket + confirmation reply). Then, once the Linear audit ticket exists, update the entry's `Linear ticket` field with its URL.
