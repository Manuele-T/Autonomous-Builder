# Commit Message and README Rules

Standards for version control output at Phase 9. Both sections are mandatory — a missing README or a bad commit message means the build is not done.

Referenced from: `SKILL.md` → Phase 9 (first push and subsequent pushes).

---

## Commit Message Rules

### Format

```
<type>(<scope>): <short title>

- bullet: what changed
- bullet: key technical decision
- bullet: security or infra note if relevant
```

### Types

| Type | When to use |
|------|------------|
| `feat` | New user-facing feature or capability |
| `fix` | Bug fix or correction |
| `refactor` | Code restructure with no behaviour change |
| `chore` | Tooling, dependencies, config |
| `security` | Security hardening, header changes, auth updates |
| `docs` | README, comments, documentation only |

### Rules

- **Title:** lowercase, imperative mood ("add auth", not "added auth" or "adding auth"), under 72 characters.
- **Body:** always 2–4 bullet points. Never leave the body empty.
- **Scope:** the folder or module affected (e.g. `api`, `frontend`, `auth`, `db`). Keep it short.
- **Never use:** "initial commit", "fix stuff", "update", "wip", "misc", or any vague placeholder.
- One commit per logical change. Don't bundle unrelated changes into one commit.

### Examples

**First push — new build:**
```
feat(app): add weather dashboard with OWM proxy

- React + Vite frontend with city search and 5-day forecast
- Express serverless function proxies all OpenWeatherMap calls
- Upstash Redis rate limiter: 100 req/15 min/IP
- Security headers set via vercel.json; no secrets in frontend bundle
```

**Subsequent push — modification:**
```
fix(api): correct CORS origin after Vercel deploy

- Updated allowed origin from localhost placeholder to live URL
- Redeployed to Vercel; confirmed preflight requests now succeed
```

**Security-only change:**
```
security(api): add missing Content-Security-Policy header

- Added CSP to vercel.json headers block for /api/* routes
- Excluded unsafe-inline and unsafe-eval
```

---

## README Rules

Written in Phase 9 **before** the first commit. The README must be in the repo on the first push — never added in a follow-up commit.

### Required Sections (in this order)

**1. Project name and one-line description**
The H1 heading is the project name. The line below it is a single sentence describing what the app does and for whom.

**2. Live demo**
A clickable link to the real Vercel URL. Never a placeholder like `https://your-app.vercel.app`. If the URL isn't confirmed yet, the README is not ready.

```markdown
## Live Demo
[https://the-spanish-masterpiece.vercel.app](https://the-spanish-masterpiece.vercel.app)
```

**3. Stack**
A brief list: frontend framework, backend runtime, database, deployment platform. One line each.

**4. Features**
3–6 bullet points describing what the app actually does. Written from the user's perspective, not the technical implementation's.

**5. Architecture**
One paragraph. Cover: how the frontend and backend are separated, how secrets are protected (proxy pattern), and any notable infrastructure decisions (e.g. why Neon over SQLite, why Pusher over WebSockets). Mandatory — do not omit.

**6. Environment variables**
A table with three columns: `Variable`, `Description`, `Where to get it`. No real values, ever.

| Variable | Description | Where to get it |
|----------|-------------|-----------------|
| `OPENWEATHER_API_KEY` | OpenWeatherMap API key | openweathermap.org/api |
| `DATABASE_URL` | Neon Postgres connection string | console.neon.tech |

**7. Local development**
Exact shell commands to clone the repo, install dependencies, add env vars, and start the dev server. Assume the reader has Node installed but nothing else.

```bash
git clone https://github.com/manue/<repo>.git
cd <repo>
npm install
cp .env.example .env   # fill in your values
npm run dev
```

**8. Security notes**
One paragraph. Cover: proxy architecture (no keys in the browser), which security headers are set and where, and rate limiting approach. Mandatory — do not omit.

### Quality Rules

- Professional English. No filler phrases ("This is a project that...", "Feel free to...").
- No internal paths (e.g. `/home/manue/projects/...`).
- No references to the build pipeline, phases, or Hermes.
- Under 150 lines total.
- After any significant modification, update the **Live demo** link and the **Features** section if the change is user-visible.

### README Checklist

Before the first commit:

- [ ] H1 heading is the project name
- [ ] One-line description immediately below H1
- [ ] Live demo section contains the confirmed `https://` URL
- [ ] Stack section present
- [ ] Features: 3–6 bullets from the user's perspective
- [ ] Architecture paragraph written (not omitted)
- [ ] Environment variables table — no real values
- [ ] Local development commands are exact and runnable
- [ ] Security notes paragraph written (not omitted)
- [ ] No internal paths or pipeline references
- [ ] Under 150 lines
