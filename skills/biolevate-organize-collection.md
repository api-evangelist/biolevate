---
name: Organize indexed files into a collection
description: Create a Biolevate collection, add indexed EliseFiles to it, list its contents, and update or delete it — the structured grouping used to scope Question Answering and Extraction jobs.
api: openapi/biolevate-api-original.json
operations: [createCollection, listCollections, getCollection, addFileToCollection, listCollectionFiles, updateCollection, removeFileFromCollection, deleteCollection]
---

# Organize indexed files into a collection (Biolevate / Elise)

Auth: `Authorization: Bearer <pat>`; base URL is your Elise domain. Collections group indexed files so jobs can run against a named set. See `data-model/biolevate-data-model.yml`.

## Steps
1. **Create a collection.** `createCollection` (POST `/api/core/collections`) with a `name` (and optional `description`). Capture the collection UUID. Send `Idempotency-Key` for safe retries.
2. **Add files.** For each indexed EliseFile UUID, `addFileToCollection` (POST `/api/core/collections/{id}/files`).
3. **Inspect.** `listCollectionFiles` (GET `/api/core/collections/{id}/files`) to see membership; `getCollection` (GET `/api/core/collections/{id}`) for metadata; `listCollections` (GET `/api/core/collections`, supports `query`) to find existing ones.
4. **Maintain.** `updateCollection` (PATCH `/api/core/collections/{id}`) to rename/redescribe; `removeFileFromCollection` (DELETE `/api/core/collections/{id}/files/{fileId}`) to drop a file; `deleteCollection` (DELETE `/api/core/collections/{id}`) to remove the collection.

## Use it downstream
Pass the collection UUID as a `collection_ids` value to `createQAJob` or `createExtractionJob` to run AI jobs against the whole grouping instead of listing individual `file_ids`.
