# UTM Builder v2 connection (audience ↔ UTM)

Purpose: tie audiences to the UTMs used to reach them, so we can report **audience performance** by joining UTM/web analytics back to an audience.

## The key
The Notion registry's `Audience ID` (`AUD-#`) is the join key. UTM Builder v2 stores this id on the UTM/campaign record (the two-way link), and the registry stores the UTM campaign IDs + links on the audience. Either side can be traversed.

## Two-way link
- **Registry → UTM:** store the UTM campaign IDs and full UTM links in the audience entry (`UTM campaign IDs`, `UTM links`).
- **UTM → registry:** UTM Builder v2 holds the `AUD-#` in an optional `audienceId` field **on the campaign** (`rpc_<ULID>`), so every issued link inherits its campaign's audience via join — the right granularity for audience-performance reporting. Delivered by PR #1 (`feat/audience-link`) on `kenlim-mops/utm_builder_v2`: Drizzle migration `0006_add_campaign_audience_id`, `audienceId` on `campaignInputSchema` (validated `^AUD-\d+$`, nullable), returned on campaign GETs, plus an optional "Audience ID (AUD-#)" form/detail field. Until that PR is merged and deployed, do the **one-way** record (registry only) and note the pending two-way link in your reply.

To set it once merged: set the campaign's `audienceId` to this audience's `AUD-#` via the campaigns API (create or PATCH `/api/v1/campaigns`).

## How to link during a build (step B8)
Only when the audience is tied to a campaign that uses UTM-tagged links:
1. Get/confirm the UTM campaign IDs and links for that campaign (issued via the UTM Builder — same source `campaign-copilot` uses to stamp destination URLs).
2. Record them in the registry entry.
3. If the two-way field is live, set the UTM record's audience reference to this audience's `AUD-#` via the UTM Builder v2 API.

Keep linkage optional — many audiences won't have UTMs at creation time; they can be linked later by editing the registry entry.
