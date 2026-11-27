# google-workspace-mcp

An OpenClaw plugin for Google Workspace that reaches Google only through Google's own MCP servers, the way Google says AI agents must from 1 February 2027.

**Status: design stage.** There is no code yet. This README states what the plugin is for, what it is not for, and the migration path it follows. Code and a working assessment will land here as they are ready.

## Why this exists

On 24 November 2026 Google announced that AI agents and access tools must use its hosted MCP servers for Workspace data instead of calling the Workspace REST APIs directly, with technical enforcement from 1 February 2027. Gmail, Drive and Calendar come first; other products follow. The same announcement introduced two usage tiers (Standard, free with published quotas; Scaled, metered above a daily free allowance from 1 April 2027), a Workspace AI Tools Addendum that forbids agents from keeping Workspace-derived data for extended periods or scraping the APIs, and a liability disclaimer for third-party agents.

Source: [Workspace Updates, November 2026: MCP servers, CLI, standardized usage tiers](https://workspaceupdates.googleblog.com/2026/11/mcp-servers-cli-standardized-usage-tiers-now-available.html).

Every agent tool built on the Workspace REST APIs, including the author's own, stops being compliant on that date. This plugin is the replacement.

## Prior art

- [openclaw/gogcli](https://github.com/openclaw/gogcli): the `gog` command-line client for Google services, REST-based.
- [steipete's `gog` skill on ClawHub](https://clawhub.ai/steipete/skills/gog): the agent skill around that client.
- The `gog` plugin bundled with OpenClaw itself.
- `google-api` (the author's, private): a unified Google API CLI for Termux and OpenClaw derived from the above, with device-flow OAuth and no dependencies. It is the direct ancestor of this plugin and the thing it retires.

## Design objectives

1. **Conform to Google's projected policy.** All Workspace data access goes through the official per-product MCP endpoints, using the OAuth client type Google specifies for agents. Nothing calls a Workspace REST API.
2. **Conform to general OpenClaw plugin standards.** Manifest, tool registration, config schema, optional tools gated behind the operator's allowlist: it should install and behave like any other OpenClaw plugin, and nothing about it should need special treatment.
3. **Narrow surface.** Expose what the operator enables and no more, and keep no local archive of fetched Workspace content, in line with the Addendum.

## Migration path

Google describes the move from API to MCP as: join the Workspace Developer Preview Program, create an OAuth client designated for an AI-powered agent (or use the quick setup for personal use), enable the product API, point the client at the product's MCP endpoint, and sign in. The servers are remote, Google-hosted, and speak Streamable HTTP.

Workspace products with an official MCP server as of November 2026: Gmail, Drive, Docs, Sheets, Slides, Calendar, Chat, People, Tasks, Classroom, plus a Universal Search server spanning Gmail, Drive, Calendar and Chat. Google publishes no copyable client configuration; the live `tools/list` of each server is the source of truth for what it offers.

For each operation an existing REST-based tool performs, one of three things is true:

| Verdict | Meaning | Examples |
|---|---|---|
| Migrate | An official MCP tool covers it. Swap the transport, keep the behaviour. | Gmail search and read, Drive files, Calendar events, Docs, Sheets, Tasks |
| Keep REST | Outside the mandate. Not Workspace data, or not a Google Cloud API at all. | SMTP send, Maps (API-key based), the OAuth flow itself |
| No equivalent | Nothing official covers it. Drop it, or meet the need another way. | Blogger, Keep, Gmail filters, Drive sharing, Gmail send |

## Not in scope

- **Replacing deficiencies in the offered MCP APIs.** If a Google MCP server has no tool for something, this plugin has no tool for it. It does not fill gaps with REST calls or workarounds.
- **How OpenClaw as a whole should conform to evolving licensing around API and MCP tools.** That is a question for the OpenClaw project, not for one plugin.

## Design policy

> This works as the powers that be intend; if your implementation deviates, that's your decision.

## Known issues on Google's side

The MCP servers are a Developer Preview and have bugs. One is filed as [Google issue 590216050](https://issuetracker.google.com/issues/590216050); others will be filed as they are confirmed.
