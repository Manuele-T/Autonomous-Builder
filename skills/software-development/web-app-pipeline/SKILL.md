---
name: web-app-pipeline
description: Use when the user asks to build, create, scaffold, deploy, or modify a web app, site, dashboard, or API. Runs a deterministic approval-driven pipeline — Phase 1 through Phase 10 for new builds, Modification Workflow for changes — and deploys to Vercel with GitHub version control. All work is confined to /home/manue/projects.
version: 1.2.0
author: manue
license: private
platforms: [linux]
metadata:
  hermes:
    tags: [development, build, deploy, vercel, github, web-app]
    related_skills: [node-express-backend, subagent-driven-development, systematic-debugging, requesting-code-review]
required_environment_variables:
  - name: GH_TOKEN
    prompt: GitHub personal access token
    help: Classic token with repo scope; set via hermes setup
    required_for: GitHub push (Phase 9)
  - name: VERCEL_TOKEN
    prompt: Vercel account token
    help: Full-account token from vercel.com/account/tokens; set via hermes setup
    required_for: Vercel deploy (Phase 8)
---

# Web App Pipeline — SKILL.md

A deterministic, approval-driven pipeline for building, deploying, and shipping web apps. Follow it phase by phase. Never skip a gate, never self-advance, never improvise.

## When to Use

- User asks to build, create, scaffold, or deploy a web app, site, dashboard, or API
- User asks to modify an existing project under `/home/manue/projects`

**Don't use for:** scripts, CLI tools, data pipelines, or anything not a deployable web app.

- **NEW BUILD** → run Phase 1 through Phase 10.
- **MODIFICATION** → follow the Modification Workflow only.
- If unsure which, ask one short question first.

## Prerequisites

- Verify cwd is `/home/manue/projects` before anything; if not, `cd /home/manue/projects`. Re-verify at the start of every build and modification.
- `GH_TOKEN` and `VERCEL_TOKEN` must be present in the environment. Use them as `$GH_TOKEN` and `$VERCEL_TOKEN`. Never run `gh auth login` or `vercel login` — token-only auth, always.
- If either token is missing, note it early but only stop when the token is actually needed for the upcoming phase (GH_TOKEN for Phase 9, VERCEL_TOKEN for Phase 8). Never print or log a token value.

## Pipeline (NEW BUILD)

### Phase 1 — Clarification
- **Ask the app type FIRST:** Is it (a) a static frontend SPA (no backend, no database, pure client-side), (b) a frontend with a simple serverless API, or (c) a full-stack app with API + database? Defaulting to (c) when the user asked for something simple will waste everyone's time. "No API, no DB" is a valid and common answer — treat it as the default until the user says otherwise.
- For simple/MVP projects, ask all questions in a single round. Reserve multiple rounds for complex projects where later answers depend on earlier choices.
- If static SPA: skip all DB/API/secret questions. Move straight to naming and GitHub visibility.
- Confirm GitHub visibility (default: private).
- Identify every secret the app will need and note them for the plan.
- Flag any real-time feature now (see `references/stack-rules.md`) — it changes the architecture.
- Create nothing yet. Do not proceed until clarification is answered.

### Phase 2 — Planning
- Derive a kebab-case folder name from the request.
- Create the folder: `mkdir -p /home/manue/projects/<project-folder>`.
- If the app has secrets (API keys, DB credentials, auth tokens), write a placeholder env file inside it: `NAME=YOUR_VALUE_HERE`. If it's a static SPA with no secrets, **skip the .env entirely** — a blank placeholder .env is just noise.
- Write `PLAN.md` inside the folder. Include: app name, stack, folder structure, build steps, validation steps, required secrets, and a Build Instructions section (one component at a time — never generate all files in one pass).

### Phase 3 — Approval Gate
- Tell the user where `PLAN.md` is.
- If an `.env` file exists (app has secrets), also tell them where it is and to replace every `YOUR_VALUE_HERE` with a real key. If no `.env` exists (static SPA), skip the env-file step.
- Ask them to: (a) review `PLAN.md`, and (b) if applicable, fill in the `.env` values. Then reply YES.
- Before continuing, if a `.env` file exists, confirm no placeholder values remain. If any still read `YOUR_VALUE_HERE`, stop.
- **Do NOT continue on any reply other than YES. This is a hard stop.**

### Phase 4 — Architecture
- Implement the approved plan exactly. No redesigns, no unrequested features.

### Phase 5 — Build
- All files go inside the project subfolder.
- Use `npm ci` if `package-lock.json` exists, otherwise `npm install`.
- Prefer simple MVP implementations. Do not overwrite existing files unnecessarily.

