# Xcelsior Agent Skill

Teach any AI agent (Claude, Cursor, Codex, …) to drive [Xcelsior](https://xcelsior.ca)
GPU compute through the official TypeScript SDK — no MCP client required.

## Install

```bash
npx skills add xcelsior-gpu/skill
```

Then set your access token (from <https://xcelsior.ca/dashboard/mcp>). Quick Connect
uses `XCELSIOR_ACCESS_TOKEN`. The SDK README names the same bearer value
`XCELSIOR_API_TOKEN`. The skill reads whichever is set.

```bash
export XCELSIOR_ACCESS_TOKEN="xcel_ai_..."
```

The agent installs [`@xcelsior-gpu/sdk`](https://www.npmjs.com/package/@xcelsior-gpu/sdk)
and can then browse the marketplace, check spot prices, estimate cost in CAD, launch
and monitor instances, and check your wallet balance — all in natural language.
Call shapes live in [`SKILL.md`](./SKILL.md).

## Why the SDK rather than raw HTTP

The SDK is typed, exposes real error classes (`XcelsiorApiError`,
`XcelsiorApiTimeoutError`), and is generated from the same OpenAPI spec that backs the
public API — so an agent using it cannot drift from the wire format the way hand-written
requests do.

## Prefer MCP?

Xcelsior also ships a hosted MCP server at `https://xcelsior.ca/mcp`. See the
dashboard's **Connect** page for a one-paste setup with your token embedded.

Full API reference: <https://docs.xcelsior.ca>
