# ObserveOne agent skill

A self-bootstrapping [agent skill](https://skills.sh) that teaches any AI agent to set up
and manage monitoring on [ObserveOne](https://observeone.com) from the command line:
uptime monitors, API checks, heartbeat/cron monitoring, status pages, incidents, alert
channels, and AI-generated Playwright synthetic test suites.

## Install

```bash
npx skills add Observeone1/observeone-agent
```

The skill drives the `obs` CLI ([`@observeone/cli`](https://www.npmjs.com/package/@observeone/cli))
and bootstraps it automatically, so an agent can go from zero to a working monitor without
a human touching the dashboard.

## What's here

- [`SKILL.md`](./SKILL.md) — the skill itself: auth, the full command surface with
  examples, and config-as-code.

## Remote / MCP clients

Agents that prefer structured tools over a CLI (ChatGPT, Claude.ai, or any MCP-capable
client) can connect the ObserveOne MCP server at <https://mcp.observeone.com> instead.

## License

Apache-2.0.
