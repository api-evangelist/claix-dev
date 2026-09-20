---
generated: '2026-09-19'
method: generated
name: Persist a document and ask it questions
description: Extract with a context-window schema so the document is kept in Claix memory, then ask it up to five questions per call, read its raw text, and delete it when done.
api: openapi/claix-dev-openapi.yml
operations: [createSchema, pdfToJson, getDocument, documentContextAsk, deleteDocument]
source: >-
  Grounded in the MCP prompt workflow.window-context-ask and
  https://www.claix.dev/documentation/window-context; operationIds verified in
  openapi/claix-dev-openapi.yml. Note the three context routes live at the origin root, not under
  /api (see conventions base_url_note).
---

# Persist a document and ask it questions

## Auth
- Same `x-api-key` as every other call. Documents are readable only by the account that created them.

## Steps
1. **Make the schema persist documents** — `createSchema` with `window_context: true` and `window_time` (minutes: 5, 10, 15, 30, 45, 60, 90, 120, 180, 240, 360, 480, 720, 1440 — or `"infinity"`, which needs Persistent Mode on the account).
2. **Extract** — e.g. `pdfToJson` (`POST /api/pdf-json`) with that `schema_id`. Keep the `document_id` (UUID) from the response; it is the handle for everything below and it expires after `window_time`.
3. **Read the raw content (optional, free)** — `getDocument` (`GET https://claix.dev/get-document/{document_id}`) returns `content`, `file_name`, `schema_id`, `processed_at`.
4. **Ask** — `documentContextAsk` (`POST https://claix.dev/document-context/{document_id}`, JSON `{ "questions": [...] }`, 1–5 questions, ≤400 chars each). Response `user_ask[]` echoes your questions; `ia_response[]` is aligned 1:1 and is **`null` when the fact is not in the document** — branch on null, do not treat it as an empty string.
5. **Delete when finished** — `deleteDocument` (`DELETE https://claix.dev/delete-document/{document_id}`, no body). Free and **irreversible**.

## Reversibility
- Step 5 cannot be undone; the document also vanishes on its own at `window_time`. The extraction charge is never reversed. See `conventions/claix-dev-conventions.yml#reversibility`.

## Errors
- **400** malformed `document_id` / questions rules broken; **404** document missing, not yours, or **expired**; **502** AI failure (retry); **405** wrong verb. See `errors/claix-dev-problem-types.yml`.

## Cost
- `documentContextAsk` €0.03 per successful call (up to 5 questions); `getDocument` and `deleteDocument` are free.
