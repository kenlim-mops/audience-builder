---
name: audience-builder
description: Self-serve HubSpot audience/segment builder AND registry lookup for the Product Marketing (PMM) team. Two modes. (1) LOOKUP — answer "does an audience already exist", "who created it", "how did it perform / where was it used", "does my idea overlap with an existing audience" by reading the Notion HubSpot Audience Registry, WITHOUT opening HubSpot or the HubSpot MCP. (2) BUILD — turn a plain-language audience description into a HubSpot list: check for overlap first, map to a curated field catalog, auto-apply compliance filters (marketable + not opted-out + confirmed), show a live count, and (only on explicit confirmation) create the list, save its full definition to the Notion registry, link any UTM Builder v2 campaigns, and file an auto-closed MOP audit ticket. Use whenever someone wants to build, create, define, size, find, or report on an audience/segment/list — e.g. "build an audience of…", "create a list of…", "do we already have an audience for…", "who made the X list", "how did the Serverless launch audience perform", "trial users who haven't spent in 30 days". Runpod HubSpot portal 24119306. Produces the audience only — it never sends email or edits contacts.
---

# Audience Builder + Registry (PMM self-serve)

Two modes:
- **LOOKUP** — answer questions about existing audiences from the Notion **HubSpot Audience Registry** (`references/registry.md`). No HubSpot needed.
- **BUILD** — create a new HubSpot audience/list, record it, link UTMs, and file an audit ticket.

You produce the **audience only** — you never send email, build a campaign, or edit contact data. Portal: **Runpod**, HubSpot `24119306`.

## First step, always: classify intent
- If the user is **asking about** audiences (exists? who made it? how did it perform? where used? overlap?) → **LOOKUP mode**. Do not create anything.
- If the user wants a **new/changed** audience ("build", "create", "give me a list of") → **BUILD mode**.
When unsure, ask one clarifying question.

## Golden rules (do not skip)
1. **Never create a list without explicit confirmation** of the readback + count.
2. **Always run the overlap check** (step B1) before building — surface existing matches so we don't duplicate.
3. **Always apply the compliance intersection** to email audiences: `marketable + not opted-out + confirmed` (`references/field-catalog.md` → Compliance). Automatic, non-negotiable.
4. **Only target vetted catalog fields**; resolve enum values live with `get_properties` (never hardcode).
5. **Every created audience is recorded in the Notion registry** — no orphan lists. The registry is the source of truth; HubSpot is just where the list physically lives.
6. **Name and audit every list** (see Naming; `AUD-#` + Linear ticket).

---

## LOOKUP mode

Read the registry (`references/registry.md` has the data source ID and field map) and answer directly:
- **Existence / ownership:** query by name, keyword, or Products; report matches with Requestor, Description, Status, Created date, HubSpot link.
- **Performance / where used:** report the "Where used / performance" field, linked UTM campaigns/links, and Last used. If deeper email metrics are needed, note you can pull them from HubSpot (`get_marketing_email_analytics`) or UTM/web analytics on request — but answer from the registry first.
- **Overlap:** given a described idea, list registry audiences with overlapping Products/criteria and explain the overlap so PMM can decide to reuse vs. build new.

Prefer the registry over HubSpot. Only reach into the HubSpot MCP if the registry can't answer and the user wants live data.

---

## BUILD mode

### B1. Overlap check (mandatory, before anything else)
Query the registry for existing audiences with similar name/Products/criteria. If a close match exists, surface it (name, AUD-#, requestor, definition, HubSpot link) and ask: **reuse it, refine it, or build new anyway?** Only proceed to build once the user chooses.

### B2. Understand + map
Restate the audience and its AND/OR logic. Map each criterion to a vetted field in `references/field-catalog.md`; for enum/status fields call `get_properties` (objectType `CONTACT`) for current valid values. Surface any criterion with no vetted field and offer the closest alternative.

### B3. Compose + apply compliance
Build the filter, then intersect with the compliance branch (marketable + not opted-out + confirmed). Keep that branch explicit in the readback.

### B4. Size it (live count) — before creating
Count matches with `search_crm_objects` (objectType `CONTACT`, your filter groups, minimal properties, read the total). Flag very large / all-marketable audiences and recommend MOPS review before creating.

### B5. Readback + confirmation gate
Show the PMM: plain-English description, exact criteria (incl. compliance), live count, proposed name (`AUD` will be assigned by the registry), any overlap found, whether a UTM/campaign will be linked, and a note that **a MOP audit ticket will be created and auto-closed**. Then ask: **"Create this audience? [Yes / No]"** Do not proceed until yes.

### B6. Create the list in HubSpot
- **v1 / now:** static list via MCP `manage_segment` (`CreateStaticSegmentOperation` + batched `AddMembersOperation`). Set its `confirmationStatus: CONFIRMED` only after the PMM approved in B5.
- **Preferred / when wired:** dynamic active list via HubSpot Lists REST API (`references/hubspot-lists.md`).
Capture the HubSpot list ID + URL.

### B7. Record it in the Notion registry (mandatory)
Create a registry entry with every known field — Name, Description (plain English), Criteria (incl. compliance branch), Requestor (the invoker), Products, Count at creation, List type, HubSpot List ID + URL, Source = `PMM self-serve (skill)`, Status = `Active`, Compliance applied = yes. Capture the assigned **AUD-#**. See `references/registry.md` for the exact data source ID and column mapping.

### B8. Link UTMs (if the audience is tied to a campaign)
If the audience will be used with UTM-tagged links:
- Record the UTM campaign IDs and full UTM links in the registry entry (`UTM campaign IDs`, `UTM links`).
- Tag those UTMs back to this audience by writing the `AUD-#` into UTM Builder v2's audience reference field (the two-way link). See `references/utm-builder.md`. If that field isn't available yet in the deployed app, do the one-way record (registry only) and note the pending two-way link.

### B9. File the MOP audit ticket (new audiences only — never on lookups)
Create a Linear issue in the **MOP** team:
- Title: `[Audience] <audience name>`
- Requestor: the person who invoked the skill (state it in the description; set assignee to the requestor if they're a Linear user, else leave for MOPS).
- Description: plain-English audience + full criteria (incl. compliance) + live count + HubSpot list link + registry entry link (AUD-#) + any UTM campaign IDs/links.
- After the list + registry entry succeed, **transition the ticket to a completed/closed state (Done)** — it's an audit record, not active work.
- Write the ticket URL back into the registry entry's `Linear ticket` field.
See `references/linear-audit.md` for team/state resolution.

### B10. Confirm back
Reply with: the registry entry link (AUD-#), the HubSpot list link, the Linear ticket link, and a one-line restatement of the audience + count.

## Naming
`pmm_<theme>_<qualifier>` lowercase-kebab, e.g. `pmm_serverless-launch_dormant-30d`. Theme = campaign/purpose, qualifier = distinguishing filter. The registry assigns the stable `AUD-#` id.

## Advanced cohorts (path B)
For logic HubSpot lists can't express (cohort math, multi-touch, cross-object aggregates), resolve contact IDs in Snowflake (where `spend_*`, `product_health_score`, `last_product_activity_date`, `*_engagement_status` originate), push a static list, and still do B5–B10 (confirmation, registry, UTM, audit ticket).

## What NOT to do
- Don't send email, enroll in workflows, or build/modify campaigns.
- Don't edit/enrich/delete contact properties.
- Don't create a list that skips the compliance intersection or the registry entry.
- Don't file a Linear ticket for lookups or overlap checks — only for a successfully created new audience.
- Don't silently target a field outside the catalog.
