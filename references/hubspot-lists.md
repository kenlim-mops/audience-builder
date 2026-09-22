# HubSpot list mechanics

## Static vs dynamic — how to choose

| | Static list | Dynamic (active) list |
| --- | --- | --- |
| Membership | Fixed at creation (a snapshot) | Auto-updates as contacts match/unmatch |
| Best for | Point-in-time cohorts; externally-computed audiences (path B) | Anything expressible in HubSpot filter logic (default) |
| How to create here | HubSpot MCP `manage_segment` (`CreateStaticSegmentOperation` + `AddMembersOperation`) | HubSpot **Lists REST API** `POST /crm/v3/lists` with `processingType: "DYNAMIC"` |

Default to **dynamic** when the criteria are native HubSpot filters (they stay fresh for the whole campaign). Use **static** for snapshots or when the audience was computed in Snowflake.

## v1 path — static via MCP (available today)

The HubSpot MCP `manage_segment` tool creates **static** lists only:
1. `CreateStaticSegmentOperation` — name + objectType `CONTACT` (optionally pre-populate with member IDs).
2. `AddMembersOperation` — add the matched contact IDs in batches.

`manage_segment` requires its own confirmation. Set `confirmationStatus: CONFIRMED` **only after** the PMM approved the readback + count in the skill's confirmation gate. Never set CONFIRMED preemptively.

To get the member IDs and the count, run `search_crm_objects` (objectType `CONTACT`) with the composed filter groups (including the compliance branch) and read the total + IDs.

## Preferred path — dynamic via Lists REST API (main build task)

Not exposed through the current MCP. Requires a HubSpot private-app token with `crm.lists.write`.

- Endpoint: `POST /crm/v3/lists`
- Body (shape): `{ "name": "...", "objectTypeId": "0-1", "processingType": "DYNAMIC", "filterBranch": { ...filter definition... } }`
- The `filterBranch` encodes the AND/OR criteria and the native "was sent/opened/clicked email or campaign" filters.
- Reference: HubSpot Lists API docs (v3) — `developers.hubspot.com` → CRM → Lists.

Wiring this token + the filterBranch compiler is the primary engineering task to move from static (v1) to dynamic (preferred).

## Native email/campaign criteria

For "received / opened / clicked a specific marketing email or campaign", use HubSpot's native list filters on the marketing email or campaign object rather than the aggregate contact properties — they are per-asset and precise. These are available in dynamic-list `filterBranch` definitions.
