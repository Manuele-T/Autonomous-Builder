# Stack Rules — Vercel

Deployment target for all builds in this pipeline. Know what works and what silently fails before choosing an architecture.

Referenced from: `SKILL.md` → Phase 1 (flag real-time features), Phase 2 (plan the stack), Common Pitfalls.

---

## What Works on Vercel

| Category | Supported |
|----------|-----------|
| Frontends | React, Vue, Svelte, Next.js, Vite |
| Backends | Express via `vercel.json`, Python (FastAPI, Flask, Django) |
| Serverless functions | `/api/*.js`, `/api/*.py` |
| Databases | Neon (Postgres), Supabase, MongoDB Atlas |
| Streaming | SSE (Server-Sent Events), HTTP streaming responses |
| Auth | JWT, session cookies, OAuth via next-auth or similar |

---

## What Never Works on Vercel

These will appear to work locally and silently fail in production. Flag any of these in Phase 1 and change the architecture before writing a line of code.

| Pattern | Why it fails | Alternative |
|---------|-------------|-------------|
| SQLite / any file-based DB | Vercel's filesystem is read-only at runtime; writes silently fail or throw | Neon (default), Supabase |
| Locally hosted databases | No inbound network access from Vercel to a private machine | Cloud DB only |
| WebSockets / Socket.io | Serverless functions close after response; persistent connections are impossible | Pusher, Ably, Partykit |
| Background jobs / scheduled tasks | No persistent process to run them | Vercel Cron (limited), Railway, Render |
| Long-running processes | Hobby plan: 10s max execution. Pro: 60s. Anything longer will be killed | Railway, Render, Modal |
| In-memory shared state | Each invocation may hit a different cold container; memory is never shared across requests | Upstash Redis, Neon |
| `express-rate-limit` with MemoryStore | Store is per-container and resets on cold start — gives zero real protection | Upstash Redis rate limiter, Vercel firewall |

---

## Default Database: Neon

Unless the user specifies otherwise, use **Neon** (free serverless Postgres).

- Connection string stored as `DATABASE_URL` in the project `.env` and in Vercel env vars.
- Use the `@neondatabase/serverless` driver for serverless functions (it uses HTTP, not TCP — works on Vercel's edge runtime).
- Use standard `pg` or `postgres` drivers for Node.js runtimes (not edge).
- Never SQLite, even for prototyping — it will need replacing before deploy.

---

## Real-Time Features

**Flag in Phase 1 if the app requires any of:**

- Live updates pushed from server to client
- Multi-user collaborative state
- Chat or presence indicators
- Live dashboards that update without polling

**Architecture options:**

1. **Pusher or Ably** — managed WebSocket service. Frontend connects to their SDK; your Vercel backend triggers events via their REST API. Stays fully serverless. Recommended for most cases.

2. **Deploy the backend elsewhere** — if the feature requires persistent server state (e.g. a game server, a long-running worker), move the backend to Railway or Render and keep only the frontend on Vercel.

SSE (Server-Sent Events) works on Vercel for one-directional streaming (server → client), but only within the 10s/60s execution limit. Fine for short streams, not for persistent connections.

---

## `vercel.json` Essentials

Every Express-on-Vercel project needs a `vercel.json`. Minimum required shape:

```json
{
  "version": 2,
  "builds": [{ "src": "api/index.js", "use": "@vercel/node" }],
  "routes": [
    { "src": "/api/(.*)", "dest": "/api/index.js" },
    { "src": "/(.*)", "dest": "/index.html" }
  ],
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
        { "key": "Permissions-Policy", "value": "camera=(), microphone=()" },
        { "key": "Strict-Transport-Security", "value": "max-age=63072000; includeSubDomains" }
      ]
    }
  ]
}
```

Route ordering matters: `/api/*` must come before the catch-all `(.*)`. Reversing them causes the catch-all to swallow API requests — see `node-express-backend` skill for the full wildcard crash pitfall.

---

## Vite + Vercel Serverless Pattern

The canonical proxy architecture for a Vite frontend with a Vercel serverless API:

```
/
├── src/              ← Vite React frontend
├── api/
│   └── index.js      ← Express app wrapped for Vercel
├── vercel.json       ← routes + security headers
├── vite.config.js    ← dev proxy: /api → localhost:3001
└── .env              ← secrets (never committed)
```

In development, Vite proxies `/api/*` to a local Express server. In production, Vercel routes `/api/*` to the serverless function. The frontend code never changes between environments.

For full Vite + Vercel deployment gotchas (scaffolding into non-empty dirs, `.env.*` gitignore trap, `vercel.json` rewrite ordering, env var push to Vercel after deploy), see the `node-express-backend` skill → `references/vite-vercel-deploy.md`.

---

## External API Keys — Supported Services

Keys stored in project `.env` and in Vercel env vars. All calls proxied server-side.

| Service | Env var name | Notes |
|---------|-------------|-------|
| RapidAPI | `RAPIDAPI_KEY` | One master key per account, works across all subscribed APIs |
| OpenWeatherMap | `OPENWEATHER_API_KEY` | Use `data/2.5` endpoints; hard-cap at 600 calls/day on free plan |
| Neon | `DATABASE_URL` | Full connection string including credentials |
| Upstash Redis | `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN` | For rate limiting and shared state |

For full OpenWeatherMap integration patterns (API quirks, forecast aggregation, icon handling, XSS-safe rendering), see the `node-express-backend` skill → `references/openweathermap-integration.md`.
