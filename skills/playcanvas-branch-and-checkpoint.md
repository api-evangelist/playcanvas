---
name: Branch and checkpoint a PlayCanvas project
description: Create a version-control branch of a project and take a checkpoint (snapshot) of it, waiting on the async job.
api: openapi/playcanvas-rest-openapi.yml
operations: [listBranches, createBranch, createCheckpoint, getJob, getCheckpoint]
---

# Branch and checkpoint a PlayCanvas project

Use this to programmatically fork a project's version history and snapshot it.

## Auth
Send `Authorization: Bearer {accessToken}` on every request (token from the
Organization Account page). HTTPS only. Base URL `https://playcanvas.com/api`.

## Steps
1. `listBranches` — GET `/projects/{projectId}/branches` to find the
   `sourceBranchId` you want to branch from (use the branch whose `id` you need;
   cursor-paginate with `?skip={lastBranchId}` while `pagination.hasMore`).
2. `createBranch` — POST `/branches` with JSON body
   `{ "name", "projectId", "sourceBranchId" }` (optionally `sourceCheckpointId`).
   Returns the new `Branch` (201).
3. `createCheckpoint` — POST `/checkpoints` with
   `{ "projectId", "branchId", "description" }`. This returns a **Job** (201),
   not the checkpoint.
4. `getJob` — poll GET `/jobs/{id}` until `status` is `complete` (or `error`).
   On complete, `data` holds the created checkpoint.
5. `getCheckpoint` (optional) — GET `/checkpoints/{id}` to re-read the snapshot.

## Rules
- `createCheckpoint` is async — always poll `getJob`; never assume immediate
  completion.
- `description` is required and must be non-empty (<= 10,000 chars).
- Errors return `{ "error": "message" }` with the HTTP status; `429` carries
  `X-RateLimit-*` headers — back off until `X-RateLimit-Reset` (UTC epoch s).
- No idempotency key exists; avoid retrying create calls blindly on timeouts —
  re-list to check whether the branch/checkpoint already exists.
