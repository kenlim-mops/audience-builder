# Audience Builder — for the PMM team

Build HubSpot audiences by describing them in plain language, instead of filing a MOPS ticket and waiting.

## What it is

A tool you talk to. You describe the audience you want — "everyone who got the Serverless launch email and is active in Pods," "trial users who haven't spent in 30 days" — and it builds that list in HubSpot for you. You already know the audience you want; this just gets it built without the back-and-forth.

## Why it's useful

- **No ticket, no queue.** You go from idea to a ready-to-use HubSpot list in one conversation.
- **You see the size before it's built.** It tells you how many people match so you're not guessing.
- **It won't let you build something unsendable.** Opted-out and non-marketable contacts are filtered out automatically, every time.
- **Lists are named and tracked** so they don't pile up as mystery lists no one remembers creating.

## How it works — a real example

> **You:** Build an audience of trial users who opened the Serverless launch email and haven't spent in the last 30 days.
>
> **Audience Builder:** That's ~4,200 marketable contacts — trial users who opened *Serverless Launch — May* and have $0 spend in the last 30 days (opted-out and non-marketable contacts excluded). Create this as a list named `pmm_serverless-launch_dormant-30d`? [Yes / No]
>
> **You:** Yes.
>
> **Audience Builder:** Done — here's the list. [link]

Four steps, every time: you describe it → it maps and sizes it → you confirm → it creates the list. **Nothing is created until you say yes.**

## What you can segment on

- **Email behavior** — who received / opened / clicked a specific email or campaign, and how recently
- **Product usage** — spend (7/30/90 days, by product line), product health, Pods/Serverless engagement, use case
- **Lifecycle & intent** — lifecycle stage, PQL score and tier, lead status
- **Firmographic** — country/region, industry, tier

If you ask for something it can't target yet, it tells you and suggests the closest option — so let us know what's missing.

## What it does *not* do

- It doesn't send email or build the campaign — it produces the **audience** only.
- It doesn't change any contact data.

## How you'll use it

You'll invoke it wherever we land on (likely Slack). We'll confirm the exact "how to start it" once we pick the surface with you.

## We want your input

This is a first version. The two things that would help most:
1. **Your 5–10 most-requested audiences** — those shape what it can target out of the box.
2. **Any audience you build today that's genuinely complex** (multi-step, time-based cohorts) — those may need a heavier path and we'd rather know now.

Full spec (for the curious): *Self-Serve Audience Builder for PMM — Spec.*
