# Existing server audit — Con Discordia

Audited 2026-08-11 via the Architect bot (Administrator, temporary — see
[MES-NOTES.md](../MES-NOTES.md)). Source: `discord_get_server_info`,
`discord_list_roles`, `discord_read_messages` on the key info channels.
Guild ID `1104230635431809165`.

## Headline finding

**Con Discordia is not a generic MES hub — it's a prior run of exactly the
project we're building.** It's a full annual-convention server already
running mesbot in the same verify → Member, Zeffy-ticket → event-role
pattern from the build-out plan, for a multi-track LARP con (SPI, Apocalypse,
LotN venues). It's much larger and multi-year (channels are archived by year
rather than deleted), but the access-control skeleton is the same one
`mes-blueprints/event-server.md` will formalize for MESCon Prime 2026. Where
useful, copy is reused verbatim below rather than invented fresh.

## Server basics

| Field | Value |
|---|---|
| Name | Con Discordia |
| Owner | `478286724334616594` |
| Created | 2023-05-06 |
| Members | 115 |
| Channels | 212 total — 186 text, 6 voice, 20 categories, 0 forum |

## Role hierarchy (top → bottom, 62 roles total)

Only the roles relevant to the new server's design are listed; the full
export has the rest (mostly per-venue staff/faction roles — see table
below for the shape).

| Position | Role | Notes |
|---|---|---|
| 61 (top) | `MESBot` | Managed (bot) role. Sits at the very top — required so it can manage every role below it. |
| 60 | `Verified Members` | The mesbot-owned "Member" equivalent — matches our planned `Member` role. |
| 59 | `Helper.gg` | Managed (ticketing bot). |
| 58 | `Server Admin Staff` | Hoisted. |
| 57 | `Event Lead` | Hoisted. |
| 56 | `Event Payment Manager` | |
| 55 | `Paid 2026` | The mesbot-owned event-ticket role — matches our planned `Attendee` role. Renamed per-year (`Paid 2026`, presumably `Paid 2025` etc. before). |
| 54–43 | Per-venue lead/staff roles | `ST Lead(s)`, `LotN Leads/Staff`, `Apoc Leads/Staff`, `SPI Leads/Staff`, `Tech Help!`, `National Coordinator`, `National Storyteller`, `Board of Directors`, `Avengers Bots` |
| 42 | `Server Booster` | Managed (Discord built-in). |
| 41–36 | Ticketing/bot infra roles | `ticket-support`, `Reaction Roles`, `Dyno`, `Quark Logger`, `Welcomer`, `YAGPDB.xyz` — all managed/bot roles from Helper.gg-style ticket bots, a logger, a welcome-message bot, a reaction-role self-assign bot |
| 35–2 | Faction/venue/character-trait self-assign roles | e.g. `Apocalypse`, `LotN`, `Camarilla`, `Anarch`, `Cub`…`Elder` (werewolf ranks), various IC ability tags (`Medium`, `Oracular`, `Heightened Senses`…). Self-assigned via a reaction-role bot in `#role-requests`, per the `#important-rules` copy below. Out of scope for the new event server (single-track, no factions). |
| 1 | `MES Server Architect` | Us — see hierarchy note below. |
| 0 | `@everyone` | |

**Note:** `discord_list_roles` did not return a `permissions` bitfield in
this deployment's output (only id/name/color/position/hoist/mentionable/
managed/memberCount), despite upstream's schema advertising one. Worth
retesting once we're actively setting role permissions in Phase 4 — if it's
still absent, permission verification will need to go through
`discord_get_member` or manual UI checks instead.

**Role hierarchy note:** the Architect bot's role landed at position 1 here
— just above `@everyone`, i.e. the *bottom* of the non-default roles —
because Discord inserts a newly-added bot's role just above `@everyone` in
an already-populated hierarchy. On the new server (checked the same way)
Architect is currently the *only* non-default role, so it's trivially at the
top for now. **Re-verify this after Phase 4 starts creating roles** — if
Discord's insert behavior places new roles *above* Architect rather than
below, Architect will need to be manually dragged back to the top before it
can manage them.

## Channel structure

20 categories. Only the ones with a direct analogue in our blueprint are
detailed; the rest (12 categories: `SPI OOC/IC/Tickets`, `Apocalypse
OOC/IC/Tickets`, `Malfeas`, `LotN OOC/IC/Tickets`, `Q&A`, three year-archive
categories) are per-venue/historical and don't inform a single-event server.

