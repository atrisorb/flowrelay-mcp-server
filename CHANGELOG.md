# Changelog

All notable changes to `@flowrelay/mcp-server`. Version numbers move in lockstep with the VS Code extension for any change to the tool vocabulary or to a source's capabilities; releases that only touch one surface (an extension-only UI fix, a server-only dependency bump) advance that surface alone.

## 1.0.16 – 2026-08-06

### Changed
- `zod` moved from `^3.24` to `^4.4`. No tool schema changed: `z.record` already used the two-argument form v4 requires, and the only modifiers in play (`.describe`, `.optional`, `.int`, `.default`, `.min`, `.max`) are unchanged. The web app's copy moved in the same change, which is mandatory – the 18 tool schemas are shared verbatim between the two surfaces and compared by `tests/mcp-tool-parity.test.ts`, so a version skew between the two dependency trees produces schemas that no longer match. The JSON Schema emitted to clients was diffed before and after: 18 tools, all `type: object`, descriptions and source enums identical.

## 1.0.15 – 2026-08-02

### Added
- Explicit author metadata in `package.json`.
- Startup banner on `stderr` logging version, copyright, and license when the server connects.
- `## License` section in `README.md` and `@license` headers across TypeScript source files.

## 1.0.14 – 2026-07-29

### Fixed
- Added a default 30-second `AbortSignal` timeout to API client HTTP requests (`FlowRelayAPI.prototype.requestRaw`) to prevent MCP tools from hanging indefinitely if the server or network connection drops.

## 1.0.13 – 2026-07-25

### Added
- `discord_send_message` accepts `last_release_notes` as an artifact shortcut, so the latest release notes of a project can be posted as a `.md` attachment like every other artifact.

### Fixed
- Release-notes generation now persists: the server was writing the insight row with a column that does not exist, so every `generate_release_notes` call failed at save time with a database error after the model had already run.
- Release notes honour the per-source `filters` passed alongside `source` / `repo` / `style` instead of silently ignoring them.
- `source`, `repo` and `style` are validated server-side: an unknown source or a style outside `release_notes` / `pr_description` now returns a 400 naming the field, instead of being accepted and quietly changing the output.

## 1.0.12 – 2026-07-24

### Added
- `generate_release_notes`: turn a project's merged work into release notes or a PR description, returned as Markdown. Costs 3 credits per run.
- `list_digests`: read the scheduled activity digests of a project, newest first. Reading is free – digests are generated on the schedule configured in the dashboard.

## 1.0.11 – 2026-07-24

### Added
- `ask_project`: ask one question about a project and get an answer grounded in its indexed codebase, connected baselines and last 14 days of activity. Answers synchronously (no job to poll) and cites the events it used as `ev:` ids. Costs 2 credits per question.

## 1.0.10 – 2026-07-22

### Added
- `incident_io` joins the source enum: incident.io public incident events (created / updated / status updated) are now filterable in `generate_handoff` and the insight tools, with incident types surfaced by `list_filter_options` and severities as the priority dimension.

## 1.0.9 – 2026-07-20

### Added
- `vercel` joins the source enum: Vercel deployment results (succeeded / failed / cancelled / promoted) are now filterable in `generate_handoff` and the insight tools, with projects surfaced by `list_filter_options`.

## 1.0.8 – 2026-07-16

### Added
- `circleci` joins the source enum: CircleCI workflow results (succeeded / failed / cancelled) are now filterable in `generate_handoff` and the insight tools, with projects surfaced by `list_filter_options`.

## 1.0.7 – 2026-07-15

### Added
- `buildkite` joins the source enum: Buildkite build results (passed / failed / cancelled) are now filterable in `generate_handoff` and the insight tools, with pipelines surfaced by `list_filter_options`.

## 1.0.6 – 2026-07-14

### Added
- `gmail` and `asana` complete the source enum.

## 1.0.5 – 2026-07-08

### Fixed
- Added `pagerduty` to the source enum. It was added to the platform but never reached the client, so `generate_handoff` / the insight tools rejected `sources: ["pagerduty"]` locally before the request left the machine.
- `set_active_project` `clear` parameter description referenced the retired no-project personal scope; it now states that clearing requires selecting a project again before generating.

### Changed
- Every tool description is now self-contained: sources, filters, the async job flow, credit costs and how to discover ids are spelled out in-tool so an agent can operate without opening the docs. Filter descriptions state that source ids are validated (unknown → `400`) while `eventTypes` / `priorities` / `projects` values are matched leniently and discoverable via `list_filter_options`.

## 1.0.4 – 2026-07-04

### Fixed
- Server announced stale version `1.0.0` to MCP clients; now reads from `package.json`.
- `generate_handoff` tool description no longer mentions personal scope (removed since 1.0.0).


## 1.0.3 – 2026-07-03

### Added
- Figma visual context support. `list_filter_options` now surfaces whether Figma is selectable for the project: Figma filters require the project's processing region to be Global. When Figma is selected, generations attach rendered frame previews plus the indexed design scene (layout, texts, prototype flows) and cost 1 extra credit.

### Changed
- Generation endpoints return `400` when Figma is the only selected source on a non-Global project; the error message explains the region requirement.

## 1.0.2 – 2026-07-03

### Changed
- `generate_handoff` and the three insight tools output the server-rendered `markdown` field (canonical serializer, byte-identical to the dashboard copy button) with a title+summary fallback for pre-markdown servers.

## 1.0.1 – 2026-06-29

### Changed
- `discord_send_message` can now send a Flow Relay artifact instead of plain text. Provide exactly one of: `content` (inline text); `handoff_id` or `insight_id` (renders that artifact to Markdown and attaches it as a `.md` file); or `artifact` (`last_handoff` / `last_correlation` / `last_onboarding` / `last_architecture`, with `project_id`) to send the latest active artifact of that kind. The Markdown matches the dashboard copy button.

## 1.0.0 – 2026-06-28

### Added
- `list_filter_options` tool – fetches the real selectable filter values (resources, branches, event types, priorities) per source for a project via `GET /api/v1/handoffs/filters`. Agents call it before generating so `filters` use real values instead of guesses. Brings the tool count to 15.

### Changed
- `generate_handoff` and the three insight tools (`generate_correlation_insight`, `generate_onboarding_brief`, `generate_architecture_insight`) accept per-source `filters`: `projects`, `eventTypes`, `branches`, `priorities`.
- `list_handoffs` status enum aligned to the database: `active`, `archived`, `all` (dropped the non-existent `paused`/`completed`).
- `list_insights` status enum aligned to the database: `active`, `archived`, `all` (dropped `completed`).

### Removed
- Dead inline-handoff response branch – `generate_handoff` is always asynchronous (202 + jobId) and the server polls to completion.
