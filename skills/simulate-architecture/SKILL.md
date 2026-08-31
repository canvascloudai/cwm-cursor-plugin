---
name: simulate-architecture
description: Design, cost, stress-test, compare clouds, inject failures, or train an RL agent against simulated AWS/GCP/Azure/OCI/DigitalOcean infrastructure. Use when the user wants Cloud World Model — not real provisioning, live bills, or invented customers.
---

# Simulate architecture with Cloud World Model

Use the **Cloud World Model** MCP server (`cloud-world-model` at `https://www.cloudworldmodel.ai/mcp`). CWM is a simulator, not a real cloud account. Do not invent a simulator, customers, revenue, live bills, or features that are not live. Prefer MCP tools over guessed costs.

## Operate the live anonymous tools (no API key)

As of 2026-08-31 the hosted anonymous MCP exposes **9 tools**. A World Model API key still unlocks the full tool set and persistent **owned** simulations.

1. `scenario.list` — compact catalog cards only (~40 scenarios, ~37KB). Fields: `id`, `title`, `name`, `description` (capped at 500 chars), `difficulty`, `tags`, `category`, `duration`, `provider` / `providers` / `providerSummary`, `resourceCount`, `connectionCount`. Does **not** include `resources`, `connections`, or traffic/failure graphs. Optional filters: `provider`, `category`, `difficulty`.
2. `scenario.get` — hydrate **one** scenario. Required argument: `scenarioId` (not `id`). Returns the full graph: `resources`, `connections`, `defaultTrafficPatterns`, and related metadata.
3. `simulation.create` — start a demo from hydrated `resources` / `connections`, or from your own graph (`compute`, `database`, `storage`, `network`, `cache`, `queue`, `kubernetes` with provider `aws` | `gcp` | `azure` | `oci` | `digitalocean`).
4. `simulation.step` — advance time (max 20 persisted steps in demo).
5. `simulation.metrics` — read current state without advancing time.
6. `simulation.inject_traffic` — set absolute RPS, or omit `traffic` for a random spike (capped at 10000 RPS).
7. `simulation.inject_failure` — fail one node (target by `resourceId` or `resourceName` for replay).
8. `simulation.recover_resource` — recover a reversible failure. Cannot restore `instance_kill`.
9. `simulation.delete` — free a demo slot. Success: `{ deleted: true, id }`.

## Documented workflow

1. Call `scenario.list` and pick a card. Prefer `resourceCount` ≤ 10 for anonymous create.
2. Call `scenario.get` with `{ "scenarioId": "<id from the card>" }`. Do not pass `id`.
3. Call `simulation.create` with the hydrated `resources` and `connections`. Trim extra keys if the create schema rejects `additionalProperties`. Resource items allow `id`, `type`, `name`, `provider`, `characteristics`, `recoveryPolicy`. Connection items allow `sourceId`, `targetId`, `label`. Scenario graphs often include layout (`x`/`y`), `status`, `location`, connection `id`, and traffic presets — strip those before create.
4. **Pass `simulationId` from create on every later call.** Cursor and Grok Bot open a fresh MCP session per tool call and do not persist `Mcp-Session-Id`. Omitting `simulationId` returns `NO_ACTIVE_SIMULATION`. The create id is a short-lived unguessable capability (~30 min) that survives that teardown. Do not treat it as a durable share link or reuse expired browser-workspace IDs.
5. Stress with `simulation.inject_traffic` or `simulation.inject_failure`, then `simulation.step`, then `simulation.metrics`. Do not step just to inspect.
6. Call `simulation.recover_resource` to close a reversible failure story. Lower traffic to a serviceable level first, then step until `recoveryProgress.state` is `healthy`. It cannot restore `instance_kill` (that permanently removes the instance).
7. Call `simulation.delete` when finished so the next anonymous create is not blocked by the 2-sim cap.

## Anonymous caps (honest, not workarounds)

- Max **2** active anonymous simulations
- Max **10** resources per create
- Max **20** persisted steps
- ~**30 min** TTL on the simulationId capability
- **10000 RPS** traffic cap

Three live catalog scenarios currently exceed the 10-resource create cap. `scenario.get` will hydrate them; anonymous `simulation.create` will bounce:

- `aws-multi-region-failover` (11)
- `zombie-infra-aws` (13)
- `github-inspired-cascading-retry-storm` (14)

Pick a scenario with `resourceCount` ≤ 10, trim resources/connections yourself, or use an API key.

If create fails because two sims are already active, `simulation.delete` an existing id (or wait for TTL).

## Coverage and accuracy

After steps, read `coverageSummary` when present (use `responseMode: "full"` if compact omitted it). Report **modeled vs estimated vs known-gap** (and any not-observable categories). `modeled` means deterministic list-price rules, not a billing-validated invoice.

If accuracy validators are available (authenticated): only trust `valid` / `overallValid` when `checked` is `true`. `checked: false` is a vacuous pass. `checked: true` with `skippedCount` > 0 is a **partial** verdict.

## Never

- Provision real cloud resources or claim access to a customer's live account.
- Treat simulated `costPerHour` as an invoice, customer count, or revenue figure.
- Reuse expired browser/demo IDs or invent metrics the tools did not return.
- Assume `scenario.list` returned graphs. It returns cards. Hydrate with `scenario.get`.
- Omit `simulationId` on Cursor / Grok Bot after create.
