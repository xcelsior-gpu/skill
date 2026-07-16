---
name: xcelsior-gpu
description: Rent and manage GPU compute on Xcelsior directly from your agent — list available GPUs, check spot pricing, estimate cost, launch and monitor instances, and check your wallet balance via the REST API.
---

# Xcelsior GPU Compute

Xcelsior is a GPU cloud where you rent verified GPUs by the hour (or run serverless
inference) and pay in CAD. This skill lets an agent drive the platform's public REST API.

## Setup

1. Get a Bearer token: open <https://xcelsior.ca/dashboard/mcp> (the token is shown
   ready to copy) or Settings → API & SSH. Alternatively use an existing
   `$XCELSIOR_ACCESS_TOKEN`.
2. Export it so requests can authenticate:
   ```bash
   export XCELSIOR_ACCESS_TOKEN="xcl_..."      # required
   export XCELSIOR_API_URL="https://xcelsior.ca"  # optional; defaults to prod
   ```
3. Every request sends `Authorization: Bearer $XCELSIOR_ACCESS_TOKEN`.

Base URL: `https://xcelsior.ca` (override with `$XCELSIOR_API_URL` for staging/local).

## Core operations

### List available GPUs (marketplace)
```bash
curl -s "$XCELSIOR_API_URL/marketplace?active_only=true" \
  -H "Authorization: Bearer $XCELSIOR_ACCESS_TOKEN"
```

### Check live spot prices
```bash
curl -s "$XCELSIOR_API_URL/api/v2/marketplace/spot-prices" \
  -H "Authorization: Bearer $XCELSIOR_ACCESS_TOKEN"
```

### Estimate cost before launching (always do this first)
```bash
curl -s -X POST "$XCELSIOR_API_URL/api/pricing/estimate" \
  -H "Authorization: Bearer $XCELSIOR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"gpu_model":"RTX 4090","duration_hours":2,"spot":true}'
```

### Launch an instance
```bash
curl -s -X POST "$XCELSIOR_API_URL/instance" \
  -H "Authorization: Bearer $XCELSIOR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"my-training-run","gpu_model":"RTX 4090","num_gpus":1,"pricing_mode":"spot","interactive":true}'
```
Returns a `job_id`. Only `name` is strictly required, but you almost always want
`gpu_model`, `num_gpus`, and `pricing_mode` (`spot` | `on_demand`).

### Watch / list / stop instances
```bash
curl -s "$XCELSIOR_API_URL/instances" -H "Authorization: Bearer $XCELSIOR_ACCESS_TOKEN"
curl -s "$XCELSIOR_API_URL/instance/JOB_ID" -H "Authorization: Bearer $XCELSIOR_ACCESS_TOKEN"
curl -s "$XCELSIOR_API_URL/instance/JOB_ID/logs" -H "Authorization: Bearer $XCELSIOR_ACCESS_TOKEN"
curl -s -X POST "$XCELSIOR_API_URL/instances/JOB_ID/cancel" -H "Authorization: Bearer $XCELSIOR_ACCESS_TOKEN"
```

### Check wallet balance
```bash
curl -s "$XCELSIOR_API_URL/api/billing/wallet/CUSTOMER_ID" \
  -H "Authorization: Bearer $XCELSIOR_ACCESS_TOKEN"
```

### List GPU tiers
```bash
curl -s "$XCELSIOR_API_URL/tiers" -H "Authorization: Bearer $XCELSIOR_ACCESS_TOKEN"
```

## Guardrails

- **Estimate cost first.** Call `/api/pricing/estimate` and confirm with the user
  before launching anything priced above ~$5/hr.
- **Prefer spot** (`pricing_mode: "spot"`) unless the user needs guaranteed uptime —
  it's typically ~60% cheaper.
- **Stop what you start.** Cancel idle instances; the user is billed per second while
  they run.
- Never print or commit the access token. Treat `$XCELSIOR_ACCESS_TOKEN` as a secret.

## Full reference

Interactive API docs (every endpoint, request/response schemas, more examples):
<https://docs.xcelsior.ca>
