# Xcelsior Agent Skill

Teach any AI agent (Claude, Cursor, Codex, …) to drive [Xcelsior](https://xcelsior.ca)
GPU compute directly through its public REST API.

## Install

```bash
npx skills add xcelsior-gpu/skill
```

Then set your access token (from <https://xcelsior.ca/dashboard/mcp>):

```bash
export XCELSIOR_ACCESS_TOKEN="xcl_..."
```

The agent can now list GPUs, check spot pricing, estimate cost, launch and monitor
instances, and check billing — all in natural language.

Prefer MCP? Xcelsior also ships a hosted MCP server at `https://xcelsior.ca/mcp` —
see the dashboard's **Connect** page.

Full API reference: <https://docs.xcelsior.ca>
