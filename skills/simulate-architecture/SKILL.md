---
name: simulate-architecture
description: Design, cost, stress-test, compare clouds, inject failures, or train an RL agent against simulated AWS/GCP/Azure/OCI/DigitalOcean infrastructure. Use when the user wants Cloud World Model — not real provisioning, live bills, or invented customers.
---

# Simulate architecture with Cloud World Model

Use the **Cloud World Model** MCP server (`cloud-world-model` at `https://www.cloudworldmodel.ai/mcp`). Do not invent a simulator, customers, revenue, or a live cloud bill.

## Operate the live tools

1. Prefer the MCP tools already in this session. Anonymous demo is capped but functional (six tools, session call/step limits). A World Model key unlocks the full tool set and persistent **owned** simulations.
2. **Create owned API simulations.** Call `simulation.create` (or authenticated create/claim tools when the key is present). Treat the returned ID as the handle you manage. Do not reuse ephemeral browser-workspace IDs from earlier UI sessions — those expire and are not durable API objects.
3. Start from `scenario.list` or an explicit resource graph (`compute`, `database`, `storage`, `network`, `cache`, `queue`, `kubernetes`) with provider `aws` | `gcp` | `azure` | `oci` | `digitalocean`.
4. Stress with `simulation.inject_traffic` or `simulation.inject_failure`, then advance time with `simulation.step`. Read current state with `simulation.metrics` — do not step just to inspect.
5. After steps, read `coverageSummary` when present (use `responseMode: "full"` if compact omitted it). Report **modeled vs estimated vs known-gap** (and any not-observable categories). `modeled` means deterministic list-price rules, not a billing-validated invoice.
6. If accuracy validators are available (authenticated): only trust `valid` / `overallValid` when `checked` is `true`. `checked: false` is a vacuous pass. `checked: true` with `skippedCount` > 0 is a **partial** verdict.

## Never

- Provision real cloud resources or claim access to a customer's live account.
- Treat simulated `costPerHour` as an invoice, customer count, or revenue figure.
- Reuse expired browser/demo IDs or invent metrics the tools did not return.
