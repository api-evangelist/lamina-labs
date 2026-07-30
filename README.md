# Lamina Labs

Lamina Labs is a San Francisco company (Y Combinator, Spring 2026) building near-real-time
video infrastructure for large language models. Its product **Simi** turns prompts,
documents, and AI-generated answers into whiteboard-style teaching videos in seconds —
handling script writing, illustration, animation, and narration automatically, in over 80
languages — for training, onboarding, product walkthroughs, sales, marketing, and
education.

- Website: https://www.laminalabs.ai/
- Simi app: https://app.laminalabs.ai/simi

## Developer surface

Lamina Labs publishes **no OpenAPI, no API reference, and no SDKs**. Its machine-readable
contract is a **public remote MCP server**:

- `https://api.laminalabs.ai/mcp` — server `lamina-simi` v0.1.0, MCP protocol 2025-06-18
- Two tools: `simi_submit_video` (submit, `jobs:create`) and `simi_get_video` (poll,
  `jobs:read`)
- Tools only — `resources/list` and `prompts/list` return "Unsupported MCP method"

Authorization is an OAuth 2.1-shaped server with RFC 8414 metadata, mandatory PKCE
(S256), refresh tokens, public-client support, and **dynamic client registration** — the
MCP remote-server authorization profile, which lets an agent register itself.

## Artifacts

| Artifact | Method |
|---|---|
| `mcp/` | searched — live tool schemas |
| `well-known/` | searched — OIDC + RFC 8414 metadata (both 200) |
| `authentication/`, `scopes/` | searched — from authorization server metadata |
| `errors/`, `rate-limits/`, `conventions/`, `conformance/`, `lifecycle/` | probed |
| `security/domain-security` | probed |
| `data-model/`, `agentic-access/`, `skills/`, `llms/` | derived / generated |
| `packages/` | searched — no first-party SDK exists |

Backed by: y-combinator
