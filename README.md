# Cloud World Model — Cursor Plugin

Official Cursor / Grok Bot plugin for [Cloud World Model](https://www.cloudworldmodel.ai) by Canvas Cloud AI.

This repository packages the **existing hosted MCP**. It is not a simulation engine and does not ship a local server. One-click install points Cursor and Grok Bot at the live Streamable HTTP endpoint so anyone can simulate multi-cloud architecture without provisioning real resources.

**Grok Bot plugins are this same Cursor Marketplace listing.** There is no separate Grok Bot package. Once the plugin is listed, install it from Plugins in Cursor or in Grok Bot.

## Who it is for

Architects, SREs, FinOps, and AI agents who want to:

- Design AWS, GCP, Azure, OCI, or DigitalOcean topologies in a sandbox
- Estimate cost, latency, CPU, errors, and resilience before touching a real account
- Inject traffic spikes or node failures and step the model forward
- Compare providers or train an RL agent against modeled infra

Anonymous demo works with **zero config**. A World Model key unlocks the full tool set and persistent owned simulations.

## What this plugin installs

| Piece | Role |
| --- | --- |
| `mcp.json` | Hosted Streamable HTTP MCP — no stdio, no `npx`, no localhost |
| `skills/simulate-architecture` | Operational guidance for design / cost / chaos / comparison workflows |
| `.cursor-plugin/plugin.json` | Marketplace manifest (`cloud-world-model` v1.1.0) |

Hosted MCP URL (verified live, no API key required to initialize):

```text
https://www.cloudworldmodel.ai/mcp
```

As of 2026-08-31 the hosted anonymous MCP exposes **9 tools** (no API key). Caps: max 2 active simulations, 10 resources, 20 persisted steps, ~30 min TTL, 10000 RPS traffic cap. A World Model key unlocks the full tool set and persistent owned simulations.

This plugin does **not** package the local stdio server (`npx cwm-mcp`). That package defaults to `localhost:5000` and is the wrong transport for Grok Bot and other people's machines.

## Anonymous 9-tool loop

`scenario.list` returns **compact catalog cards only** (~40 scenarios, ~37KB): `id`, `title`, `name`, `description` (capped at 500 chars), `difficulty`, `tags`, `category`, `duration`, `provider` / `providers` / `providerSummary`, `resourceCount`, `connectionCount`. It does **not** include `resources`, `connections`, or traffic/failure graphs.

Documented agent workflow:

1. `scenario.list` — pick a card. Prefer `resourceCount` ≤ 10 for anonymous create.
2. `scenario.get` with required argument **`scenarioId`** (not `id`) — hydrate one full graph (`resources`, `connections`, `defaultTrafficPatterns`, etc.).
3. `simulation.create` with those hydrated `resources` / `connections`. Trim extra keys if create rejects `additionalProperties`: resource items allow `id`, `type`, `name`, `provider`, `characteristics`, `recoveryPolicy`. Connection items allow `sourceId`, `targetId`, `label`.
4. Pass **`simulationId` from create on every later call.** Cursor and Grok Bot open a fresh MCP session per tool call and do not persist `Mcp-Session-Id`. Omitting `simulationId` returns `NO_ACTIVE_SIMULATION`. The create id is a short-lived unguessable capability (~30 min) that survives that teardown.
5. `simulation.inject_traffic` / `simulation.inject_failure` → `simulation.step` → `simulation.metrics`.
6. `simulation.recover_resource` to close a reversible failure story (cannot restore `instance_kill`).
7. `simulation.delete` to free a demo slot (`{ deleted: true, id }`).

Three live catalog scenarios currently exceed the 10-resource create cap. `scenario.get` will hydrate them; anonymous create will bounce: `aws-multi-region-failover` (11), `zombie-infra-aws` (13), `github-inspired-cascading-retry-storm` (14). Pick `resourceCount` ≤ 10, trim, or use an API key.

CWM is a simulator, not a real cloud account. Do not invent customers, revenue, live bills, or features that are not live. Prefer MCP tools over guessed costs. Report coverage as **modeled vs estimated vs known-gap**. Only trust accuracy validators when `checked` is `true`.

## Install from Cursor Marketplace (once listed)

1. Open **Customize** (or Plugins) in Cursor, or **Plugins** in Grok Bot.
2. Search for **Cloud World Model**.
3. Install. Reload if prompted.
4. Confirm the `cloud-world-model` MCP server is connected and the `simulate-architecture` skill is available.

No API key is required for the anonymous demo.

## Test locally before submit

1. Copy this repo to the local plugins folder:

   ```bash
   mkdir -p ~/.cursor/plugins/local
   cp -R . ~/.cursor/plugins/local/cloud-world-model
   ```

   Or symlink the working tree:

   ```bash
   ln -s /path/to/this-repo ~/.cursor/plugins/local/cloud-world-model
   ```

2. In Cursor: **Developer: Reload Window**.
3. Open **Customize** and confirm:
   - Plugin `cloud-world-model` is discovered
   - MCP server `cloud-world-model` is listed and connects to `https://www.cloudworldmodel.ai/mcp`
   - Skill `simulate-architecture` appears (Agent Decides / `/simulate-architecture`)

On Teams / Enterprise, local plugin imports may be disabled by admin policy. A marketplace listing with the same name takes precedence over the local copy.

## Optional API key

Anonymous demo works without a key. To unlock the full tool set and persistent **owned** API simulations (not ephemeral browser IDs):

1. Get a World Model key from [Getting started](https://www.cloudworldmodel.ai/getting-started) (register a Canvas Cloud AI key, then exchange it for a `cwm_live_...` World Model key).
2. In Cursor: **Plugins →** this plugin **→ Configure**.
3. Set **Cloud World Model API key** (`CWM_API_KEY`).

`mcp.json` in this version is **URL-only** — it does not send `Authorization: Bearer ${CWM_API_KEY}`. An empty `${CWM_API_KEY}` substitution would break anonymous Cursor installs and Grok Bot. After you set the variable in Configure, a later plugin version can wire the header once Cursor's optional-variable behavior is confirmed. Until then, a keyed user can also add a personal MCP entry that includes the header (and disable the plugin's duplicate server so only one copy runs).

**Never commit a key.** This repo has no secrets.

## Submit to Cursor Marketplace

Submit this public GitHub repository: [`https://github.com/canvascloudai/cwm-cursor-plugin`](https://github.com/canvascloudai/cwm-cursor-plugin). That is the remote Cursor Marketplace review needs. Origin was used to author the first slice; keep it if you want a second remote, but do not submit an Origin URL.

1. Confirm `.cursor-plugin/plugin.json` `repository` is `https://github.com/canvascloudai/cwm-cursor-plugin`.
2. Confirm the repo is public, open source (MIT), has no secrets, and the logo path `assets/logo.svg` is relative.
3. Apply as a publisher and submit the GitHub URL at [cursor.com/marketplace/publish](https://cursor.com/marketplace/publish).
4. Reviews are manual. Expect about **1–2 weeks**. Questions: [marketplace-publishing@cursor.com](mailto:marketplace-publishing@cursor.com).

Grok Bot picks up the **same** marketplace listing. Do not submit a second plugin.

Checklist from [Cursor's plugin reference](https://cursor.com/docs/reference/plugins):

- [x] `.cursor-plugin/plugin.json` with unique kebab-case `name`
- [x] `mcp.json` hosted URL only (no localhost, no stdio)
- [x] Skill with YAML frontmatter
- [x] Logo committed and referenced by relative path
- [x] MIT `LICENSE`
- [x] README covering install and optional config
- [x] Public GitHub repo: [canvascloudai/cwm-cursor-plugin](https://github.com/canvascloudai/cwm-cursor-plugin)
- [ ] Local test: `~/.cursor/plugins/local/cloud-world-model` + Reload Window

## Docs

- Product: [cloudworldmodel.ai](https://www.cloudworldmodel.ai)
- MCP quick start: [cloudworldmodel.ai/mcp-quickstart](https://www.cloudworldmodel.ai/mcp-quickstart)
- Getting a key: [cloudworldmodel.ai/getting-started](https://www.cloudworldmodel.ai/getting-started)
- Cursor plugins: [cursor.com/docs/plugins](https://cursor.com/docs/plugins)
- Plugin reference / submit checklist: [cursor.com/docs/reference/plugins](https://cursor.com/docs/reference/plugins)
- Marketplace publish: [cursor.com/marketplace/publish](https://cursor.com/marketplace/publish)

## License

[MIT](LICENSE) © Canvas Cloud AI
