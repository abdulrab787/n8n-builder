# n8n Workflow Builder

This project exists to design, build, and validate **n8n workflows** in my n8n instance using Claude. It is not a code project: the deliverable is a working, high-quality workflow in n8n.

## Instance details (fill in)
- n8n URL: `https://your-instance.app.n8n.cloud`
- Hosting: `cloud` (matters for Code node limits, see below)
- Services with credentials already set up in n8n: `<e.g. Gmail, Slack, Google Sheets, OpenAI>`
- Workflow naming prefix: `<optional>`

## Tools

### 1. n8n-MCP server ([czlonkowski/n8n-mcp](https://github.com/czlonkowski/n8n-mcp))
Needs `N8N_API_URL` and `N8N_API_KEY` for the management tools. Run `n8n_health_check` first if anything looks disconnected.

**Documentation tools (no API needed)**
- `tools_documentation`: read first in a new session for tool best practices.
- `search_nodes` (use `includeExamples: true`), `get_node` (detail `minimal` ~200 tokens, `standard` ~1k, `full` 3-8k; also docs, property search, versions).
- `validate_node` (`mode: minimal` for required fields, `mode: full` with `profile: runtime` before finalizing), `validate_workflow`.
- `search_templates` (by task, by nodes, by metadata), `get_template`.

**Instance tools (need API)**
- Workflows: `n8n_create_workflow`, `n8n_get_workflow`, `n8n_update_partial_workflow` (diff-based, preferred), `n8n_update_full_workflow`, `n8n_list_workflows`, `n8n_validate_workflow`, `n8n_autofix_workflow`, `n8n_workflow_versions` (history, diff, rollback), `n8n_delete_workflow`.
- Testing: `n8n_test_workflow`, `n8n_executions`.
- Other: `n8n_explore_node_resources` (real dropdown values such as channels or sheets), `n8n_deploy_template`, `n8n_manage_credentials`, `n8n_manage_folders`, `n8n_manage_datatable`, `n8n_list_catalog`, `n8n_audit_instance`.

### 2. n8n skills ([czlonkowski/n8n-skills](https://github.com/czlonkowski/n8n-skills))
Skills teach correct usage of the MCP tools. Invoke the matching skill **before** the work it covers:
- **Expression Syntax**: writing `{{ }}` expressions. Webhook payload lives under `$json.body`.
- **MCP Tools Expert**: tool choice, nodeType formats, validation profiles.
- **Workflow Patterns**: webhook, HTTP API, database, AI, and scheduled patterns.
- **Node Configuration**: property dependencies, operation-specific fields, AI connection types.
- **Validation Expert**: interpreting errors and false positives, fix loops.
- **Code JavaScript / Code Python / Code Tool**: Code nodes return `[{json:{...}}]`; Code *Tools* return strings. Prefer JavaScript (Python imports are almost all blocked on n8n Cloud).
- **Error Handling**: error outputs, `retryOnFail`, Error Trigger workflows, webhook 4xx/5xx responses.
- **Binary & Data**: files never live in `$json`; use `$binary` and keep them alive across transforms.
- **Sub-workflows**: extract shared logic or anything over ~10 nodes; verb-first names.
- **AI Agents**: Agent vs LLM Chain vs Classifier, tool descriptions as prompts, `$fromAI`, memory, RAG.
- **Multi-Instance / Self-Hosting**: only if I use several instances or ask for deployment help.

If a tool or skill is unavailable or failing, say so instead of working from memory.

## Process for every request
1. **Understand**: confirm trigger, inputs, outputs, and services. Ask only about blockers; otherwise state assumptions.
2. **Template first**: run `search_templates` (several search modes in parallel) before building from scratch. If a template is used, attribute it: "Based on template by **[author]** (@[username]). View at: [url]".
3. **Research nodes** if no template fits: `search_nodes` then `get_node`. Never guess node types, versions, or parameter names. Run independent lookups in parallel.
4. **Configure explicitly**: set **every** relevant parameter. Relying on defaults is the top cause of runtime failures.
5. **Validate in layers**: `validate_node` (minimal, then full/runtime) -> `validate_workflow` -> fix all errors. Validation passing is necessary, not sufficient.
6. **Build incrementally** with `n8n_update_partial_workflow`, batching multiple operations per call. Connection operations need all four string parameters; IF nodes need `branch: "true"` / `"false"` to route outputs; Switch nodes need the right output index.
7. **Test**: use `n8n_test_workflow` (with safe test data), inspect with `n8n_executions`, then fix.
8. **Report**: what was built, workflow name/ID, assumptions, and manual steps left (connect credentials, activate, real test data).

Work quietly: run tools without running commentary and summarize once at the end.

## Quality standards
- Descriptive node names ("Fetch Open Invoices", not "HTTP Request1"); sticky notes for non-obvious logic; readable layout.
- Error handling: Error Trigger workflow, `retryOnFail` for flaky APIs, per-node error outputs, `continueOnFail` only when intentional.
- Prefer native nodes over Code nodes; keep Code nodes short and commented.
- Check expressions against the actual data shape of the upstream node.
- Never hardcode credentials or keys; use n8n credentials.
- Authenticate webhooks and validate incoming data.
- Batch or paginate large datasets; avoid redundant nodes and API calls.
- Make workflows idempotent where possible (no duplicates on re-run).

## Safety rules
- **Never edit a production workflow directly.** For an existing workflow: export or copy it first (or confirm `n8n_workflow_versions` has a restore point), then modify the copy or get my go-ahead.
- Create new workflows **inactive**. Ask before activating.
- Ask before `n8n_delete_workflow`, `n8n_update_full_workflow` on an existing workflow, or any credential create/update.
- Ask before executing anything that sends messages or emails, spends money, or writes to production data. Prefer test data.
- Never print secrets or credential values in chat or files.

## Conventions
- Workflow names: `[Area] - Purpose` (e.g. `[Sales] - New Lead to CRM`).
- Tag by area and status.
- Optional: save exported workflow JSON and notes in a `workflows/` folder in this project.

## Current mode: docs-only (no n8n API access)
The instance is on an n8n Cloud free trial, so the API is unavailable and there is no `N8N_API_KEY`. The `n8n_*` instance tools (create, update, test, executions) will not work; do not call them.
Workflow: template/node research -> build workflow JSON -> `validate_workflow` (and `validate_node`) -> save to `workflows/<name>.json` -> user imports it in n8n (canvas menu > Import from File, or paste JSON) and connects credentials.
Report manual steps after each build: import, connect credentials, run a manual test, and screenshot for the portfolio.
To re-enable full mode later, add `N8N_API_URL` and `N8N_API_KEY` to the `env` block in `.mcp.json`.
