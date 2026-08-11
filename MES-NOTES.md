# MES fork notes

This is a fork of [barryyip0625/mcp-discord](https://github.com/barryyip0625/mcp-discord),
maintained under `MindsEyeSociety/mcp-discord` for driving the setup and
management of MES Discord servers via the Model Context Protocol.

## Baseline

- Forked from upstream release **v1.4.1** (2026-07-13).
- Work happens on the **`mes-main`** branch, branched directly from the `v1.4.1`
  tag (not from upstream `main`, which carries unreleased commits).
- `main` is left as an untouched mirror of upstream `main` for comparison/PR-back
  purposes.

Built from source rather than installed via `npx mcp-discord` / npm — the npm
`latest` dist-tag is stale at `1.3.4` (July 2025) and is missing the tools and
fixes added in 1.4.0/1.4.1.

## Build

```
npm install
npm run build   # tsc -> build/
npm test        # Jest, --runInBand
```

`build/` is gitignored; it is not committed. Baseline on `mes-main`: 17 test
suites / 361 tests passing.

## Known upstream issues relevant to us

- **#41** — HTTP transport (`app.ts`) fails on a second client `initialize`
  call (single shared `McpServer` instance). We use **stdio** transport only.
- **#32** — `discord_read_messages` can advertise an empty `inputSchema` over
  the HTTP path due to a `ZodEffects` wrapper from `.refine()`. Not hit under
  stdio.
- **#43** — `discord_read_messages` does not surface attachment metadata
  (id/name/url/contentType).
- README's "minimum required permissions" invite link (bitfield
  `52076489808`) is missing `Manage Roles`, `Read Message History`, and
  `Send Messages in Threads` despite the prose above it listing them. The
  corrected minimum bitfield is `327222897744`. We use Administrator for
  build-out, so this doesn't block us, but it's a candidate upstream PR.
- README recommends enabling the **Presence** intent; the code
  (`src/index.ts`) never requests `GatewayIntentBits.GuildPresences`. We leave
  it off.

## Local patch log

None yet. Patches are added reactively — see the main build-out plan for the
process (touches `src/schemas.ts`, `src/tools/<area>.ts`, `src/tools/tools.ts`,
`src/server.ts`, `src/toolList.ts`). Log each patch here with the date and the
tool it adds/changes.

## Secrets

`DISCORD_TOKEN` lives in `.env` (gitignored), loaded via `dotenv`. Never
commit it, paste it into chat, or add it to `.claude.json`.

## Architect bot

Discord application "MES Server Architect", Client ID `1536579898934689802`.
Private app (Public Bot off, Install Link set to None on the Installation
tab) — not discoverable or self-installable by anyone else.

Invite link (Administrator, `bot` scope only — not a secret, but only use it
to add the bot to servers you control, since it grants Administrator on
whatever guild it's used in):

```
https://discord.com/oauth2/authorize?client_id=1536579898934689802&permissions=8&integration_type=0&scope=bot
```

Per the build-out plan: invite to the existing server for the audit (Phase 3),
remove Administrator there once the audit export is written, invite to the
new event server for build-out (Phase 4), then kick the bot from both once
build-out is signed off (Phase 7).
