---
generated: '2026-09-19'
method: generated
name: Extract with Agent mode reasoning
description: Use an Agent-mode schema to get both the extracted records and a second, reasoned layer of answers (agent_data) about the document in one call.
api: openapi/claix-dev-openapi.yml
operations: [createSchema, agentPdfToJson, agentExcelToJson, agentDocToJson, agentImgToJson, agentTxtToJson]
source: >-
  Grounded in the MCP prompt workflow.agent-document-analysis and
  https://www.claix.dev/documentation/agent/txt-to-json; operationIds verified in
  openapi/claix-dev-openapi.yml. The /agent/* routes live at the origin root (https://claix.dev/agent/...), not under /api.
---

# Extract with Agent mode reasoning

## Auth
- `x-api-key`.

## Steps
1. **Create an Agent-mode schema** — `createSchema` with `is_agent_mode: true`, the usual `schema_definition`, and an `agent_definition` whose parameters are typed `boolean | string | closed | integer`; a `closed` parameter needs a non-empty `options[]`. Optional `resumen_agent` (≤500 chars) adds an instruction for the reasoning phase. `json-excel` schemas cannot be Agent mode.
2. **Call the agent route that matches the input** — `agentPdfToJson` (`POST https://claix.dev/agent/pdf-json`), `agentExcelToJson` (`/agent/excel-json`), `agentDocToJson` (`/agent/doc-json`), `agentImgToJson` (`/agent/img-json`), or `agentTxtToJson` (`/agent/txt-json`, JSON `content`). Same multipart `file` + `schema_id` (+ optional `space_id`) as the standard routes and the same size limits.
3. **Read two payloads** — `data[]` (the schema extraction) and `agent_data` (one value per `agent_definition` parameter).

## Errors
- **400** the schema is not Agent mode or `agent_definition` is invalid; **422** nothing extractable in the first (extraction) phase; **502** failure in either the extraction or the agent phase — retry. See `errors/claix-dev-problem-types.yml`.

## Cost
- Billed at the same per-document rate as the standard route (€0.15 / €0.10) — see `plans/claix-dev-plans-pricing.yml`.
