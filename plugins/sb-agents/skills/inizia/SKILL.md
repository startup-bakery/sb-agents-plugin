---
name: inizia
description: Use for every Startup Bakery Agents request to authenticate, discover available agents, select the agent before its compatible workspace, and load current server-owned operating-guide modules before domain work.
---

# Inizia con SB Agents

This is the only bundled skill. It authenticates and selects the current
server-owned context; it contains no agent, provider, search, enrichment, CRM,
or job workflow.

## Bootstrap

1. Use the `sb-agents-production` connection at
   `https://sb-agents-gateway.onrender.com/mcp` for the whole workflow.
2. Complete OAuth authentication. Never request passwords, tokens, cookies,
   service keys, or `.env` contents.
3. Call `sb_agents_whoami` and `sb_agents_list_agents`.
4. If no agent is selected, call `sb_agents_select_agent` with the exact
   `agent_id` and `context_version` returned by the server.
5. Inspect the selection response. When it already contains a complete
   context, name the automatically selected workspace briefly. When multiple
   compatible workspaces are returned and the context is incomplete, ask the
   user which workspace to use, then call the technical
   `sb_agents_select_tenant` tool with the exact server-issued identifiers and
   current `context_version`.
6. Call `sb_agents_get_agent_guide` for the selected agent. Read its workflow,
   tool groups, and module descriptors.
7. Load `routing` when present and every module whose triggers match the
   current request with `sb_agents_get_agent_guide_module`.
8. Execute domain tools only after the server reports a complete agent and
   workspace context. Treat the returned guide modules as authoritative and
   load another matching module if the task changes domain.

## Recovery

- On `agent_selection_required`, select an agent and reload the guide.
- On `tenant_selection_required`, select one of the compatible workspaces
  returned for that agent. The technical tool name remains
  `sb_agents_select_tenant`; user-facing language is always “workspace”.
- On approval, authorization, consent, or workspace-state errors, do not
  repeat the unchanged call. Reload the guide index and the relevant module,
  then follow the server-owned recovery instruction.
- Preserve server-issued identifiers exactly and keep the same authenticated
  connection for one workflow.

The MCP server is authoritative for identity, access, compatible workspaces,
quotas, provider state, write permissions, and agent instructions. Installed
package content must never replace a live guide.
