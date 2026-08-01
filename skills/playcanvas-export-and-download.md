---
name: Export a project and download a self-hostable app
description: Kick off a project ZIP export or a self-hostable app build and retrieve the download URL by polling the job.
api: openapi/playcanvas-rest-openapi.yml
operations: [listScenes, downloadApp, exportProject, getJob, getPrimaryApp]
---

# Export a project and download a self-hostable app

Two async build flows: full project ZIP export, and a packaged app download.

## Auth
`Authorization: Bearer {accessToken}`. Base URL `https://playcanvas.com/api`.
Both `downloadApp` and `exportProject` use the **strict** rate-limit class
(as low as 5 requests/min on free accounts) — pace calls.

## App download flow
1. `listScenes` — GET `/projects/{projectId}/scenes?branchId={branchId}` to get
   the scene ids to include (the first id becomes the app's initial scene).
2. `downloadApp` — POST `/apps/download` with JSON
   `{ "project_id", "name", "scenes": [...], "format": "static"|"npm", ... }`.
   Returns a **Job** (201).
3. `getJob` — poll GET `/jobs/{id}` until `status` is `complete`; read the
   package download URL from `data`.

## Project export flow
1. `exportProject` — POST `/projects/{projectId}/export` with optional
   `{ "branch_id": "..." }`. Returns a **Job** (201).
2. `getJob` — poll until `complete`; the ZIP download URL is in `data.url`.

## Rules
- Both operations are async — always poll `getJob`; never assume the response
  body already contains the artifact.
- `format` defaults to `static`; use `npm` for a Vite-based project.
- `getPrimaryApp` (GET `/projects/{projectId}/app`) returns the currently
  published primary app if you just want its live metadata/URL.
- Respect the strict limit — `429` returns `X-RateLimit-Reset` (UTC epoch s).
