# n8n Workflow Builder

A portfolio of **n8n automation workflows** designed, built and validated with Claude (via the [n8n-MCP](https://github.com/czlonkowski/n8n-mcp) server and [n8n-skills](https://github.com/czlonkowski/n8n-skills)). Each workflow is exported as importable JSON.

## Workflows

| Workflow | What it does | Stack |
|---|---|---|
| [Daily Voice AI Research Newsletter](workflows/agency-daily-voice-ai-newsletter.json) | Every morning at 7AM, pulls RSS feeds, builds a digest, has a research agent analyse it and a writer agent draft a newsletter, then emails it via Gmail. | Schedule Trigger, RSS Read, Code, 2x AI Agent (OpenRouter free models), Gmail |
| [Lead Intake API](workflows/sales-lead-intake-api.json) | Authenticated webhook that validates a lead payload, upserts it into Google Sheets by email (no duplicates), and returns proper 201 / 400 / 502 responses. | Webhook (header auth), Set, IF, Google Sheets, Respond to Webhook |
| [Error Alert](workflows/ops-error-alert.json) | Catch-all error workflow: Error Trigger passes failure details to the Send Alert sub-workflow. Assign it in each workflow's Settings. | Error Trigger, Execute Workflow |
| [Notify: Send Alert](workflows/notify-send-alert.json) | Reusable sub-workflow with typed inputs (`title`, `message`, `severity`) that formats and emails an alert and returns `{ ok }` or `{ ok:false, error }`. | Execute Workflow Trigger, Set, Gmail |
| [FAQ Chat Agent](workflows/support-faq-chat-agent.json) | Chat assistant that answers only from a Google Sheets FAQ, remembers the conversation, and can email a human after the user agrees. | Chat Trigger, AI Agent (OpenRouter free model), Simple Memory, Google Sheets Tool, Gmail Tool |

### Daily Voice AI Research Newsletter

```
Schedule 7AM -> Feed List -> Read Feed -> Build Digest
  -> Research Agent -> Newsletter Writer Agent -> Prepare Email -> Gmail
```

- Two-stage agent pipeline: research first, then writing, each with its own model.
- Runs on free OpenRouter models, so no paid LLM cost.
- Sticky note on the canvas documents the flow.

### Lead Intake API

```
POST /webhook/lead-intake -> Validate Payload -> Valid?
  yes -> Save Lead to Sheet (upsert on email, 3 retries) -> 201
                          \-> error output -> 502
  no  -> 400 with validation message
```

Shows: header-auth webhook, input validation before work, idempotent upsert, status codes that match the cause, and a wired error output.

### FAQ Chat Agent

```
Chat Trigger -> AI Agent <- model (OpenRouter free)
                         <- memory (20 messages, per chat session)
                         <- tool: Search FAQ sheet (Google Sheets)
                         <- tool: Escalate to support team (Gmail, fixed recipient)
```

Shows: tool names and descriptions written as prompts, grounding answers in a data source, `$fromAI` only for content (the recipient is fixed in the node), session memory, and a capped iteration count.

### Sub-workflow pattern

`[Ops] - Error Alert` calls `Notify: Send Alert` through an Execute Workflow node. The sub-workflow has a typed "Define Below" input contract and returns a consistent result shape, so changing the alert channel happens in one place. Import `Notify: Send Alert` first and paste its workflow ID into the caller.

## How to import

1. In n8n: canvas menu -> **Import from File** (or paste the JSON).
2. Connect your own credentials (OpenRouter, Gmail). No secrets are stored in this repo.
3. Run a manual test, then activate.

## Approach

Template-first research, explicit node configuration, layered validation (`validate_node` -> `validate_workflow`), new workflows created inactive, and no hardcoded credentials. See [CLAUDE.md](CLAUDE.md) for the full process and standards.

## Repo layout

- `workflows/` - exported workflow JSON
- `.claude/skills/` - n8n skills used while building
- `CLAUDE.md` - project instructions for Claude

## License

MIT, see [LICENSE](LICENSE). The bundled skills in `.claude/skills/` come from [czlonkowski/n8n-skills](https://github.com/czlonkowski/n8n-skills) and keep their original license.
