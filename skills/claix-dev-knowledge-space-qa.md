---
generated: '2026-09-19'
method: generated
name: Group documents in a knowledge space and query them together
description: Create a knowledge space, attach persisted documents to it at extraction time, then ask questions that span every document in the space.
api: openapi/claix-dev-openapi.yml
operations: [createSpace, pdfToJson, excelToJson, spaceContextAsk, deleteSpace]
source: >-
  Grounded in the documented MCP prompt workflow.space-context-ask (https://www.claix.dev/documentation/mcp
  section 7) and https://www.claix.dev/blog/knowledge-spaces-ai-agents; operationIds verified in
  openapi/claix-dev-openapi.yml.
---

# Group documents in a knowledge space and query them together

## Auth
- `x-api-key`. A space, its documents and the schema must all belong to the same account.

## Steps
1. **Create the space** — `createSpace` (`POST https://claix.dev/create-space`, JSON `{ "name": "..." }`). Expect **201** and keep `space.space_id`. Free.
2. **Extract into it** — any extraction (`pdfToJson`, `excelToJson`, `docToJson`, `imgToJson`, `txtToJson` or the agent variants) with the extra multipart field `space_id`. This only takes effect when the schema has `window_context` enabled — that is what makes the document persist. Repeat for every document.
3. **Ask across the space** — `spaceContextAsk` (`POST https://claix.dev/space-context/{space_id}`, JSON `{ "questions": [...] }`, 1–5 questions, ≤400 chars). Same response shape as a single-document ask: `user_ask[]`, `ia_response[]` with `null` where no document carries the answer. A **400** with a space that "has no current documents with content" means every document has expired or none was attached.
4. **Tear down** — `deleteSpace` (`DELETE https://claix.dev/delete-space/{space_id}`, no body). Free and **irreversible**; the docs do not state what happens to documents still inside it, so delete them individually first (`deleteDocument`) if that matters.

## Notes
- Documents still expire individually at their schema's `window_time`; a space does not extend them.
- Over A2A the same flow is the skills `create-space` → `extract-*` (with `space_id`) → `query-space`; over MCP the space tools are documented but were **not** in the live `tools/list` on 2026-09-19 (`mcp/claix-dev-mcp.yml`).

## Errors
- **404** space not found / not yours; **400** malformed UUID or question rules; **502** AI failure (retry). See `errors/claix-dev-problem-types.yml`.
