---
name: Generate a Simi explainer video
description: Submit a whiteboard-style explainer video generation job to Lamina Labs' Simi and poll it until the playback and download URLs are ready.
api: mcp/lamina-labs-mcp.yml
transport: mcp
server: https://api.laminalabs.ai/mcp
operations:
  - simi_submit_video
  - simi_get_video
scopes:
  - jobs:create
  - jobs:read
generated: '2026-07-19'
method: generated
source: https://api.laminalabs.ai/mcp
---

# Generate a Simi explainer video

Simi turns a prompt into a narrated, whiteboard-style teaching video. The surface is a
remote MCP server with exactly two tools. Submission is asynchronous: you submit, you
get a job id back immediately, then you poll.

## Authorize

The server's authorization metadata is at
`https://api.laminalabs.ai/.well-known/oauth-authorization-server`.

- Grant: `authorization_code` (plus `refresh_token`), PKCE `S256` is required.
- Public clients are supported (`token_endpoint_auth_methods_supported` includes `none`)
  and the server exposes dynamic client registration at `/oauth/register`, so a client
  can register itself rather than using a pre-issued secret.
- Request `jobs:create` to submit and `jobs:read` to poll. Send the token as
  `Authorization: Bearer <token>`.

`initialize` and `tools/list` answer without credentials, but every tool call needs a
token. An unauthenticated call returns
`{"error":{"code":"UNAUTHORIZED","message":"Missing credentials"}}`.

## Step 1 — Submit the job

Call `simi_submit_video`.

- `prompt` (required, string, 1–10000 chars) — what the video should teach or explain.
- `duration_seconds` (integer, 1–300, default 60) — note the tool caps at 300 seconds
  even though the product markets longer videos; longer runs are not available here.
- `aspect_ratio` (string, default `16:9`) — e.g. `16:9`, `9:16`, `1:1`.
- `language` (string, default `English`).

It returns a `job` object. Read the job id from it — that is what the next step needs.

**Do not blind-retry this call.** There is no idempotency key, and the provider annotates
this tool `idempotentHint: false`. A retry creates a second job and generates (and bills)
a second video. If a submission times out, poll before resubmitting.

## Step 2 — Poll for readiness

Call `simi_get_video` with `job_id`. It returns:

- `ready` (boolean) — the gate. Keep polling while false.
- `playback` — playback status and signed media URLs.
- `video_url` and `download_url` — both nullable; they stay null until `ready` is true.
- `app_url` — always present, the workspace URL for the job.

`simi_get_video` is `readOnlyHint: true` and `idempotentHint: true`, so it is safe to
retry freely. Back off between polls: the endpoint allows 120 requests per 60-second
window and reports `x-ratelimit-limit`, `x-ratelimit-remaining`, and `x-ratelimit-reset`.
Respect `x-ratelimit-remaining` rather than polling in a tight loop.

The media URLs are **signed**; treat them as short-lived credentials, do not log them,
and fetch or re-poll rather than caching them indefinitely.

## Errors

Errors use a custom envelope, not RFC 9457:
`{"error":{"code":"...","message":"...","details":{...}}}`.

- `UNAUTHORIZED` (401) — no or invalid bearer token. Re-run the OAuth flow.
- `VALIDATION_ERROR` (400) — read `error.details.fieldErrors` for per-field messages,
  most likely a prompt outside 1–10000 chars or a duration outside 1–300.
- `NOT_FOUND` (404) — unknown path, unknown job id, or an unsupported HTTP method
  (this API returns `NOT_FOUND` rather than `405`).

## Caveats

- No enumerated job status vocabulary is published — `ready` is the only documented
  signal, and failure states are undocumented. Set your own wall-clock timeout.
- `openWorldHint` is true on both tools: output is model-generated video. Do not present
  it as verified fact without review.
- The server reports version `0.1.0`; treat the contract as unstable.
