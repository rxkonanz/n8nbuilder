# n8n Workflow Builder

## Project Overview

This project lets you build n8n workflows and AI agents by prompting Claude. Claude uses two tools in combination:

1. **n8n MCP** (`n8n-mcp`) — direct API access to your n8n Cloud instance via `.mcp.json`
2. **n8n Skills** (`n8n-mcp-skills` plugin) — 7 specialist skills that auto-activate for node config, expressions, patterns, validation, and code

Target: **n8n Cloud** for building AI agents and automations.

---

## Setup Status

| Tool | Status | Action needed |
|------|--------|---------------|
| n8n-mcp MCP server | `.mcp.json` created | Fill in `N8N_API_URL` and `N8N_API_KEY` |
| n8n-skills plugin | Installed (user scope) | None — auto-activates |

### Connecting the MCP to your n8n Cloud instance

1. Get your API key: n8n Cloud dashboard → **Settings → n8n API → Create an API key**
2. Edit `.mcp.json` in this folder:
   - Replace `https://your-instance.app.n8n.cloud` with your actual instance URL
   - Replace `your-api-key-here` with your API key
3. Restart Claude Code — the MCP will connect automatically
4. Verify: ask Claude "List my n8n workflows"

> `.mcp.json` is gitignored — your credentials stay local.

---

## Available Tools

### n8n MCP Tools (via `.mcp.json`)
- List, get, create, update, delete, execute workflows
- Browse 1,084+ node docs (core + community)
- Search workflow templates (2,700+)
- Validate node configurations

### n8n Skills (auto-activate by context)
| Skill | Activates when |
|-------|----------------|
| MCP Tools Expert | Searching nodes, validating configs |
| Workflow Patterns | Designing workflow architecture |
| Expression Syntax | Writing `{{ }}` expressions |
| Validation Expert | Debugging validation errors |
| Node Configuration | Configuring node operations/params |
| Code JavaScript | Writing Code node JS scripts |
| Code Python | Writing Code node Python scripts |

---

## How Claude Should Approach Workflow Requests

1. **Clarify** — confirm the goal, trigger type, required integrations, expected output
2. **Design** — outline the node structure before building (trigger → logic → output)
3. **Build** — use MCP tools to create or update the workflow in n8n Cloud
4. **Verify** — list the workflow or run a test execution to confirm it works

For non-trivial workflows, show the design first and get approval before creating.

### Parameter Casing Rules

Before creating or updating any node, always call `get_node` to verify the exact parameter names and their accepted values. Never assume casing.

- **Parameter keys** are `camelCase` — e.g. `chatId`, `returnAll`, `parse_mode` (never `ChatId`, `ReturnAll`)
- **Enum values** must match exactly — e.g. `"sendMessage"` not `"SendMessage"`, `"getAll"` not `"GetAll"`, `"metadata"` not `"Metadata"`
- **Resource and operation fields** are always lowercase strings — always set them explicitly, never rely on defaults
- **Expression output fields** — verify the actual output key from the previous node (e.g. chainLlm outputs `text`, not `output` or `response`) before referencing with `$json.fieldName`
- When in doubt, retrieve a live workflow with `n8n_get_workflow` (mode `full`) and inspect the saved parameters as the source of truth

---

## Conventions

### Naming
Format: `[Trigger] → [Action] — [Purpose]`
Example: `Webhook → Slack — New Lead Notification`

### Tagging
- Category: `ai-agent`, `automation`, `webhook`, `data`
- Status: `active`, `draft`, `archived`

### AI Agent Workflows
- **AI Agent node** (LangChain) as orchestrator
- **OpenAI Chat Model** node for LLM calls
- **Window Buffer Memory** for conversational context
- Tools connect as sub-nodes to the AI Agent

### Automations
- Add **Error Trigger** workflow for fault alerting
- Use **Schedule Trigger** for recurring jobs
- Use **IF nodes** for branching (not Code nodes)

### Credentials
- Never hardcode API keys — always reference saved n8n credentials
- If a credential doesn't exist yet, ask the user to add it in n8n before proceeding
