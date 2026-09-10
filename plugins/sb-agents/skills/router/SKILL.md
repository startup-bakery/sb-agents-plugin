---
name: router
description: Use for every Startup Bakery Agents request to authenticate, discover available agents, select the agent before its compatible tenant, and load current server-owned operating-guide modules before domain work.
---

# SB Agents Router

This is the only bundled skill. It contains no provider-specific Sourcing
Agent guide: domain instructions live on the MCP server and are loaded when
needed. The small catalog handoff rules below only prevent a client from
choosing a stale or incompatible write path.

## Bootstrap

1. Use the `sb-agents-production` connection. Its endpoint is
   `https://sb-agents-gateway.onrender.com/mcp`. Keep this connection as the
   single endpoint for the workflow.
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

## Sourcing handoff

Keep one authenticated MCP connection for a sourcing flow and preserve
server-issued identifiers exactly as returned. When a company or directory
response includes `company_selection_ref` and `company_candidate_ids`, pass
the same reference and selected IDs to `clay_people_from_companies`; inspect
the returned `company_scope_statuses` and retry only IDs marked
`retry_recommended: true`.

For ReportAziende or other external companies without a usable native
capability, call `company_domain_find_batch` for at most 50 records. Use only
`results[].domain` from `verified` results as top-level `company_identifiers`
in one `clay_people_search` call, up to 100 domains. An `inferred` domain is
low confidence and needs review; do not turn it into a signed company
selection or invent a replacement reference.

Before domain work, load the live `routing` module and the matching
`companies`/`people` guide modules. If a request returns `selection_ref_invalid`,
do not retry the unchanged payload: refresh the guide and use the verified
domain fallback when appropriate. Search discovery does not require a tenant
public-search admin toggle; normal OAuth, agent/tenant context, and
server-reported capabilities still apply.

## Contact and HubSpot tool routing

The live tool catalog, not this bundle, decides which write path is available.
After a People shortlist is approved, inspect the catalog:

- If both `contact_enrichment_hubspot` and
  `contact_enrichment_hubspot_status` are present, load the live
  `contact-enrichment` and `hubspot` modules. Call
  `hubspot_import_requirements` with `objects: ["companies", "contacts"]`,
  use only its active `company_owner`, `company_sdr_owner`, `contact_owner`,
  and `contact_sdr_owner` values, then obtain explicit approval for the
  credit-consuming/CRM-writing action. Call `contact_enrichment_hubspot` with
  the exact People `selection_ref`, selected `candidate_ids`, requested
  `mode`, the four owner values, and an optional stable `idempotency_key`.
- If the direct pair is absent and the catalog advertises the generic
  enrichment tools, follow the server's quote-first instructions with
  `contact_enrichment_preview`, `contact_enrichment_confirm`, and the same
  `request_id`/`job_id` through the legacy result contract.
- Never choose a provider, call both paths for one selection, or use a tool
  merely because it exists in an installed copy of this plugin. A direct
  operation has no quote or separate confirmation step.

For a pending direct result, preserve `operation.operation_id` and call only
`contact_enrichment_hubspot_status` with that ID. If the nested enrichment
result supplies `retry_after_seconds`, wait that long. Do not call
`sb_agents_job_status`, `sb_agents_job_result`, `sb_agents_job_cancel`,
`contact_enrichment_preview`, or `contact_enrichment_confirm` for the direct
operation. The direct status call may complete the HubSpot write, so it uses
the approval already obtained for the initial action.

## Recovery

- `agent_selection_required`: return to agent selection and reload the guide.
- `tenant_selection_required`: select one of the compatible tenants returned
  for that agent.
- Missing domain instructions: refresh the guide index and load the relevant
  module; do not rely on a remembered or installed Sourcing skill.
- If a domain tool returns an approval, authorization, consent, or tenant-state
  error such as `No approval received`, do not repeat the same call unchanged.
  Reload the selected agent guide and the matching module, then follow the
  server's explicit recovery state. Ask for approval only when the guide marks
  the operation as a write or quota-consuming action; for a read-only call that
  still fails, report the gateway state instead of switching endpoint or
  connection.
- A returned `job_id`: load the job module advertised by the guide and follow
  the same identifiers until the server declares a terminal result.

The MCP's identity, entitlements, tenant compatibility, quotas, provider state,
write permissions and live guide are authoritative. Installed package content
must never override them.
