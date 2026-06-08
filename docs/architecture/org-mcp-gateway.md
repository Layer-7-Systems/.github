# Org-wide MCP API Gateway

Status: draft, blocked on scope decision
Owner: unassigned
Originating discussion: [autotask-mcp-retell#23](https://github.com/Layer-7-Systems/autotask-mcp-retell/issues/23)

## Problem

Layer 7 internal tools (voice agents, dashboards, automations, future Workflow Tutor / Vigil clients) call several third-party APIs directly: Autotask, IT Glue, RMM, Guardz. Each tool authenticates separately, logs separately, and there is no single place to answer "who called what, when, with which parameters, and what came back."

This makes audit, rate limiting, credential rotation, abuse detection, and post-incident review harder than it should be, and the problem grows with every new internal tool.

## Proposed direction

A single organization-level MCP server that fronts all third-party API usage and exposes connector-specific tools to internal clients. Phase 1 is read-only. No tool gets write access until audit, identity, and failure behavior are verified end to end.

Properties we want:

- One MCP endpoint, one auth model, one audit log.
- Per-caller identity (which tool, which user or agent, which session).
- Tool schemas discoverable via MCP, not out-of-band docs.
- Audit log captures requester, timestamp, tool name, parameter summary, success/failure, and the downstream API call where applicable.
- Read-only by default. Write tools are opt-in per connector and per caller.

## Non-goals (for now)

- Replacing existing per-repo MCP servers that serve a specific product surface (for example the Retell-facing tools in `autotask-mcp-retell`). Those can either stay as-is or, later, proxy through the org gateway.
- Becoming a general-purpose API gateway for third-party clients. This is internal-only.

## Open questions

These are blockers. The issue cannot be sprinted until these are answered.

1. **Ownership.** Who owns the org MCP server, builds it, and operates it?
2. **Scope.** Which connectors are in Phase 1? Autotask only, or Autotask + IT Glue + RMM + Guardz at once?
3. **Transport and auth.** HTTP + bearer token per caller? mTLS? Something else?
4. **Audit sink.** Where do audit records go? Postgres? S3 + Athena? An existing logging stack?
5. **Identity model.** How do callers identify themselves? Static service tokens, short-lived JWTs from an internal IdP, something tied to GitHub or Google identity?
6. **Hosting.** Same Hostinger VPS family as the Retell MCP server, or somewhere with better isolation?
7. **Relationship to `autotask-mcp-retell`.** Does the Retell MCP server eventually call the org MCP server for its Autotask reads, or does it keep its own Autotask client?

## Phase 1 acceptance criteria (once unblocked)

- Connection details and auth method documented in this repo.
- Tool list and schemas published, ideally generated from code.
- At least one read-only MCP call exercised end to end against a real connector.
- Audit log verified to capture all required fields for that call.
- Documented plan for adding write tools, with an explicit gate that requires sign-off.

## Next step

Decide ownership and Phase 1 scope. Until then this document is the canonical home for the idea, and `autotask-mcp-retell#23` should be closed in favor of an issue against this repo.
