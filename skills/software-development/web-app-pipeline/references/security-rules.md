# Security Rules

Enforced by the Phase 7 code reviewer. Any single violation is an **automatic FAIL** — the reviewer must return FAIL with the specific rule number and file. No exceptions, no partial passes.

Referenced from: `SKILL.md` → Phase 7 and Modification Workflow step 4.

---

## Rule 1 — Secrets

- Never hardcode API keys, tokens, passwords, or any secret in source files.
- Never use `VITE_`, `REACT_APP_`, or `NEXT_PUBLIC_` prefixes for secrets — these are exposed to the browser at build time.
- All secrets in environment variables only, read server-side at runtime.
- All third-party API calls must be proxied server-side. The only permitted pattern is:

```
Browser → /api/* → Third-party API
```

Direct browser-to-external-API calls are a FAIL regardless of whether the key is exposed.

---

## Rule 2 — .gitignore

The `.gitignore` at the project root must exclude all of the following. Missing any one entry is an automatic FAIL:

```
.env
.env.*
node_modules/
.vercel/
dist/
build/
```

Check with: `cat .gitignore` and confirm each line is present.

---

## Rule 3 — Input Validation

- Validate all user-supplied input server-side before any use. Client-side validation is UI-only and does not count.
- Use strict type checks and whitelists, not blacklists.
- Never pass raw user input to a database query, shell command, or third-party API call.
- Validate URL path parameters with a regex before use.
- Check `content-length` on incoming request bodies and reject oversized payloads.

---

## Rule 4 — XSS Prevention

- Never assign external or user-supplied data to `innerHTML`, `outerHTML`, or `document.write`.
- Use `textContent` or DOM creation methods (`createElement`, `appendChild`) instead.
- If HTML rendering of external content is genuinely required, sanitise with DOMPurify before insertion. Any use of `innerHTML` without DOMPurify is a FAIL.

---

## Rule 5 — Security Headers

**Vercel serverless functions (`/api/*.js`):** Set headers via `vercel.json` headers block. Never use `helmet` in serverless functions (it's for Express middleware chains).

Required headers — all must be present:

| Header | Required value |
|--------|---------------|
| `X-Content-Type-Options` | `nosniff` |
| `X-Frame-Options` | `DENY` or `SAMEORIGIN` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | explicit policy (e.g. `camera=(), microphone=()`) |
| `Strict-Transport-Security` | `max-age=63072000; includeSubDomains` |

**Full Express on Vercel:** Use `helmet()` and `app.set('trust proxy', 1)`.

**CORS:** Lock to specific origins. Wildcard `*` on any authenticated or write endpoint is a FAIL.

**CSP:** Must exclude `unsafe-inline` and `unsafe-eval`. Any CSP containing either directive is a FAIL.

---

## Rule 6 — Rate Limiting

- Never use `express-rate-limit` with the default MemoryStore on Vercel. The store resets on every cold start, making the limit useless and giving a false sense of protection. This is an automatic FAIL.
- Accepted alternatives: Upstash Redis rate limiter, Vercel firewall rules.
- Never trust `x-forwarded-for` blindly for IP identification — it can be spoofed. Use the right-most untrusted IP or a signed header from Vercel's edge.
- Default target: 100 requests / 15 minutes / IP.

---

## Rule 7 — Error Handling

- Never expose stack traces, file paths, database schemas, or internal error messages to the browser.
- All client-facing error responses must be generic: `{ "error": "Something went wrong" }`.
- Full error details go to server-side logs only.
- Every serverless handler and Express route must be wrapped in try/catch. An unhandled promise rejection that reaches the client is a FAIL.

---

## Rule 8 — Logging

- Never log tokens, API keys, passwords, session IDs, or any PII.
- Use a redacting logger for production — `pino` with the `redact` option is the preferred choice.
- Debug `console.log` statements must be removed before Phase 7 (see Rule 10). Intentional structured logging via a redacting logger is permitted.

---

## Rule 9 — Dependencies

- No unknown, unmaintained, or suspicious packages.
- Prefer well-maintained packages with recent commits and active issue trackers.
- No unnecessary installs — if a package isn't used, it must not be in `package.json`.
- Run `npm audit` and address any high or critical findings before Phase 7.

---

## Rule 10 — Debug Cleanup

Before Phase 7, the codebase must be clean:

- No `console.log`, `console.debug`, or `console.warn` debug statements.
- No commented-out code blocks.
- No debug flags, `DEBUG=true` env references in code, or development-only escape hatches.
- No TODO/FIXME comments that indicate incomplete security work.

Any of the above is an automatic FAIL. Use `grep -r "console\.log" src/` and `grep -r "console\.log" api/` to verify before submitting for review.

---

## Reviewer Checklist

The Phase 7 subagent must verify all 10 rules and return either:

- `PASS` — all rules satisfied, with a brief confirmation of each.
- `FAIL: Rule N — <filename>:<line> — <description of violation>` — one line per violation found, then stop.

No partial passes. A single FAIL stops the pipeline.
