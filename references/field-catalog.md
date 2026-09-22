# Curated field catalog — CONTACT (portal 24119306)

The vetted set PMM can segment on. Only target fields listed here. For enum/status fields, **resolve current valid options live** with `get_properties` (objectType `CONTACT`) before composing a filter — the options below are guidance, not a frozen list. If a requested criterion isn't here, surface the gap and offer the closest vetted alternative.

## Compliance (ALWAYS applied to email audiences — not optional)

| Field | Rule |
| --- | --- |
| `hs_marketable_status` | must be marketable (`true`) |
| `hs_email_optout` | must NOT be opted out of all email |
| `hs_emailconfirmationstatus` | prefer confirmed / not unsubscribed-bounced where the campaign requires it |

Intersect every email audience with: marketable **AND** not opted-out (**AND** confirmed when required). Keep this branch explicit in the readback.

## Email behavior (native, auto-maintained)

| Field | Meaning / use |
| --- | --- |
| `hs_email_last_email_name` | Name of the last marketing email received — "got email X" |
| `hs_email_open`, `hs_email_click`, `hs_email_delivered`, `hs_email_replied` | Lifetime engagement counts |
| `hs_email_last_open_date`, `hs_email_last_click_date`, `hs_email_last_send_date`, `hs_email_last_reply_date` | Engagement recency |
| `hs_email_sends_since_last_engagement` | Disengagement / sunset targeting |

For "was sent / opened / clicked **a specific marketing email or campaign**", prefer HubSpot's native list criteria on the email/campaign (see `hubspot-lists.md`) rather than a contact property — it's more precise than the aggregate fields above.

## Product usage (synced from Snowflake — Runpod's differentiator)

| Field | Meaning / use |
| --- | --- |
| `product_health_score` | Composite engagement score (higher = stronger) |
| `last_product_activity_date` | Recency of any product activity |
| `spend_7_days`, `spend_30_days`, `spend_90_days` | Total spend windows (USD) |
| `spend_*_sls`, `spend_*_ns` | Spend by product line (Serverless / non-serverless) |
| `first_spend`, `lifetimegpucloud` | First spend date / lifetime GPU cloud spend |
| `pods_engagement_status` | Pods engagement stage (enum — resolve live) |
| `serverless_engagement_status` | Serverless engagement stage (enum — resolve live) |
| `runpod_s_products` | Which Runpod products the contact uses (enum — resolve live) |
| `primary_use_case`, `use_case` | Workload / use-case targeting (enum — resolve live) |
| `gpus_in_use_*`, `gpu_quantity_in_use_*` | GPU types / quantities in use |

## Scoring & lifecycle

| Field | Meaning / use |
| --- | --- |
| `pql_score`, `pql_threshold` (A1–B2), `pql_primary_reason` | Product-qualified-lead scoring |
| `mql_type` | Marketing / Product / Partner qualified |
| `lifecyclestage` | Lifecycle stage (enum — resolve live) |
| `hs_lead_status` | Sales/outreach status (enum — resolve live) |

## Firmographic / behavioral

| Field | Meaning / use |
| --- | --- |
| `country`, `state`, `region`, `hs_country_region_code` | Geo |
| `industry`, `market_tier` | Firmographic |
| `zi_region` | ZoomInfo region |
| `sl_last_demo_cta` | Storylane demo CTA clicked |
| `num_conversion_events` | Number of form submissions |
| `last_webinar_registered_date` | Webinar registration |
| `page_visited_before_signup`, `hs_analytics_last_url` | Web behavior |

## Common audience recipes (starting points)

- **"Got email X and active in Pods"** → native "was sent email X" criterion `AND` `pods_engagement_status` = active-ish value (resolve live) `AND` compliance.
- **"Trial users, no spend in 30 days"** → `first_spend` is known `AND` `spend_30_days` = 0 (or `last_product_activity_date` older than 30d) `AND` compliance.
- **"High-intent Serverless leads"** → `pql_threshold` in {A1,A2} `AND` `serverless_engagement_status` active `AND` compliance.
- **"Re-engage disengaged"** → `hs_email_sends_since_last_engagement` >= N `AND` compliance (and consider excluding recent sends).
