# MOP audit ticket (Linear)

An auto-created, auto-closed Linear ticket that serves as the audit record for each **newly created** audience. Never created for lookups or overlap checks.

## Resolve the team + states
- Team: **MOP** (Mops). Resolve with `list_teams` (match key/name `MOP`) to get the team ID; cache it for the session.
- States: use `list_issue_statuses` for the MOP team to find a `completed`-type state (e.g. "Done"). You need it to close the ticket after success.

## Create (step B9)
Use `save_issue` (create) on the MOP team:
- **Title:** `[Audience] <audience name>` (e.g. `[Audience] pmm_serverless-launch_dormant-30d`)
- **Description (markdown):**
  - **Requestor:** <name/email of the invoker>
  - **Audience (plain English):** …
  - **Criteria:** … (include the compliance branch)
  - **Count at creation:** …
  - **HubSpot list:** <url>
  - **Registry entry:** <Notion page url> (AUD-#)
  - **UTMs:** <campaign IDs / links, or "none">
- **Assignee:** the requestor if they're a Linear user; otherwise leave unassigned for MOPS.

## Close after success
Once the HubSpot list **and** the registry entry both exist, transition the ticket to the MOP `Done` (completed) state via `save_issue` (update with the completed state ID). If either upstream step failed, do NOT close — leave it open and report the failure.

## Write back
Put the ticket URL into the registry entry's `Linear ticket` field, and include it in the final confirmation reply to the user.

## Guardrails
- New successfully-created audiences only.
- Requestor is the invoker of the skill, not the automation identity — capture it explicitly.
- Auto-close is intentional (audit trail). If a run is configured to leave tickets open for review, follow that configuration instead; default is create + close.
