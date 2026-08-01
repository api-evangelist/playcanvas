---
name: Manage PlayCanvas project assets
description: List, read, create, update, and delete code/text assets in a project branch via the REST API.
api: openapi/playcanvas-rest-openapi.yml
operations: [listAssets, getAsset, getAssetFile, createAsset, updateAsset, deleteAsset]
---

# Manage PlayCanvas project assets

Automate script/html/css/text/shader/json assets in a project branch.

## Auth
`Authorization: Bearer {accessToken}`. Base URL `https://playcanvas.com/api`.
Most asset writes use the **assets** rate-limit class.

## Steps
1. `listAssets` — GET `/projects/{projectId}/assets?branchId={branchId}` with
   `skip`/`limit` (default 16, max 100000) to enumerate assets.
2. `getAsset` — GET `/assets/{assetId}?branchId={branchId}` for a single
   asset's metadata; `getAssetFile` — GET
   `/assets/{assetId}/file/{filename}?branchId={branchId}` for raw file bytes.
3. `createAsset` — POST `/assets` as **multipart/form-data** with fields
   `name`, `projectId`, `branchId` (required), optional `parent`, `preload`,
   `file`, `pow2`. Only `script`, `html`, `css`, `text`, `shader`, `json` types
   are supported. Returns the `Asset` (201).
4. `updateAsset` — PUT `/assets/{assetId}` as **multipart/form-data** with the
   new `file` (required). Same supported types.
5. `deleteAsset` — DELETE `/assets/{assetId}?branchId={branchId}`.
   **Permanent and unrecoverable** unless a checkpoint exists — take one first
   (see the branch-and-checkpoint skill).

## Rules
- `createAsset`/`updateAsset` require `multipart/form-data`, not JSON.
- `branchId` is required on read/delete asset calls.
- Watch the `state` field (`ready`|`processing`|`error`) after writes.
- Errors: `{ "error": "message" }` + HTTP status; honor `429` + `X-RateLimit-*`.