| Category | Channels | Analogue in our blueprint |
|---|---|---|
| **Information** | `#announcements`, `#know-your-staff`, `#important-rules`, `#mes-policies`, `#arrivals`, `#identify-yourself` | → **Lobby**. `#arrivals` is mesbot's configured verification channel (`!setver`) — see copy below. `#important-rules`/`#mes-policies` → our `#welcome`. |
| **OOC Channels** | `#general-chat`, `#common-room` (voice), `#role-requests`, `#find-a-game`, `#questions-for-staff`, `#pictures-and-memes` | → **Members**. `#role-requests` is a reaction-role self-assign channel — not needed for us (single event role, mesbot-owned, not self-assign). |
| **Tickets for Tech and Event Leads/Coordinators**, **Tech and Support Tickets** | `#ticket-request` + an empty category | → our `#help-desk`. They use a ticketing bot (Helper.gg) for structured support tickets; we're not replicating the ticketing bot, just a staffed channel. |
| **Staff Operations** | `#discord-needs`, `#staff-voice`, `#staff-chat`, `#nst-hideout`, `#warnings` | → **Staff**. |
| **BOT CHANNNELS** | `#dyno-is-watching`, `#ticket-log`, `#logger-is-watching`, `#talk-to-bots`, `#tupper-is-tupping`, `#mesbot-logs` | `#mesbot-logs` is mesbot's configured logging channel (`!setlog`) → our `#bot-log`. The rest are logging for bots we're not installing. |

## Reused copy

Pulled verbatim/paraphrased from `#important-rules`, `#mes-policies`,
`#identify-yourself`, and `#announcements` for the new server's `#welcome`
and `#verify` channels:

- **No-DM-staff policy** (`#important-rules`): staff must never be DM'd —
  use the ticket/help channel instead, DMs get missed and aren't logged.
  First offense = warning, repeat = removal. Worth keeping verbatim; it's a
  real operational rule, not decorative.
- **Nickname convention**: members set their nickname to `First Last
  MES#` (or `First L MES#`) OOC; may switch to character name in-character.
  Not needed for a single-track event server with no IC/OOC split, but the
  *pattern* (nickname carries verification-adjacent info) is worth knowing
  about if the event ever goes in-character.
- **Verification copy** (`#identify-yourself`): explains `!auth`, that
  verification can take a few minutes, that multiple Discord accounts can
  link to one membership number but only one gets paid-participant access,
  and that DMs must be allowed. Good template for our `#verify` channel.
- **Ticket-purchase copy** (`#announcements`): "Sign up for your ticket at
  [Zeffy link] — you'll get the appropriate role automatically... it checks
  every 60 seconds... keyed on your membership number, make sure you enter
  it correctly in this format: `US2019080055`." **This is the single most
  useful piece of reused copy** — it's a live, battle-tested explanation of
  exactly the fragile membership-number matching called out as Risk #1 in
  the build-out plan. Reuse near-verbatim in the new server's `#welcome`.
- **Code of Conduct excerpt** (`#mes-policies`): pulled from the Membership
  Handbook pgs 29-31 — the five Code of Conduct points, the Zero Tolerance
  Policy, and the reporting contacts (`board@modernenigmasociety.org`,
  NDIA/PACT). This is governing-document material, not server-specific —
  link to it rather than re-pasting the full text, to avoid drift if the
  Handbook changes. See the `mes-governance` skill for the canonical source.

## Live confirmation of the mesbot pattern

`#arrivals` message history shows the exact flow the new server will use:
mesbot posts `Welcome @user Name (MembershipNumber) membership verified!` on
OAuth completion, and separately `🎉 @user now has the Paid 2026 role for
ConDiscordia 2026 — see you there!` when the Zeffy-ticket join lands. Both
land in the same channel — confirms `!setver` is meant to double as the
event-role announcement channel, not just the membership one (matches
`_announce_event_role` in `main.py`).

## Known gaps confirmed during this audit

- **No permission-overwrite data anywhere in `discord_get_server_info`** —
  confirmed by grepping the full raw payload for "permission"/"overwrite":
  zero matches. This is the gap flagged in `MES-NOTES.md`; the Phase 4
  verification pass will need to rely on the Discord UI or a reactive patch
  (`discord_get_channel_permissions`) rather than an MCP read-back.
- `discord_get_server_info`'s response is large enough (98KB / 3,434 lines
  for 212 channels) to exceed the tool-result token limit and get written to
  a file instead of returned inline — not a problem for the new server
  (small channel count), but worth knowing if this tool is ever pointed at
  Con Discordia again.

## Next step

Remove the Architect bot's Administrator permission from Con Discordia now
that this export is complete (per Phase 3.4 of the build-out plan) — it
should not retain standing access to the existing server while the new one
is being built.