### Phase 6 — Validation
- Run build and test commands from inside the subfolder.
- Report pass/fail with specific errors. On FAIL, stop — do not proceed to review or deploy.

### Phase 7 — Code Review Gate
- Use `delegate_task` to spin up a reviewer subagent. Pass it the full absolute path of the subfolder.
- Instruct it to check every item in `references/security-rules.md`.
- Reviewer returns **PASS** or **FAIL** with specific issues.
- On FAIL: report to user and stop. On PASS: continue to Phase 8.

### Phase 8 — Deploy (Vercel)
Deploy BEFORE GitHub. The user must verify the live site first.
- Verify you are inside the project subfolder.
- Link if needed: `vercel link --yes --token $VERCEL_TOKEN`.
- Add each required env variable to Vercel production using `vercel env add`, one per variable, skipping any that already exist. Never print or log the values.
- Deploy: `vercel --prod --yes --token $VERCEL_TOKEN`.
- Capture the URL from stdout. Confirm it is non-empty and starts with `https://`. If not, treat as failed.
- CORS update: check the proxy file for hardcoded origins. If a placeholder differs from the real URL, update it and redeploy.
- Ask: "Live at [URL]. Test it and reply YES to push to GitHub, or describe issues to fix."
- **Do NOT proceed to Phase 9 until the user replies YES.**

### Phase 9 — Version Control (GitHub)
Only after the user replies YES to the live site.
- Check for an existing repo: `git rev-parse --git-dir`.

**First push:**
1. `git init && git branch -M main`
2. Write `README.md` (see `references/commit-readme-rules.md`) with the confirmed live URL.
3. `git add .`
4. Commit using the format in `references/commit-readme-rules.md`.
5. `gh repo create <folder> --private/--public --source=. --remote=origin --push`

**Subsequent push:**
1. `git add .` → commit → `git push origin main`

- Capture repo URL: `gh repo view --json url -q .url`.
- On any failure, report the exact error and stop.

### Phase 10 — Report
- Summarise what was built, the subfolder path, and build/review verdicts.
- Provide the live Vercel URL and the GitHub repo URL.
- List any manual next steps.

## Modification Workflow
1. Confirm the exact project folder with the user. `cd` into it.
2. Make only the requested changes. No rebuilds, no unrequested features.
3. Run validation (Phase 6). On FAIL, stop.
4. Run code review (Phase 7, using `references/security-rules.md`). On FAIL, stop.
5. If CORS origins changed, update the proxy file before redeploying.
6. Redeploy to Vercel (Phase 8 path). Report the live URL and ask the user to verify.
7. Do NOT push to GitHub until the user confirms the live site is correct.
8. Push to GitHub (Phase 9 subsequent-push path) with a conventional commit.
9. Update `README.md` if the change is user-visible.
10. Report final URLs (Phase 10 format).

## Verification Checklist

Before reporting a build complete, every item must be true:

- [ ] Validation (Phase 6) passed
- [ ] Phase 7 code review returned PASS against `references/security-rules.md`
- [ ] Live Vercel URL resolves over `https://`
- [ ] User replied YES confirming the live site
- [ ] GitHub push succeeded and repo URL was captured
- [ ] `README.md` contains the real live URL (not a placeholder)

If any item is unchecked, the build is not done.

## Common Pitfalls

1. **Self-advancing past a gate.** Phase 3 and Phase 8 both require explicit YES from the user. "Looks good" or silence is not YES.
2. **Deploying before validation passes.** Phase 6 must be green before Phase 7 runs. Phase 7 must PASS before Phase 8 runs.
3. **SQLite on Vercel.** File-based DBs don't work on serverless. Default is Neon. See `references/stack-rules.md`.
4. **Hardcoding secrets.** All keys via env vars, all third-party calls proxied server-side. See `references/security-rules.md`.
5. **Leaving debug console.log before Phase 7.** Any debug remnant is an automatic FAIL. See `references/security-rules.md` rule 10.
6. **Wrong `express-rate-limit` store.** MemoryStore resets on cold start. Use Upstash Redis or Vercel firewall. See `references/security-rules.md` rule 6.
7. **Defaulting to full-stack when the user wants simple.** Not every app needs Express + Neon. Phase 1 MUST ask "static SPA, simple backend, or full-stack?" upfront. Going straight to API + DB architecture for a throwaway test or a simple dashboard is over-engineering. The user will tell you to strip it out, and you'll redo the plan.
8. **Writing placeholder secrets for apps with no secrets.** A static SPA (no backend, no DB) has zero secrets. Writing `DATABASE_URL=YOUR_VALUE_HERE` and then asking the user to fill it in is confusing noise. Skip the `.env` entirely when there are no backend services.
