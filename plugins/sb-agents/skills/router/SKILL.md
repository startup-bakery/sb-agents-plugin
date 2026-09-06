---
name: router
description: Use for every Startup Bakery Agents request to authenticate, discover available agents, select the agent before its compatible tenant, and load current server-owned operating-guide modules before domain work.
---

# SB Agents Router

This is the only bundled skill. It contains no Sourcing Agent workflow: agent
instructions live on the MCP server and are loaded when needed.

## Bootstrap

1. Use the `sb-agents-production` connection. Its endpoint is
   `https://sb-agents-gateway.onrender.com/mcp`. Never fall back to S086 or to a
   legacy `sourcing-agent-test` connection.
2. Complete OAuth authentication. Never ask for passwords, tokens, cookies,
   service keys, or `.env` contents.
3. Call `sb_agents_whoami` and `sb_agents_list_agents`. The server is the
   authority for identity, access and setup-required agents.
4. If an agent is not selected, call `sb_agents_select_agent` with the exact
   `agent_id` and `context_version` returned by the server.
5. Call `sb_agents_get_agent_guide` for the selected agent. Read its workflow,
   tool groups and module descriptors.
6. Call `sb_agents_get_agent_guide_module` for `routing` when present and for
   every module whose triggers match the request. Treat the returned content as
   the current operating instructions; load another module if the task changes
   domain.
7. Inspect the response from `sb_agents_select_agent`. If it already contains
   a complete `context`, the server selected the only compatible tenant: name
   it briefly and do not ask the person to choose it again. If `context` is
   absent and more than one compatible tenant is returned, ask in chat which
   one to use, then call `sb_agents_select_tenant` with the exact `agent_id`,
   `organization_id`, `tenant_id` and current `context_version`.
8. Execute domain tools only when the server reports a complete agent and
   tenant context. Browser OAuth authenticates the person and grants access;
   it never selects an agent or tenant.

## Recovery

- `agent_selection_required`: return to agent selection and reload the guide.
- `tenant_selection_required`: select one of the compatible tenants returned
  for that agent.
- Missing domain instructions: refresh the guide index and load the relevant
  module; do not rely on a remembered or installed Sourcing skill.
- A returned `job_id`: load the job module advertised by the guide and follow
  the same identifiers until the server declares a terminal result.

The MCP's identity, entitlements, tenant compatibility, quotas, provider state,
write permissions and live guide are authoritative. Installed package content
must never override them.
