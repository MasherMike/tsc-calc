# Backend Proxy Design Spec
**Project:** OCS Nutrition Calculator
**Date:** 2026-03-14
**Status:** Approved

---

## Purpose

A lightweight Node.js proxy server deployed on Railway that sits between the frontend (GitHub Pages) and the Anthropic API. Keeps the API key off the client and off GitHub.

---

## Repository

- **New GitHub repo:** `ocsc-proxy` (separate from `tsc-calc`)
- **Deploy target:** Railway (free tier, $5/mo credit, no cold starts)

---

## Architecture

```
Browser (GitHub Pages: mashermike.github.io)
    │
    │  POST /api/claude
    │  Header: X-Proxy-Token: <shared_secret>
    │  Body: Anthropic Messages API payload (JSON, max 50kb)
    │
    ▼
Railway — ocsc-proxy (Node.js 18+ + Express)
    │  1. Validate X-Proxy-Token → 403 if wrong
    │  2. Inject headers: Authorization, anthropic-version, Content-Type
    │  3. Forward body to api.anthropic.com/v1/messages (30s timeout)
    ▼
Anthropic API
    │
    ▼  Response (pass-through status + body)
Railway → Browser
```

---

## File Structure

```
ocsc-proxy/
  index.js        # Express server (~70 lines)
  package.json    # dependencies: express, cors; engines: node >= 18
  .env.example    # template: ANTHROPIC_API_KEY=, PROXY_TOKEN=
  .gitignore      # node_modules, .env
  .nvmrc          # 18
  README.md       # setup + Railway deploy instructions
```

---

## Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/api/claude` | Proxy to Anthropic Messages API |
| `GET`  | `/health`     | Railway health check — returns `200 { status: 'ok' }` |

---

## Security Model

| Secret | Storage | Usage |
|--------|---------|-------|
| `ANTHROPIC_API_KEY` | Railway env var | Injected into `Authorization: Bearer` header on every outbound request |
| `PROXY_TOKEN` | Railway env var + hardcoded constant in frontend JS | Validated on every inbound request via `X-Proxy-Token` header |

- CORS allowed origins: `https://mashermike.github.io`, `http://localhost:3000`, `http://localhost:5500`, `http://127.0.0.1:5500` (common Live Server ports). During Railway testing before frontend is finalized, temporarily allow the `*.up.railway.app` URL — remove before production.
- `PROXY_TOKEN` in frontend JS is a known trade-off — stops casual abuse, not a true secret
- Request body capped at **50kb** via `express.json({ limit: '50kb' })` to limit token cost exposure

---

## Startup Behavior

If `ANTHROPIC_API_KEY` or `PROXY_TOKEN` is missing at startup, the server **must throw and exit** (`process.exit(1)`) with a clear error message. Do not start in a degraded state.

---

## Error Handling

| Condition | Response |
|-----------|----------|
| Missing or wrong `X-Proxy-Token` | `403 { error: 'Forbidden' }` |
| Malformed / oversized JSON body | `400 { error: 'Bad Request' }` |
| Anthropic returns an error | Forward Anthropic's HTTP status code and JSON body verbatim |
| Upstream request exceeds 30s timeout | `504 { error: 'Upstream timeout' }` |
| Unexpected server error | `500 { error: 'Internal Server Error' }` |

---

## Headers Injected by Proxy (outbound to Anthropic)

```
Authorization: Bearer <ANTHROPIC_API_KEY>
anthropic-version: 2023-06-01
Content-Type: application/json
```

---

## Environment Variables (Railway)

```
ANTHROPIC_API_KEY=sk-ant-...       # Set in Railway dashboard — never commit
PROXY_TOKEN=<random 32-char string> # Set in Railway dashboard — never commit
# Do NOT set PORT — Railway injects it automatically
```

The server must listen on `process.env.PORT || 3000` — never hardcode port 3000, or Railway will fail to bind.

---

## Node Version

- **Require Node 18+** — uses native `fetch` (no `node-fetch` dependency)
- Pin in `package.json` engines field: `"node": ">=18"`
- `.nvmrc` set to `18`

---

## Frontend Integration

```js
const PROXY_URL = 'https://<railway-url>';
const PROXY_TOKEN = '<same value as Railway PROXY_TOKEN env var>';

const response = await fetch(`${PROXY_URL}/api/claude`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Proxy-Token': PROXY_TOKEN
  },
  body: JSON.stringify({
    model: 'claude-haiku-4-5-20251001',
    max_tokens: 1024,
    messages: [...]
  })
});
```

The body is identical to the Anthropic Messages API — the proxy is transparent.

---

## Out of Scope (this spec)

- PDF text extraction endpoint (`/api/parse-pdf`) — Round 2, Step 2
- Per-user rate limiting
- Request logging / analytics
- Auth beyond shared token

---

## Testing Plan

1. **Local:** create `.env` with `ANTHROPIC_API_KEY` and `PROXY_TOKEN` → `node index.js` → `curl -X POST localhost:<PORT>/api/claude -H "X-Proxy-Token: ..." -H "Content-Type: application/json" -d '{"model":"claude-haiku-4-5-20251001","max_tokens":10,"messages":[{"role":"user","content":"ping"}]}'`
2. **Health check:** `curl http://localhost:<PORT>/health` → `{ status: 'ok' }`
3. **Auth rejection:** send wrong `X-Proxy-Token` → verify `403 { error: 'Forbidden' }`
4. **Oversized body:** send body > 50kb → verify `400`
5. **Deployed:** test from browser console on live site; check Railway logs for request activity
6. **Timeout simulation:** not required for initial deploy; verify Railway surfaces a `504` if Anthropic hangs beyond 30s

---

## Railway Setup Steps

1. Create Railway account at railway.app (sign up with GitHub)
2. Create new GitHub repo: `ocsc-proxy`
3. Push `index.js` and `package.json` to repo
4. Railway → New Project → Deploy from GitHub repo → select `ocsc-proxy`
5. Railway → Variables → add `ANTHROPIC_API_KEY` and `PROXY_TOKEN`
6. Railway auto-deploys; note the generated public URL (e.g. `ocsc-proxy.up.railway.app`)
7. Add proxy URL and `PROXY_TOKEN` value as constants in `index.html`
