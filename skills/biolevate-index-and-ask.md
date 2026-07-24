---
name: Index a document and ask questions
description: Upload or reference a document in a Biolevate storage provider, index it as an EliseFile, then run an asynchronous Question Answering job and retrieve grounded answers with source annotations.
api: openapi/biolevate-api-original.json
operations: [listProviders, getUploadUrl, confirmUpload, uploadFile, createFile, getFile, createQAJob, getQAJob, getQAJobOutputs, getQAJobAnnotations]
---

# Index a document and ask questions (Biolevate / Elise)

Auth: every request sends `Authorization: Bearer <pat>` (Personal Access Token). Base URL is your per-tenant Elise domain. Ids are UUIDs; job timestamps are epoch-millis. See `conventions/biolevate-conventions.yml` and `errors/biolevate-problem-types.yml`.

## Steps
1. **Pick a storage provider.** `listProviders` (GET `/api/core/providers`) and choose the `providerId` for the backend that holds (or will hold) your document.
2. **Get the file into the provider.** Either upload directly with `uploadFile` (POST `/api/core/providers/{providerId}/items`), or request a presigned URL with `getUploadUrl` (POST `.../items/upload-url`), PUT the bytes to it, then `confirmUpload` (POST `.../items/confirm`). Send an optional `Idempotency-Key` header so retries are safe.
3. **Index it.** `createFile` (POST `/api/core/files`) with the `providerId` + item `key` to create an indexed EliseFile. Capture the returned file UUID. Optionally confirm with `getFile` (GET `/api/core/files/{id}`).
4. **Ask.** `createQAJob` (POST `/api/core/qa/jobs`) with `file_ids` (and/or `collection_ids`) and a `questions[]` list (each with an `id`, `question`, and `answer_type`). Capture the returned `jobId`.
5. **Poll.** `getQAJob` (GET `/api/core/qa/jobs/{jobId}`) until `status` is `SUCCESS` (or `FAILED`). Back off ~2s between polls.
6. **Retrieve.** `getQAJobOutputs` (GET `/api/core/qa/jobs/{jobId}/results`) for answers; `getQAJobAnnotations` (GET `.../annotations`) for the source passages the AI used.

## Error handling
401/403 → invalid token or insufficient permission; 404 → missing provider/file/job; other 4xx/5xx → surface `status_code` + `message`.
