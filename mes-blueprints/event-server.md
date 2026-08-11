# Blueprint — MES: MESCon Prime 2026

Target guild: **MES - MESCon Prime 2026**, ID `1536582623139201164`.
Applied via the `discord` MCP server (Architect bot). This is the versioned
source of truth — if the live server diverges from this file, prefer editing
this file and re-applying over hand-editing in the Discord UI, so drift stays
visible.

Access model: **members-only, gated lobby** — `@everyone` sees only the
Lobby; verified MES members additionally see Members; ticket-holders
additionally see Event. Informed by the [existing-server audit](existing-server-audit.md),
which confirmed Con Discordia already runs this exact mesbot pattern at a
larger scale — role names and channel purposes below map directly onto ones
already proven there.

## Role hierarchy (create top → bottom; order is load-bearing)

A bot can only manage roles positioned *below* its own highest role, so
creation and position-fixup must land in this order, topmost first.

| # | Role | Color | Hoist | Permissions | Notes |
|---|---|---|---|---|---|
| 1 | `MES Server Architect` | — | no | Administrator | Already exists (managed/bot role, auto-created on invite). Confirmed at the top already since it's currently the only non-default role. **Removed after Phase 6 sign-off.** |
| 2 | `MESBot` | — | no | Manage Roles, View Channels, Send Messages, Read Message History, Ban Members | Already exists once mesbot is invited (Phase 5) — a managed/bot role, not created by us. Must land directly below Architect and above every role it manages. |
| 3 | `Event Staff` | `#9b59b6` (matches Con Discordia's `Event Lead` purple) | yes | Manage Messages, Manage Threads, Move Members, View Channels, Send Messages, Read Message History | Organizers. |
| 4 | `Moderator` | `#e67e22` | yes | Manage Messages, Kick Members, Timeout Members, View Channels, Send Messages, Read Message History | |
| 5 | `Attendee` | `#00b0ff` (matches Con Discordia's `Paid 2026`) | yes | none (access is via category overwrites, not role permissions) | **mesbot-owned** — granted only via `!setevent`. Never hand-assign. |
| 6 | `Member` | `#2ecc71` | no | none | **mesbot-owned** — granted only via portal OAuth. Never hand-assign. |
| — | `@everyone` | — | — | View Channel **denied** at guild default | Everyone lands with no channel visibility except what Lobby's category overwrite grants back. |

Apply order:
1. Confirm `MES Server Architect` is at the top (already true — see audit).
2. `discord_create_role` for `Event Staff`, `Moderator`, `Attendee`, `Member` — in that order, top to bottom.
3. `discord_edit_role` on each (and on `@everyone` via guild default) to fix final positions if creation order didn't land them correctly — verify with `discord_list_roles` after.
4. mesbot's own `MESBot` role appears automatically when it's invited in Phase 5 — **at that point, re-check positions** and drag `MESBot` above `Attendee`/`Member` in the Discord UI if it doesn't land there automatically (mesbot's role assignment silently fails otherwise, per `README.md`'s own setup instructions).

## Channels

Category-level overwrites carry the access model; channels inherit and get
no per-channel overwrites unless noted.

### Category: Lobby
Visible to `@everyone`.
- `#welcome` (text, read-only for `@everyone` — deny Send Messages) — rules + what the event is + how to verify. Copy below.
- `#verify` (text, `@everyone` can view and send) — mesbot's configured verification channel (`!setver`). Where newcomers run `!auth`.

### Category: Members
Visible to `Member` only (and above).
- `#announcements` (staff-post only — deny Send Messages for `Member`)
- `#general`
- `#introductions`

### Category: Event
Visible to `Attendee` only (and above).
- `#event-announcements` (staff-post only)
- `#event-general`
- `#schedule`
- `#help-desk` — staffed support channel; the Risk #1 mitigation (mistyped membership numbers → missing Attendee role) routes here.
- Voice/game rooms — **TBD, needs Phase 0 event details** (game format not yet specified).

### Category: Staff
Visible to `Event Staff` + `Moderator` only.
- `#staff-chat`
- `#bot-log` — mesbot's configured logging channel (`!setlog`).

### Category-level permission overwrites

| Category | `@everyone` | `Member` | `Attendee` | `Event Staff` / `Moderator` |
|---|---|---|---|---|
| Lobby | Allow View Channel | (inherits) | (inherits) | (inherits) |
| Members | Deny View Channel | Allow View Channel | (inherits from Member since Attendee is only granted alongside Member) | Allow View Channel |
| Event | Deny View Channel | Deny View Channel | Allow View Channel | Allow View Channel |
| Staff | Deny View Channel | Deny View Channel | Deny View Channel | Allow View Channel |

Apply order: `discord_create_category` × 4, then `discord_set_channel_permissions`
on each category (not each channel), then create channels with `parentId`
set so they inherit.

## Reused copy for `#welcome`

Drawing directly on the [existing-server audit](existing-server-audit.md) —
this copy is already proven at Con Discordia, not invented fresh:

```
=======================================
WELCOME TO MESCON PRIME 2026!
=======================================

This server is for verified MES members and event attendees only.

**Getting verified:**
Head to #verify and send `!auth`. MESBot will DM you a sign-in link — it
expires after 5 minutes, so use it promptly. Log in with your MES portal
account and click Authorize. You'll be verified within about a minute.

**Getting your ticket role:**
Sign up for your ticket at [Zeffy link — TBD]. You'll get the Attendee role
automatically — it checks every 60 seconds, keyed on your MES membership
number, so make sure you enter it correctly in this format: US2019080055.
If you've paid and the role hasn't shown up after a few minutes, double-
check the number you entered, or ask in #help-desk.

**A few rules:**
- Do not DM staff — questions and issues go through #help-desk or #verify.
  DMs get missed and aren't logged. This is enforced.
- Be excellent to each other — see the MES Code of Conduct for the full
  policy: [link to governing docs].
```

Placeholders (`[Zeffy link — TBD]`, `[link to governing docs]`) get filled
once Phase 0's event details (portal event ID, Zeffy ticketing link) are
confirmed — needed for Phase 5 (`!setevent`) regardless.

## Deliberately not built via MCP

No tool exists for any of these — set by hand in the Discord UI once the
channel/role structure above is in place: guild icon/name, Community
features, rules screening, AFK/system channel, default notification level,
scheduled events, invites, emoji.

## Open items before this can be fully applied

- **Portal event ID** for `!setevent` (Phase 5) and the **Zeffy ticket
  link** for the `#welcome` copy — still needed from you (Phase 0 item 1).
  Not blocking role/channel creation, only the final copy and mesbot config.
- **Event voice/game room list** — depends on the event's actual format,
  not yet specified.

## As applied (2026-08-11)

Roles, categories, and channels above are live. Role creation landed the
hierarchy correctly on the first pass — Discord inserted each new role
directly below the previous one, so no `discord_edit_role` position fixup
was needed:

| Role | ID | Position |
|---|---|---|
| `MES Server Architect` | `1536582840790032415` | 5 (top) |
| `Event Staff` | `1536587020648849448` | 4 |
| `Moderator` | `1536587041343672411` | 3 |
| `Attendee` | `1536587065104269372` | 2 |
| `Member` | `1536587149477158963` | 1 |
| `@everyone` | `1536582623139201164` | 0 |

`MESBot`'s own role does not exist yet — appears in Phase 5.

| Category | ID |
|---|---|
| Lobby | `1536587477161213973` |
| Members | `1536587491769843762` |
| Event | `1536587524544143504` |
| Staff | `1536587543926145044` |

All 11 planned text channels created under their categories (IDs omitted
here — see `discord_get_server_info` for the live list). `#welcome`'s
copy is posted with `[Zeffy link — TBD]` and `[link to governing docs —
TBD]` placeholders still open, per the item above.

**Deviation from the original design:** rather than denying `ViewChannel`
on `@everyone`'s guild-wide base permissions, access control was
implemented entirely through **category-level overwrites** — `@everyone`
gets an explicit `ViewChannel` deny on Members/Event/Staff (Lobby is left
untouched, visible by guild default), with explicit allows layered on top
for the roles that should see each category. This avoids having to also
re-grant `SendMessages`/etc. everywhere, since the guild-wide baseline
permissions were never touched. Three additional **channel-level**
`SendMessages` denies make the three "staff-post-only" channels read-only:
`@everyone` on `#welcome`, `Member` on `#announcements`, `Attendee` on
`#event-announcements`.

**Cleanup:** Discord's auto-created default channels/categories (`Text
Channels` → `#general`, `Voice Channels` → voice `General`) were deleted —
they weren't part of the blueprint and the duplicate `#general` name was
confusing next to the Members-category one.

**Not yet verified:** the permission-overwrite matrix itself — `mcp-discord`
has no read-back tool (confirmed in the existing-server audit), so the 14
`discord_set_channel_permissions` calls above are confirmed only by their
individual "success" responses, not by re-reading the applied state. Spot-
check in the Discord UI, or build the reactive `discord_get_channel_permissions`
patch flagged in `MES-NOTES.md`, before relying on this for the event.

## Phase 5 — mesbot onboarding (2026-08-11)

mesbot was invited by hand (bots can't add other bots). Two gaps surfaced
that the original blueprint didn't anticipate, since mesbot's role didn't
exist yet when it was written:

1. **mesbot's role landed at position 1** — below `Attendee`/`Member`, same
   bottom-of-hierarchy behavior observed for the Architect bot on Con
   Discordia. Fixed with `discord_edit_role` (position 5, directly below
   `MES Server Architect`). Confirmed via `discord_list_roles`: hierarchy
   now reads exactly as designed — `MES Server Architect`(6) → `MESBot`(5)
   → `Event Staff`(4) → `Moderator`(3) → `Attendee`(2) → `Member`(1) →
   `@everyone`(0).
2. **mesbot's role had no visibility into the `Staff` category** — the
   category overwrites only granted `Event Staff`/`Moderator`, and mesbot
   needs to see and post in `#bot-log`. Added a `ViewChannel`+`SendMessages`
   allow overwrite for the `MESBot` role (`1536594921148907533`) on the
   `Staff` category. Add the same pattern for the `MESBot` role on any
   future category it needs to post into.

Since mesbot doesn't filter out bot-authored messages (only messages from
*itself*; see `on_message` in `main.py`) and its admin commands check only
`guild_permissions.administrator`, the Architect bot — which holds
Administrator — was able to drive configuration directly instead of
requiring commands to be typed by a human:

- `!role Member` → `Role found: Member (ID: 1536587149477158963)`
- `!setver <#1536588152565796934>` (sent in `#verify`) → `✅ Verification channel set to #verify`
- `!setlog` (sent with no args, directly in `#bot-log`, using the
  current-channel default) → `Logging channel set to #bot-log`

**`!setevent` is intentionally not yet run** — the portal event and its
Zeffy ticketing link don't exist yet. The `#welcome` copy's two `TBD`
placeholders are also still open, pending the same information. Run
`!setevent <portal_event_id> Attendee` and fill the placeholders once both
exist.

## Phase 4b — Venue categories and roles (as applied, 2026-08-11)

Three concurrent-game venues added: `LotN` (vampire), `Apoc` (werewolf),
`SPI`, each with a `Lead`/`Second`/`Staff` staff tier and a `Player`
visibility role. Full design in the plan file
(`i-want-to-setup-scalable-kurzweil.md`, Phase 4b) — this section records
what actually landed.

**Role hierarchy** — inserted between `Moderator` and the pre-existing
`Attendee`/`Member`, which had to move down to make room:

| Position | Role |
|---|---|
| 18 | `MES Server Architect` |
| 17 | `MESBot` |
| 16 | `Event Staff` |
| 15 | `Moderator` |
| 14–6 | `LotN Lead/Second/Staff`, `Apoc Lead/Second/Staff`, `SPI Lead/Second/Staff` (venue-grouped, in that order) |
| 5 | `Attendee` (moved down from 14) |
| 4–2 | `LotN Player`, `Apoc Player`, `SPI Player` |
| 1 | `Member` (moved down from 13) |
| 0 | `@everyone` |

**Bug caught mid-build:** the 9 `Lead`/`Second`/`Staff` roles were
initially created with their permissions (`ManageChannels`,
`ManageMessages`, etc.) as **guild-wide base permissions** on the role —
which would have let e.g. `LotN Lead` manage channels in *every* category,
not just their own. Caught before any category overwrites were applied;
fixed with 9 `discord_edit_role` calls setting `permissions: []`, so all
elevated permissions now come **only** from the category-scoped overwrites
below. Worth remembering for any future role that's meant to be
category-scoped: `discord_create_role`'s `permissions` param is always
guild-wide — scoped permissions can only be granted via
`discord_set_channel_permissions` on the specific category/channel.

**Category overwrites** (mirrors the Members/Event/Staff pattern — deny
`@everyone`, allow specific roles), applied identically to all three venue
categories (`LotN` `1536601540876304437`, `Apoc` `1536601542583517184`,
`SPI` `1536601543183179822`):

| Role | Allow |
|---|---|
| `@everyone` | — (deny `ViewChannel`) |
| `<Venue> Player` | `ViewChannel` |
| `<Venue> Lead` / `<Venue> Second` | `ViewChannel`, `ManageChannels` |
| `<Venue> Staff` | `ViewChannel`, `ManageMessages`, `ManageThreads`, `MuteMembers`, `DeafenMembers`, `MoveMembers` |
| `Event Staff` / `Moderator` | `ViewChannel` (oversight into every venue) |

`SendMessages`/`ReadMessageHistory` aren't set explicitly anywhere here —
same as the original Members/Event/Staff categories, they're inherited
from `@everyone`'s untouched guild-wide base permissions once `ViewChannel`
makes the category visible.

**Channels**: `#lotn-ooc`, `#lotn-questions`, `#apoc-ooc`,
`#apoc-questions`, `#spi-ooc`, `#spi-questions` — 2 per venue, minimal
starter set per the locked decision. `Lead`/`Second` are expected to build
out IC/game rooms themselves using their `ManageChannels` grant.
`#role-requests` created in the existing **Event** category (not a new
category) with a holding message explaining self-assignment is deferred —
no reaction-role bot in this stack yet; venue roles are assigned manually
by an admin/the Architect bot via `discord_assign_role` for now.

**Verified**: `discord_get_server_info` after all changes shows 7
categories / 18 text channels total, all under correct parents;
`discord_list_roles` confirms the 19-role hierarchy above exactly. Same
caveat as Phase 4 applies — the category overwrite matrix itself has no
programmatic read-back (`mcp-discord` gap), confirmed only by each
`discord_set_channel_permissions` call's individual success response.
