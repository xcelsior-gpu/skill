---
name: xcelsior-gpu
description: Rent and manage GPU compute on Xcelsior directly from your agent — list available GPUs, check spot pricing, estimate cost, launch and monitor instances, and check your wallet balance using the official TypeScript SDK.
---

# Xcelsior GPU Compute

Xcelsior is a GPU cloud where you rent verified GPUs by the hour (or run serverless
inference) and pay in CAD. This skill lets an agent drive the platform through the
official SDK — typed, with real error classes, rather than hand-rolled HTTP.

## Setup

1. Get an access token: open <https://xcelsior.ca/dashboard/mcp> (Quick Connect shows a
   token ready to copy) or go to Settings → API & SSH.
2. Export it:
   ```bash
   export XCELSIOR_ACCESS_TOKEN="xcl_..."
   ```
3. Install the SDK in the working directory:
   ```bash
   npm install @xcelsior-gpu/sdk
   ```

Construct a client once and reuse it:

```ts
import { XcelsiorApiClient } from "@xcelsior-gpu/sdk";

const client = new XcelsiorApiClient({ token: process.env.XCELSIOR_ACCESS_TOKEN });
```

The client is organised by resource — `gpu`, `marketplace`, `instances`, `billing`,
`serverless`, `volumes`, `teams`, `sshKeys` — mirroring the REST API.

## Core operations

### List available GPUs

```ts
const gpus = await client.gpu.listAvailable();
```

### Browse or search the marketplace

```ts
const listings = await client.marketplace.list();
const filtered = await client.marketplace.search({ /* filters */ });
```

### Estimate cost before launching (always do this first)

```ts
const quote = await client.billing.estimateCost();
```

### Launch an instance

```ts
const run = await client.instances.create({ name: "my-training-run" });
```

Only `name` is strictly required. You will usually also want to specify the GPU model,
GPU count, and pricing mode (`spot` or `on_demand`) — inspect the `JobIn` type the SDK
exports for the exact field names rather than guessing.

### Watch, list, and stop instances

```ts
const all = await client.instances.list();
const one = await client.instances.get({ job_id: "JOB_ID" });
const logs = await client.instances.getLogs({ job_id: "JOB_ID" });

await client.instances.cancel({ job_id: "JOB_ID" });
```

Request parameters are snake_case (`job_id`, `customer_id`), matching the API.

### Check wallet balance

```ts
const wallet = await client.billing.getWallet({ customer_id: "CUSTOMER_ID" });
```

### List GPU tiers

```ts
const tiers = await client.instances.listTiers();
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

- **Estimate cost first.** Call `client.billing.estimateCost()` and confirm with the
  user before launching anything priced above roughly $5/hr.
- **Prefer spot** unless the user needs guaranteed uptime — it is typically much
  cheaper, at the cost of being interruptible.
- **Stop what you start.** Cancel idle instances; the user is billed per second while
  they run.
- **Never print or commit the access token.** Treat `XCELSIOR_ACCESS_TOKEN` as a secret.
- **Do not invent fields.** If you are unsure of a parameter, read the exported types
  from the SDK instead of guessing at the wire format.

## Full reference

Interactive API docs, request/response schemas, and more examples:
<https://docs.xcelsior.ca>
