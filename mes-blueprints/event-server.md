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
