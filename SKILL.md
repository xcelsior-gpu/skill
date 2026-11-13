---
name: xcelsior-gpu
description: Rent and manage GPU compute on Xcelsior directly from your agent — browse the marketplace, check spot prices, estimate cost, launch and monitor instances, and check your wallet balance using the official TypeScript SDK.
---

# Xcelsior GPU Compute

Xcelsior is a GPU cloud where you rent verified GPUs by the hour (or run serverless
inference) and pay in CAD. This skill lets an agent drive the platform through the
official SDK — typed, with real error classes, rather than hand-rolled HTTP.

## Setup

1. Get an access token: open <https://xcelsior.ca/dashboard/mcp> (Quick Connect shows a
   token ready to copy) or go to Settings → API & SSH. Agent keys look like `xcel_ai_...`.
2. Export it. Quick Connect and the MCP CLI use `XCELSIOR_ACCESS_TOKEN`. The SDK
   README uses `XCELSIOR_API_TOKEN` for the same bearer value. Read whichever is set.
   ```bash
   export XCELSIOR_ACCESS_TOKEN="xcel_ai_..."
   ```
3. Install the SDK in the working directory:
   ```bash
   npm install @xcelsior-gpu/sdk
   ```

Construct a client once and reuse it:

```ts
import { XcelsiorApiClient } from "@xcelsior-gpu/sdk";

const token = process.env.XCELSIOR_ACCESS_TOKEN || process.env.XCELSIOR_API_TOKEN;
const client = new XcelsiorApiClient({ token });
```

The published client exposes `artifacts`, `auth`, `billing`, `inference`, `instances`,
`marketplace`, `marketplaceV2`, `serverless`, and `volumes`. Request arguments are
snake_case (`job_id`, `customer_id`, `gpu_model`), matching the API.

## Core operations

### Find a GPU

```ts
const listings = await client.marketplace.list();
const filtered = await client.marketplace.searchGet({
  gpu_model: "RTX 4090",
  min_vram: 24,
});
const spotListings = await client.marketplaceV2.search({
  gpu_model: "RTX 4090",
  spot_available: true,
});
const spot = await client.marketplaceV2.getSpotPrices();
const tiers = await client.instances.listTiers();
```

V1 filtered search is `marketplace.searchGet`. V2 search is `marketplaceV2.search`.
Spot history for one model is `marketplaceV2.getSpotHistory({ gpu_model })`. The CAD
reference table is `billing.referencePricing()`.

### Estimate cost before launching (always do this first)

```ts
const quote = await client.billing.estimateCost({
  gpu_model: "RTX 4090",
  duration_hours: 1,
  spot: true,
});
```

Pass the GPU, duration, and `spot` flag you intend to launch. An empty call uses the
spec default: RTX 4090, 1 hour, on demand.

### Launch an instance

```ts
const run = await client.instances.create({
  name: "my-training-run",
  gpu_model: "RTX 4090",
  num_gpus: 1,
  pricing_mode: "spot",
});
```

Only `name` is required. `pricing_mode` is `"spot"` or `"on_demand"` and defaults to
`"on_demand"`. `JobIn` also accepts `exclusive_gpu`, `service_tier_price_id`,
`gpu_exclusive_price_id`, `queue_bid_units`, `queue_bid_price_id`,
`placement_preference`, `volume_ids`, `image`, `command`, `git_repo`, and
`init_script`. Read the exported `JobIn` type for those fields. Field names are
snake_case; `max_bid` is not a field.

### Watch, list, and stop instances

```ts
const all = await client.instances.list();
const one = await client.instances.get({ job_id: "JOB_ID" });
const logs = await client.instances.getLogs({ job_id: "JOB_ID" });

await client.instances.cancel({ job_id: "JOB_ID" });
```

### Check wallet balance

```ts
const wallet = await client.billing.getWallet({ customer_id: "CUSTOMER_ID" });
```

## Error handling

The SDK throws typed errors, so distinguish a real failure from a timeout:

```ts
import { XcelsiorApiError, XcelsiorApiTimeoutError } from "@xcelsior-gpu/sdk";

try {
  await client.instances.create({ name: "my-run" });
} catch (err) {
  if (err instanceof XcelsiorApiTimeoutError) {
    // retry — the request may or may not have landed
  } else if (err instanceof XcelsiorApiError) {
    // err.statusCode / err.body carry the API's own message
  }
}
```

A `401` means the access token is missing or expired — mint a fresh one from Quick
Connect rather than retrying.

## Guardrails

- **Estimate the launch you mean.** Call `client.billing.estimateCost()` with that
  `gpu_model`, `duration_hours`, and `spot`, and confirm with the user before
  launching anything priced above roughly C$5/hr.
- **Prefer spot** unless the user needs guaranteed uptime. Pass `pricing_mode: "spot"`.
  The default is `on_demand`. Spot is cheaper and interruptible.
- **Stop what you start.** Cancel idle instances; the user is billed per second, in CAD,
  while they run.
- **Never print or commit the access token.** Treat both env vars as secrets.
- **Do not invent fields.** If you are unsure of a parameter, read the exported types
  from the installed SDK instead of guessing at the wire format.

## Full reference

Interactive API docs, request/response schemas, and more examples:
<https://docs.xcelsior.ca>
