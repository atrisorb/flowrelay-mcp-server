# Flow Relay MCP Server

Flow Relay MCP Server adds project-aware, multi-tenant Flow Relay tools to MCP clients such as Claude Desktop and Claude Code.

It connects to Flow Relay API v1 using an API key and supports:

- Project scope (personal project or organization project). Every handoff and AI insight is tied to a project. Events and integrations remain user-level.

## Package

- Name: @flowrelay/mcp-server
- Version: 1.0.16

## What Is Included

The server currently exposes these tools:

- get_workspace_context
- list_projects
- set_active_project
- list_filter_options (lists the selectable resources, branches, event types and priorities per source for a project – call this before generating so handoff/insight filters use real values instead of guesses; also reports whether Figma is selectable, since Figma requires the project's processing region to be Global)
- list_handoffs (returns project-specific handoffs, or an aggregated view of all accessible project handoffs if no project is active; filter by status: active, archived, or all)
- ask_project (ask a question about a project and get an answer grounded in codebase, baselines and 14 days of activity; 2 credits per question)
- generate_handoff (processed asynchronously, and the server automatically polls until the job finishes. Requires active project or project_id. Accepts per-source filters: projects, eventTypes, branches, priorities.)
- generate_correlation_insight (accepts per-source filters)
- generate_onboarding_brief (accepts per-source filters)
- generate_architecture_insight (accepts per-source filters)
- generate_release_notes (generates release notes or PR descriptions from code activity; accepts per-source filters)
- list_digests (lists past scheduled activity digests for a project)
- list_insights
- list_integrations
- list_events
- list_untracked_resources (lists event-producing resources not scoped to any project – useful for discovering untracked activity)
- discord_list_channels
- discord_send_message (plain text, or a handoff/insight by id or `last_*` – `last_handoff`, `last_correlation`, `last_onboarding`, `last_architecture`, `last_release_notes` – rendered to Markdown and attached as a `.md` file)

## Environment Variables

Required:

- FLOWRELAY_API_KEY

Optional:

- FLOWRELAY_PROJECT_ID
- FLOWRELAY_BASE_URL

If FLOWRELAY_PROJECT_ID is set, it becomes the default project context for project-aware tools unless you override it per call.

## Quick Start (Claude Desktop)

Add this to your claude_desktop_config.json:

```json
{
  "mcpServers": {
    "flowrelay": {
      "command": "npx",
      "args": ["-y", "@flowrelay/mcp-server"],
      "env": {
        "FLOWRELAY_API_KEY": "fr_your_api_key_here",
        "FLOWRELAY_PROJECT_ID": "your_project_id"
      }
    }
  }
}
```

## Multi-Tenant Behavior

- Every handoff and insight is tied to a project; you need an active project (`set_active_project`) or `project_id`. Events and integrations remain user-level.
- You can select a project during the MCP session with set_active_project.
- You can override scope per call by passing project_id where supported.

Recommended flow:

1. Call get_workspace_context
2. Call list_projects
3. Call set_active_project
4. Run handoff and query tools in the selected scope

## Local Development

From this folder:

```bash
npm install
npm run build
```

Create a tarball package:

```bash
npm pack
```

## Troubleshooting

- Error: Missing FLOWRELAY_API_KEY
  - Set FLOWRELAY_API_KEY in your MCP client configuration.
- Project not found or inaccessible
  - Run list_projects and use one of the returned IDs.
- No events or handoffs returned
  - Verify active scope and data availability in that scope.

## Related Docs

- Documentation & Platform: https://www.flowrelay.it

## License

Copyright © 2026 Adriano Sorbello ([@atrisorb](https://github.com/atrisorb)). All rights reserved.

Distributed under the GNU Affero General Public License v3.0 or later (AGPL-3.0-or-later). See [LICENSE](LICENSE) for more information.
