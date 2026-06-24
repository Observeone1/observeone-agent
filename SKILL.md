---
name: observeone
description: >-
  Set up and manage monitoring on ObserveOne from the command line: uptime (URL)
  monitors, API checks, heartbeat/cron monitoring, status pages, incidents, alert
  channels, and AI-generated Playwright synthetic test suites. Use when a user wants
  to monitor a site or API, get alerted on downtime or slow responses, watch a cron
  job, run end-to-end browser tests, or manage monitoring as code.
---

# ObserveOne

ObserveOne is a monitoring platform. You drive it with the `obs` CLI (`@observeone/cli`).
Every command takes `--json` for machine-readable output, so prefer that and parse the
result. Run `obs <command> --help` for the live flag list of any command.

## 1. Make the CLI available

Check whether `obs` is installed; if not, either install it globally or run it on demand:

```bash
obs --version || npm i -g @observeone/cli      # global install (gives you `obs`)
# or run without installing:
npx @observeone/cli@latest --version            # npm
pnpm dlx @observeone/cli@latest --version       # pnpm (pnpx is deprecated)
```

If you cannot install globally, replace `obs` below with `npx @observeone/cli@latest`
or `pnpm dlx @observeone/cli@latest`.

## 2. Authenticate

You need an ObserveOne API key. You cannot mint the first key yourself: if `OBS_API_KEY`
is not already set, ask the user to create one at https://app.observeone.com/settings/api
and provide it. Set it as an environment variable so it stays out of the command line and
the conversation:

```bash
export OBS_API_KEY="obs1-api-..."   # key from https://app.observeone.com/settings/api
```

Once authenticated, you can mint additional keys with `obs api-key create --name "Agent"`.

Other ways to authenticate:

```bash
obs login                 # interactive, browser-based
obs login --headless      # reads OBS_EMAIL / OBS_PASSWORD (CI)
obs login --api-key <k>   # explicit key
```

Verify auth with any read command, e.g. `obs monitor list --json`.

## 3. Build payloads safely (offline, no login)

When you are unsure of a resource's fields, inspect the schema before creating. These work
offline: `monitor`, `check`, `heartbeat`, `alert-channel`, `status-page`, `incident`.

```bash
obs templates list --json            # every resource type and its required fields
obs schema monitor                   # JSON Schema (Draft-07) for a type
obs validate -r monitor -f ./m.json  # validate a payload before sending
```

Aliases: `api-check` = `check`, `url-monitor` = `monitor`.

## 4. Common tasks

Every resource (`monitor`, `check`, `heartbeat`, `alert-channel`, `status-page`,
`incident`) shares: `list`, `get <id>`, `create`, `update <id>`, `delete <id> -y`.
`monitor`, `check`, and `heartbeat` also have `toggle <id>` (pause/resume) and `run <id>`.

### Uptime (URL) monitor

```bash
obs monitor create --name "Frontend" --url "https://example.com" --interval "*/5 * * * *" --json
obs monitor list --status up --is-active true --json
obs monitor run <id>            # trigger a check now
obs monitor delete <id> -y
```

### API check (with assertions)

```bash
obs check create --name "Auth API" --url "https://api.example.com/auth" --method POST \
  --interval "*/5 * * * *" --header "Authorization=Bearer test" \
  --status-code 200 --response-time-under 800 --json
```

Assertion shorthands: `--status-code`, `--response-time-under`, `--text-contains`,
`--json-path` (+ `--json-path-value`), `--header-exists`, `--regex-match`. Or pass raw
assertion JSON with `--assertion '<json>'`.

### Heartbeat (watch a cron / background job)

```bash
obs heartbeat create --name "Daily Backup" --period 86400 --grace 3600 --json
# then have the job ping the URL the create call returns on each successful run
```

### Alerting

Create a channel, then attach it to monitors with `--alert-channel-id` (repeatable).

```bash
obs alert-channel create --name "Ops" --type email --email "ops@example.com" --json
obs monitor create --name "API" --url "https://api.example.com" \
  --interval "*/5 * * * *" --alert-channel-id <channel-id> --json
```

Channel types: `email`, `slack`, `discord`, `teams`, `telegram`, `sms`, `webhook`.

### Status page

```bash
obs status-page create --name "Public Status" --slug "public-status" --json
obs status-page add-monitor <sp-id> <resource-id> --type url-monitor --name "API" --order 1
```

### Incidents

```bash
obs incident create --title "API Outage" --priority HIGH --description "Investigating" --json
obs incident resolve <id>     # status verbs: resolve, close, reopen
```

### AI Playwright suites (synthetic browser tests)

Suites are AI-generated from a URL. New suites are created only with `generate`.

```bash
obs suite generate https://example.com --name "Smoke Tests" --max-tests 5 --json
obs suite run <id> --wait     # run and block for the result (CI-friendly)
obs suite list --json
```

## 5. Monitoring as code

Manage the whole stack declaratively.

```bash
obs export                  # write all remote resources to obs.json
obs apply --dry-run         # preview the diff
obs apply                   # apply obs.json (updates only changed resources)
```

Note: `apply` cannot create incidents or new suites (use `obs incident create` /
`obs suite generate` for those).

## Notes for agents

- Always pass `--json` and parse the envelope rather than scraping human output.
- Delete is destructive and prompts unless you pass `-y`. Confirm intent before using `-y`.
- Cron expressions drive intervals (e.g. `*/5 * * * *` = every 5 minutes).
- Remote / structured-tool clients (ChatGPT, Claude.ai, or any MCP-capable agent) can
  alternatively connect the ObserveOne MCP server at https://mcp.observeone.com instead of
  the CLI.
