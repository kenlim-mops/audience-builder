# Audience Builder — Setup Guide (Claude)

How to install and run the `audience-builder` skill in Claude, and what it needs to be wired to. For what the tool does day-to-day, see [README.md](./README.md); for the design rationale, see the spec in Notion.

---

## 1. How a Claude skill works (30 seconds)

A skill is a folder containing `SKILL.md` (instructions Claude loads on demand) plus reference files. Claude auto-discovers skills placed in a skills directory and invokes this one when someone asks to build, find, or report on an audience. There is nothing to "run" — you install the folder and the connectors it depends on, and Claude does the rest conversationally.

---

## 2. Prerequisites (connectors + access)

The skill orchestrates existing systems through Claude connectors (MCP servers). All of these must be connected **and authorized** in the Claude environment where the skill runs, with the access levels below.

| System | Used for | Access needed |
| --- | --- | --- |
| **HubSpot** connector | Resolve properties, size audiences, create the list | Portal `24119306` (Runpod); read contacts/properties; write Lists (`manage_segment`; `crm.lists.write` for dynamic lists) |
| **Notion** connector | Read/write the Audience Registry | Access to *Marketing Home → Runpod MOPS Hub → HubSpot Audience Registry* (data source `bdae8ac8-9285-4798-85c6-aa228b35648d`) |
| **Linear** connector | File the auto-closed MOP audit ticket | Write to the **Mops** team |
| **UTM Builder v2** (HTTP API) | Two-way audience↔UTM link | Optional; only once PR #1 is merged/deployed. Until then the skill records UTMs one-way in the registry |

> The connectors run under the identity of whoever's Claude environment executes the skill. That identity needs the write permissions above. For team self-serve, configure these connectors centrally (see Option B) rather than per person.

---

## 3. Option A — Claude Code (single user)

For an individual (e.g. you, or one PMM on Claude Code desktop/CLI):

1. Clone the repo into your personal skills directory:
   ```bash
   git clone https://github.com/kenlim-mops/audience-builder.git ~/.claude/skills/audience-builder
   ```
   (Or a project-scoped location: `<repo>/.claude/skills/audience-builder`.)
2. Ensure the HubSpot, Notion, and Linear connectors are added and authorized in Claude Code (`/mcp` in an interactive session shows their status).
3. Start (or restart) Claude Code so the skill is discovered. It should appear as `audience-builder` in the skills list.
4. Verify (see §5).

To update later: `git -C ~/.claude/skills/audience-builder pull`.

---

## 4. Option B — Distribute to the PMM team (recommended)

The lowest-friction surface for a non-technical PMM team is **Claude in Slack**, where connectors are configured once at the workspace/org level and everyone invokes the skill by typing a request.

1. **Wire the connectors centrally** — have a workspace admin connect and authorize the HubSpot, Notion, and Linear connectors for the Claude workspace, with the access in §2. This means PMM never handles credentials.
2. **Make the skill available to the team** — publish `audience-builder` as an org/workspace skill (or package it as a plugin) so it loads for PMM users. Point the source at this repo so updates flow from `git`.
3. **Tell PMM how to invoke it** — they just describe the audience in a channel or DM (see README). No install on their side.
4. **Set the guardrail defaults** you want enforced (compliance intersection is always on; decide whether large/all-contacts audiences require MOPS review before creation).

> Exact publishing steps depend on your Claude workspace/admin settings; if you package it as a plugin, keep this repo as the plugin source so `git` remains the update path.

---

## 5. Verify the install

Run these in order; each should work without touching HubSpot directly:

1. **Lookup (read-only):** ask *"What audiences do we already have for Serverless?"* → the skill should query the Notion registry and answer (empty is fine on a fresh registry).
2. **Sizing (read-only):** ask *"How many marketable contacts opened the last Serverless email and have no spend in 30 days?"* → it maps to the catalog and returns a live count, without creating anything.
3. **Full build (writes):** ask *"Build that as an audience."* → confirm it (a) shows a readback + count, (b) waits for your explicit yes, (c) creates the list, (d) writes a registry row with an `AUD-#`, (e) files an auto-closed MOP ticket. Use a throwaway name and archive it afterward if this is just a test.

If step 3 stops at the confirmation gate and does nothing until you say "yes," the safety behavior is working as intended.

---

## 6. Configuration reference (already baked into the skill)

These values live in the skill's reference files — listed here so you know what to change if anything moves:

- **HubSpot portal:** `24119306` (`SKILL.md`)
- **Registry data source:** `bdae8ac8-9285-4798-85c6-aa228b35648d` (`references/registry.md`)
- **Linear team / audit behavior:** `Mops`, create + auto-close on new audiences only (`references/linear-audit.md`)
- **UTM link:** campaign-level `audienceId` = `AUD-#` (`references/utm-builder.md`)
- **Curated field catalog + naming convention:** `references/field-catalog.md`, `SKILL.md`

---

## 7. Troubleshooting

- **Skill doesn't trigger** → confirm the folder is under a skills directory and Claude was restarted; check the `description` in `SKILL.md` matches how people are phrasing requests.
- **"Can't read the registry"** → the Notion connector isn't authorized, or lacks access to the MOPS Hub database. Re-authorize / share the DB.
- **List creation fails** → HubSpot connector missing Lists write scope. v1 uses static lists (`manage_segment`); dynamic lists need `crm.lists.write` + the Lists REST API.
- **No MOP ticket created** → Linear connector not connected, or no write access to the Mops team. Ticket is only created on a *successful new build*, never on lookups.
- **UTM link not set** → expected until PR #1 (`utm_builder_v2#1`) is merged/deployed; the skill records UTMs one-way in the registry meanwhile.
